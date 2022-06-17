---
title: Profiles
description: "Customizando o layout do wireshark para melhor análise."
lesson: 1
layout: post
parent: "/analise-de-trafego"
---

## Tabela de conteúdos
1. [Wireshark profiles](#wireshark-profiles)
    1. [Coluna time e o tempo](#coluna-time-e-o-tempo)
    2. [Adicionar nova coluna tempo](#adicionar-nova-coluna-tempo)
    3. [Adicionar colunas através de campos nos protocolos](#adicionar-colunas-através-de-campos-nos-protocolos)
    4. [Ativar e desativar colunas](#ativar-e-desativar-colunas)
    5. [Cores no tráfego](#cores-no-tráfego)
    6. [Layout do Wireshark](#layout-do-wireshark)

---

## Wireshark profiles

Os perfis são configurações que se pode fazer no wireshark pra melhor visibilidade do que está sendo analisado. Seja mudar a posição dos blocos apresentados como especificar cores específicas para determinadas situações, assim sendo mais fácil de visualizar aquilo ocorrendo no sniffer.
Para criar um novo perfil, deve-se clicar com o botão direito em profiles (canto inferior direito) e em "new..."

### Coluna Time e o tempo

A coluna “time” no wireshark está marcando o tempo de chegada dos pacotes a partir do momento que o primeiro pacote foi capturado. Sendo assim, a primeira captura foi realizada em 0.000... e servirá de referência para marcação de tempo de todos os outros pacotes.

É possível mudar para que o tempo agora seja em relação ao pacote anterior, e não ao primeiro. Assim é mais fácil de perceber problemas de lentidão na rede.

Para isso, vamos em View, nas opções no menu superior do wireshark. Após isso, acharemos “Time Display Format” e escolheremos “Seconds since previous displayed packet”. - Assim, a referência para o tempo será o pacote anterior, não o primeiro. A ideia é adicioanr uma nova coluna para essa marcação de tempo, mantendo a anterior.

### Adicionar nova coluna tempo

Para adicionar uma nova coluna, abriremos as preferências e clicaremos no + dentro da aba de colunas.

Modificamos o nome e escolheremos o tipo "Delta time”.

Após isso, arrastaremos a linha para abaixo da “Time”, assim as colunas ficarão uma do lado da outra.

![Untitled](/assets/imagens/wireshark/imagem5.png)

Na imagem é possível ver como ficaram organizadas as colunas de tempo.

A coluna "Time" indicando o tempo em relação à primeira captura, já a coluna "Delta" indicando o tempo em relação à captura anterior.

Obs: sempre que configurar o tempo através do menu “View”, a coluna Time será a afetada.

### Adicionar coluna através de campos nos protocolos

É interessante adicionar uma coluna sempre que um determinado campo for constantemente analisado, assim, não é necessário buscar aquele campo dentro dos protocolos sempre que precisar.

Para adicionar, por exemplo, o TTL como uma coluna, abriremos um pacote, clicaremos com botão direito no campo e em “apply to column”.

Também é possível editar o nome da coluna ao clicar com botão direito. Assim mudaremos de "Time to Live" para "TTL", diminuindo o espaço ocupado pela coluna e ainda conseguindo entender o que aquela coluna indica.

### Ativar e desativar colunas

Ao clicar no head com o botão direito - a faixa cinza onde estão todas as colunas - é possível desativar e ativar colunas facilmente. Também é possível realizar edições nas colunas.

### Cores no tráfego

Para colorir um determinado filtro no wireshark, vamos em “View” e “Coloring Rules”. Alí é possível editar as regras existentes ou adicionar uma nova.

![Untitled](/assets/imagens/wireshark/imagem6.png)

Adicionamos a cor verde ao filtro tcp.flags.syn==1. Dessa forma, sempre que houver um TCP com flag SYN setada, ele será verde. 

- OBS: A nova regra de cor criada foi movida para a segunda posição, abaixo de "bad TCP". Isso foi feito pois a lista segue uma ordem de prioridade, fazendo essa mudança estou indicando que a prioridade na cor será bad TCP.
Sendo assim, caso ocorra um TCP retransmission, não será verde - seria, se a regra criada manualmente estivesse em maior prioridade. 

### Ajustando o layout do wireshark

Na aba de preferências é possível modificar o layout do wireshark.

O padrão é o primeiro, em formato de escada.

![Untitled](/assets/imagens/wireshark/imagem7.png)

É interessante também utilizar a segunda opção.
Também é possível modificar o que está dentro de cada painel, logo abaixo das figuras.
