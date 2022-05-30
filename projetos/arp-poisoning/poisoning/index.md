---
title: "ARP poisoning"
description: "Teoria e prática sobre o envenenamento ARP, um grande perigo às redes Ethernet. O foco será principalmente ataques do tipo Man in the middle, mas também serão apresentados outros tipos."
codigoURL: "https://github.com/filipe1417/arp-poisoning"
layout: post
parent: /projetos/arp-poisoning
---

# Tabela de conteúdos
1. [ARP poisoning](#arp-poisoning)
    1. [Ataques](#ataques)
    2. [Atacando com Scapy](#atacando-com-scapy)
2. [Referências](#referências)

---

# 1. ARP poisoning

OBS para verificar: em algumas literaturas, arp spoofing é o mesmo de arp poisoning. Em outras, é diferente.
Normalmente, o conceito de arp spoofing (quando aparece diferente de arp poisoning) é o mesmo de MAC spoofing.

O ARP poisoning, ou ARP spoofing, é um perigo enorme para redes Ethernet. O ataque consiste em forjar gratuitous ARP replies para que um, dois ou mais hosts alvo atualizem o cache da tabela ARP com valores falsos. Os valores falsos enviados abrem brecha para ataques diferentes. 

No geral, o ARP spoofing é utilizado para ataques man in the middle.


## 1.1. Ataques

### Isolando hosts

É possível evitar que hosts se comuniquem entre si em uma rede, assim como que consigam acessar redes exteriores ao gateway. Isso pode ser feito divulgando endereços MAC falsos para os hosts na rede.

#### Isolar um host A da comunicação dentro da própria rede
Isso pode ser feito garantindo que as outras máquinas na rede acreditem que o host A tem um determinado endereço MAC, que na verdade não existe na rede. Ao tentar comunicação com o host A, os pacotes não chegarão a lugar algum.

#### Isolar um ou vários hosts da comunicação com a internet
Segue o mesmo princípio do tópico anterior, mas aqui o host isolado seria o próprio gateway da rede.
Isso poderia ser feito falsificando os valores nas tabelas ARP de vários hosts na rede ou de algum específico. Note que como qualquer ataque que atinja uma quantidade grande de hosts, ou toda a rede, é bem barulhento. Sendo assim, é fácil visualizar que tem algo suspeito acontecendo.


### MITM (Man in the middle)
Utilizando o man in the middle também é possível realizar os ataques do tópico "Isolando hosts".
Diferente dos ataques anteriores, nossa máquina invasora estará no meio da comunicação, não somente falsicando correspondências do cache ARP sobre outros hosts na rede, mas participando ativamente para interceptação de informações.

A ideia principal de um ataque man in the middle utilizando ARP spoofing é fazer com que dois ou mais hosts na rede pensem que a maquina do invasor é o destino da comunicação.

Um host A deseja acessar um servidor web na internet. Um invasor sabe que para isso acontecer, os pacotes passarão pelo gateway. Também sabe que posteriormente, o gateway enviará pacotes em direção ao host A para que a comunicação aconteça. O invasor, então, utiliza o ARP spoofing para realizar um ataque MITM e intercepta a comunicação. O novo caminho lógico entre o host A e o gateway passará pela máquina do invasor, sendo assim, todo tráfego não criptografado será facilmente roubado pelo invasor.

![MITM.jpg](MITM.jpg)

#### Interceptando pacotes
Nesse tipo de ataque MITM, o host invasor estará no meio (de forma lógica) da comunicação entre dois hosts na rede, e é o uso principal para o MITM. Normalmente, um dos hosts é o gateway.

#### Como ocorre?
O invasor envia diveros gratuitous ARP replies, continuamente, para as máquinas que deseja envenenar.
Nos pacotes falsificados, o host A atualizará sua tabela ARP associando o IP do gateway com o endereço MAC do invasor. Do outro lado, o gateway atualizará sua tabela arp correspondendo o endereço IP do host A com o do invasor. Sendo assim, um novo caminho lógico é criado, e os pacotes que fazem parte da comunicação host A - gateway passarão através da máquina do invasor.

Quando um pacote que não é destinado ao seu host chega, ele é descartado (isso será usado para um ataque de isolamento utilizando MITM). Então para que a interceptação de pacotes possa ocorrer, precisamos habilitar o IP forwarding na nossa máquina. Ao habilitar o IP forwarding, a sua máquina atuará como um roteador, transferindo pacotes de uma rede até outra, ou nesse caso, de uma parte da rede até outra.

#### Isolando hosts
Com o IP forwarding habilitado, os pacotes passam atraveś do host do invasor e a comunicação ocorre normalmente, mas os dados podem ser visualizados.
Para isolar o host A da comunicação, basta desabilitar o IP forwarding durante o ataque Man in the middle. Assim, garantindo que a tentativa do host A de acessar a internet nunca chegue ao gateway.

## 1.2. Atacando com Scapy
Revisando o processo de ataque MITM com interceptação, temos:
1. Invasor habilita IP forwarding 
2. Invasor escolhe vítima(s)
3. Invasor consegue endereço MAC de ambos
4. Invasor envia um fluxo constante de gratuitous ARP replies falsificados para as vítimas
4. Invasor intercepta comunicação utilizando um sniffer na rede


### Habilitando IP forwarding
O IP forwarding será ativado através do sysctl na distribuição linux Arch
- sysctl -w net.ipv4.ip_forward=1

Para desativar, basta trocar de 1 para 0

### Escolhendo vítimas
Os hosts serão a máquina 192.168.1.11 e o gateway 192.168.1.1

A ideia é capturar o tráfego entre 192.168.1.11 e a internet, principalmente protocolos sem segurança de encriptação.

### Conseguindo os endereços MAC
Isso pode ser feito de várias formas, mas sempre com base em um ARP request.
Pode-se tentar enviar pings para os hosts, assim o cache ARP será atualizado com os endereços MAC das vítimas, por exemplo.
Para a criação do script, pode-se utilizar a função nativa do scapy "getmacbyip()" ou forjar um ARP request e analisar sua reposta ao enviar para cada host alvo.


```python
alvo_MAC = getmacbyip("192.168.1.11")
gateway_MAC = getmacbyip("192.168.1.1")
print(alvoMAC, gatewayMAC)
```
<div style='background-color: #eaeaea; color: black'>  
    <pre>
    70:85:c2:a0:16:c0 e8:01:8d:18:49:38
    </pre>
 </div>

### Fluxo constante de gratuitous ARP replies
Serão criados pacotes com as seguintes informações falsas:

- Para o host alvo, diremos que o endereço IP do gateway está em nosso MAC.
- Para o gateway, que o endereço IP do host alvo está em nosso MAC.

Também será criada uma função poison() que receberá os valores necessários e atuará continuamente em um laço de repetição While.


```python
def poison(IP_alvo1, IP_alvo2, MAC_alvo1):
    arp_reply = Ether()/ARP(pdst=IP_alvo1, hwdst=MAC_alvo1, psrc=IP_alvo2, op=2)
    sendp(arp_reply, verbose=0)
```

Detalhes da função criada:
- Arp reply (operation code é 2)
- Destinado para o IP passado do alvo 1
- MAC do alvo 1 passado, visto que é necessário para que o pacote chegue até ele
- O endereço IP de origem não é o da máquina do invasor, e sim do alvo 2 - nesse caso, do gateway
- Note que o endereço MAC de origem não foi alterado, é o da máquina do invasor

Dessa forma, diremos ao alvo 1 que o alvo 2 tem o endereço MAC da nossa máquina, isso pode ser visto pois o endereço MAC de origem do pacote é o da máquina do invasor, mas o IP do gateway.

Como o ARP reply é feito para atualizar a tabela ARP das máquinas na rede, ela será atualizada com a seguinte informação:
- IP do gateway está associado ao MAC do invasor


```python
while True: 
    poison("192.168.1.11", "192.168.1.1", alvo_MAC)
    poison("192.168.1.1", "192.168.1.11", gateway_MAC)
```

![MITMscapy.png](MITMscapy.png)

Ao analisar a execução do ARP spoofing através do wireshark, é possível visualizar os ARP reply falsificados sendo enviados continuamente.

# 2. Referências

OBS: Organizar referências:
- Análise de tráfego em redes TCP/IP - João Eriberto Mota Filho
- Redes de computadores - Tanenbaum / Wetheral
- Documentação Scapy - https://scapy.readthedocs.io/en/latest/
- RFC 826 
- ...


