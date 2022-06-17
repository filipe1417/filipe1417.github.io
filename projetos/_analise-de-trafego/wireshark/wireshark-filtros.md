---
title: Wireshark filtros
description: "Quais os tipos de filtros e como utilizá-los corretamente."
lesson: 2
layout: post
parent: /analise-de-trafego
---

## Tabela de conteúdos
1. [Observações iniciais](#observações-iniciais)
2. [Filtros](#filtros)
    1. [Display filters vs Capture filters](#display-filters-vs-capture-filters)
    2. [Filtrando por IP](#filtrando-por-ip)
    3. [Filtrando por protocolos](#filtrando-por-protocolos)
    4. [Filtros como botões](#filtros-como-botões)
    5. [TCP analysis](#tcp-analysis)
    6. [Conversation filters](#conversation-filters)
    7. [Preparar e aplicar filtros](#preparar-e-aplicar-filtros)
    8. [Operadores e combinação de filtros](#operadores-e-combinação-de-filtros)
    9. [Filtros especiais](#filtros-especiais)
    10. [Reduzindo e exportando um pcap com filtros](#reduzindo-e-exportando-um-pcap-com-filtros)

---

## Observações iniciais

O wireshark sempre nos mostrará o protocolo da camada mais alta na coluna “Protocol”.

![Untitled](/assets/imagens/wireshark/imagem.png)

No exemplo acima, o protocolo que aparece é TCP, mesmo que a porta de destino seja a porta 80 (padrão para http). Isso também acontece no caso da porta de origem sendo 80.

Nesse caso, isso acontece pois houve a comunicação TCP com a porta 80 mas nenhum payload HTTP foi enviado. Assim, ao filtrar por TCP, é possível perceber tudo que aconteceu naquela porta através de TCP, desde o início da comunicação (vê-se o handshake) até outros pacotes TCP que não contém payload HTTP. Dessa forma, pode ser interessante filtrar não pelo HTTP, visualizando somente os pacotes que contêm payload, mas sim pela porta TCP 80 (ou outra que esteja segurando o servidor HTTP). 

Isso é válido pois pode ser importante visualizar o que está ocorrendo na camada que sustenta aquela aplicação.

## Filtros
Durante o uso de uma ferramenta de análise de tráfego, uma grande quantidade de pacotes é capturada e é difícil realizar uma análise eficiente ao tentar analisar a captura como um todo. Sendo assim, os filtros permitem que seja especificado o que se está procurando na captura.

Detalhe importante, sempre que clicar em um campo dentro de um pacote, no canto inferior esquerdo, o wireshark mostrará como se pode filtrar a partir daquele campo.

![Untitled](/assets/imagens/wireshark/imagem1.png)

No exemplo acima, ao clicar na flag tcp SYN, podemos visualizar no canto inferior esquerdo que podemos utilizar “tcp.flags.syn” para filtrar no Display filter.

Note que o “1” seria equivalente ao set, então para filtrar somente pacotes TCPs com onde o SYN ocorreu, faremos da seguinte forma: tcp.flags.syn==1.

### Display filters vs Capture filters

Os display filters são filtros que são aplicados após os pacotes terem sido capturados, assim, pode-se modificar como e quanto quiser. É possível adicionar um filtro, remover, adicionar novamente, retirar todos os filtros... e ainda assim a captura estará lá, completa. Os display filters simplesmente estão filtrando quais pacotes devem aparecer naquele momento, mas os outros pacotes também foram capturados e é possível visualizá-los quando quiser.

Os capture filters são aplicados antes de começar a capturar pacotes, ao escolher a interface de rede, e não podem ser modificados durante a captura. Ao definir um capture filter, só serão capturados pacotes que seguem aquele filtro definido. O padrão é capturar tudo.

**Os filtros mostrados daqui em diante serão display filters.**

### Filtrando por IP

- ip.addr==192.168.1.2 - pacotes com endereço IP de origem OU destino 192.168.1.2
- ip.src==192.168.1.2 - pacotes com endereço IP de ORIGEM 192.168.1.2
- ip.dst==192.168.1.2 - pacotes com endereço IP de DESTINO 192.168.1.2
- ip.addr=192.168.1.0/24 - pacotes de origem ou destino saíndo ou chegando naquela rede.

Utilizando o CIDR (por exemplo, /24) é possível também filtrar por somente origem ou destino.

<aside>
💡 A aba de “Endpoint” dentro de “Statistics” permite visualizar os hosts que participaram da comunicação até aquele momento. Lembrando que é possível marcar a caixa no canto inferior para limitar os endpoints mostrados ao que foi filtrado somente.

</aside>

### Filtrando por protocolo

Para filtrar diretamente por protocolo, é necessário simplesmente digitar qual protocolo quer dentro do input do Display filter.

Exemplos: arp, ip, udp, etc...

<aside>
💡 É possível acessar alguns filtros de referência ao acessar a aba “Display filters” dentro de “Analyze”.
</aside>

### Filtros como botões

Logo ao lado do input de Display filter, há um "+" onde se pode adicionar um filtro como botão. Isso permite que só seja necessário clicar no botão para aplicar o filtro. 

![Untitled](/assets/imagens/wireshark/imagem2.png)

Note na imagem que os botões “TCP SYNs” e “TCP ERRORs” foram criados para facilitar caso queira filtrar por esses parâmetros.

O círculo vermelho marca o “+”, onde é possível adicionar novos botões. Nesse caso, é difícil ver o botão pois está da mesma cor que o fundo do wireshark.

<aside>
💡 Ao criar um filtro com nome que contém //, estará sendo criado um menu dropdown. Por exemplo: TCP//Reset, criará um botão TCP que contém o botão Reset dentro.
</aside>

### TCP analysis

Essas flags não são do TCP em si, mas sim do wireshark. Ao filtrar o tcp.analysis.flags, é possível encontrar atividade suspeita que pode ser analisada. Não significa que sempre será algo suspeito, mas vale ficar de olho.

### Conversation filters

Ao clicar com o botão direito em um dos pacotes, é possível acessar a aba “conversation filters”, que permite filtrar somente os pacotes que fazem parte daquela "conversa". Ao fazer isso, estarão sendo filtrados os pacotes TCP que fazem parte daquela comunicação entre as duas portas.

![Untitled](/assets/imagens/wireshark/imagem3.png)

Note que ao aplicar o conversation filter, de 2186 pacotes, só aparecem 51. Também é possível perceber que o filtro aplicado não é somente “tcp”, mas algo mais complexo que permite filtrar além do protocolo. O conversation filter especifíca, por exemplo, que o que deve ser mostrado na tela são os pacotes da comunicação entre 2 determinados IPs e 2 determinadas portas.

É possível analisar quais foram as “conversations” que aconteceram durante a análise. É possível acessar na aba “conversations” dentro de “statistics”.
Exemplo: conexões TCP dentro daquela sessão de captura.

![Untitled](/assets/imagens/wireshark/imagem4.png)

<aside>
💡 É possível aplicar um conversation filter através dessa janela clicando com o botão direito na comunicação e em “apply filter”.
</aside>

### Preparar e aplicar filtros

É possível aplicar filtros ao clicar com botão direito em um campo do pacote e em apply filter. Ainda assim, é possível fazer o mesmo com prepare filter.

A diferença é que o prepare filter simplesmente cria o filtro e mantém dentro do input display filter, no topo do wireshark (ele não aplica). Assim é possível modificar como quiser antes de aplicar.

O apply filter adiciona ao input do display e logo em seguida aplica. Ainda é possível editá-lo, mas ele ja terá sido aplicado.

### Operadores para combinar filtros

É possível combinar filtros utilizando os seguintes operadores ja conhecidos:

- “==“ ou “eq”
- “!” ou “not”
- “ll” ou “or”
- “&&” ou “and”
- “>” ou “gt”
- “<” ou “lt” - (LT em minúsculo)

**Exemplo:**

ip.addr eq 192.168.1.1 && tcp

Mostra todas os pacotes tcp, entrando ou saíndo do endereço IP 192.168.1.1

**Exemplos com contexto:**

Alguns protocolos foram analisados e ARP parece “saudável”. Sendo assim, pode ser melhor aplicar um filtro para excluir as "conversas" ARP da saída.

Isso seria feito da seguinte forma:

!arp

Também poderia ser feito com mais de um, exemplo:

!(arp or stp)

**Não mostrar os pacotes com destino Broadcast:**

!(eth.dst == ff:ff:ff:ff:ff:ff) 

(eth - Ethernet)

<aside>
💡 Também é possível aplicar um campo do protocolo como filtro, clicando com botão direito e “apply as filter”.
</aside>

### Filtros especiais

- contains (string exata) - ex: frame contains google
- matches (regex) - ex: [http.host](http://http.host) matches “\\.(org l com l net)”
- in {range} - ex: tcp.port in {80 443 8000}

obs: contains compara a string exatamente como foi escrita, o matches não considera se alguma letra é maiúscula ou minúscula.

Mais alguns exemplos com matches ou contains:

**Ex: frame matches admin** 

Note que nesse caso o contains poderia substituir o matches, mas frames contendo palavras “admin” com alguma letra maiúscula não apareceriam.

**Ex2: frame contains GET**

### Reduzindo e exportando um pcap com filtros

Também podemos reduzir o tamanho do pcap (arquivo de captura de pacotes) sendo analisado. Para isso, definimos um display filter e após isso “export specified packets” dentro da aba “File”. Além disso, é possível especificar um “range” de pacotes para serem exportados dentro da própria janela de exportação.

---
