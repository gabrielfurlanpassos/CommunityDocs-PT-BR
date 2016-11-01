##Balanceamento de carga as a Service no Microsoft Azure

###Fabrício Sanchez

Balancear carga é um dos principais requisitos de qualquer sistema de nuvem atual. As razões para isso são claras: otimização do uso dos recursos computacionais, maximização de throughput, diminuição do tempo de resposta e redução considerável da possibilidade de overload de determinado servidor. Dessa maneira, para que uma plataforma de computação em nuvem possa ser dita “robusta” no quesito de infraestrutura como serviço, uma das características que ela deve oferecer nativamente é um (ou alguns) bom(ns) serviço(s) para balanceamento de carga.

###Algumas (poucas) palavras sobre Load Balancer

De maneira simplificada, é chamado de Load Balancer o sistema que tem a capacidade de, ao receber determinada requisição, tem a capacidade de determinar qual máquina servidora irá ser responsável por tratar (com afinidade de sessão ou não) a mesma e devolver a respectiva resposta ao solicitante, considerando que existe um conjunto de máquinas servidoras gerenciadas por ele. A Figura 1 ilustra o funcionamento de um Load Balancer genérico.


<p align = "center">
  <img src="http://i0.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/08/BasicLB_2.jpg?resize=448%2C420"/>
  <br>
  Figura 1. Load Balancer básico
</p>

Existem diferentes estratégias (leia-se algoritmos) de distribuição de carga que atuam de acordo com as diferentes camadas do protocolo OSI (ver Figura 2). Não vou me ater aos detalhes do modelo OSI aqui por não ser este o foco deste artigo. Se você quer saber mais sobre este assunto, poderá efetuar a leitura do um bom conteúdo através deste link – em inglês).

<p align = "center">

<img src="http://i1.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/08/osi.jpg?resize=620%2C440/">
  <br>
  Figura 2. Modelo OSI de comunicação
</p>

Importante frisar que um Load Balancer é algo completamente diferente de um recurso comumente implementado em cenários concorrentes para garantir resiliência de interfaces de redes nos hosts, conhecido como “Channel Bounding (CB)“. Enquanto um Load Balancer convencional (camada 4) separa a carga virtualmente por sockets, CB faz a separação em um nível mais baixo (camada 3 ou 2 em alguns casos), diretamente no hardware.

###Load Balancers no Azure

Felizmente podemos dizer que o Microsoft Azure é uma plataforma robusta, e essa afirmação se dá por vários motivos, dentre eles, por oferecer boas e robustas opções para balanceamento de carga em diferentes níveis. Nele (Azure) existem basicamente três grandes serviços (cada um com suas peculiaridades e aplicações) para este fim:


* Azure Load Balancer;
* Azure Application Gateway;
* Traffic Manager;

A seguir (neste post) iremos discutir os principais aspectos, recursos e aplicações dos dois primeiros. Traffic Manager merece um olhar mais demorado e por isso mesmo, iremos tratar sobre ele um post no futuro.

####Azure Load Balancer



Azure Load Balancer (ALB) é um balanceador Layer 4 (lembra-se do modelo OSI? Estamos falando da camada de transporte) nativo da plataforma, que pode ser entregue em uma infraestrutura tanto no modelo ARM quanto no modelo ASM. Ele pode ser utilizado tanto para balancear requisições de maneira granular em um farm de máquinas virtuais (ARM), quanto em uma infra para balancear requisições de maneira automatizada com auto-scaling já agregado através de Cloud Services (ASM). A informação sobre o modelo de deployment da infraestrutura na qual estará inserido o ALB é importante, uma vez que é ela que determina a maneira como ele deve ser configurado e consequentemente utilizado.

Conforme mencionado anteriormente, quando estamos utilizando o modelo clássico (tecnicamente conhecido como ASM), utilizamos o ALB no contexto de Cloud Services (CS). Dessa forma, o ALB utiliza o IP público atribuído ao CS no momento de sua criação para servir como “inbond address“. Assim, o ALB consegue distribuir a carga de rede entre as máquinas agrupadas abaixo do respectivo CS. A Figura 3 ilustra bem este processo.

<p align = "center">
  <img src="http://i0.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/08/asm-lb.png?resize=604%2C259">
  <br>
  Figura 3. ALB no modelo clássico (ou ASM)
</p>


Vale lembrar que é preciso indicar manualmente as portas cujo o tráfego será balanceado para que o ALB consiga realizar o NAT adequadamente. Este processo deve ocorrer em cada instância do farm. No novo portal do Azure este processo é conhecido como “Inbound rules“. Ele deve ser configurado na área de gestão da interface de rede, na opção “Security group“, “Inboud rules“. Já no portal antigo, nas configurações da máquina virtual, você deve adicionar os chamados endpoints.

A outra opção é utilizar o ALB no modelo Resource Group (conhecido tecnicamente como ARM). Neste caso, o ALB é adicionado de maneira independente, não sendo necessário criar um Cloud Service para que o balanceamento ocorra. Em função disso, o ALB precisa ser criado manualmente e posteriormente adicionado a mesma rede virtual das máquinas virtuais que se quer balancear. Outra diferença aqui é que o IP público não é do CS mas sim, do próprio ALB e no caso, o DNS name deve apontar para este IP público através de uma entrada do tipo A no gerenciador do domínio, que pode ser o próprio Azure DNS (veja um post que fiz sobre esse assunto aqui).

A Figura 4 apresenta uma visão geral do ALB trabalhando no modelo ARM.

<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/08/arm-lb.png?resize=620%2C248">
  <br>
  Figura 4. ALB no modelo resource manager (ARM)
</p>

Outro aspecto importante a ser ressaltado aqui é a “perda” proposital (isso ocorre por conta do modelo granular oferecido pelo padrão ARM) de automação. Isso porque, conforme mencionado anteriormente, quem cuida de automatizar alguns recursos como “escala” e o próprio balanceamento de carga, é o Cloud Service. Isto quer dizer que, aqui, cada aspecto da arquitetura deve ser cuidadosamente implementado pelo administrador dos recursos. Especificamente sobre escala, se você precisa de um modelo dele automatizado para ARM, o recurso que você deve implementar é o VM Scale Set, mas isso é assunto pra outro post.

Existem ainda vários outros aspectos super importantes relacionados ao ALB como: modelos de afinidade de sessão, algoritmos de balanceamento, etc., entretanto, como a documentação do Azure é bem farta no apresentar destes conceitos, vou direcionar sua leitura a partir daqui para a mesma. Basta seguir este link para chegar até lá.

####Application Gateway

O Application Gateway é um serviço de balanceamento de carga layer 7. Isso significa dizer que ele consegue entregar algumas funcionalidades extremamente úteis que o ALB convencional (mencionado anteriormente) não consegue pelo fato de “enxergar” a aplicação (e portanto, HTTP). Destaco aqui duas principais:

* Afinidade de sessão por cookie: refere-se a capacidade do balanceador de preservar a sessão de navegação do usuário no mesmo servidor durante todo o ciclo de navegação deste utilizando cookies para este fim. O ALB convencional também consegue preservar sessão, entretanto, ele utiliza uma metodologia conhecida como “Source IP”. Em linhas gerais, para o ALB, apenas o IP quente (aquele que vale na web) é o que conta para que o ALB consiga preservar a sessão do usuário (lembrando que o ALB é layer 4, o que limita sua atividade em níveis mais altos). A desvantagem dessa abordagem é que se 1000 usuários, por exemplo, estiverem debaixo de um proxy, todos serão direcionados para o mesmo servidor pelo ALB. Expliquei isso para explicitar o porque a abordagem por cookies é tão interessante. No caso do Application Gateway, além das informações de origem e destino, o load balancer consegue criar um cookie que atrela um código único aquele usuário e desta maneira consegue manter a sessão ativa no mesmo servidor. Se outro usuário na mesma rede, por exemplo, pedir requisitar acesso a um mesmo recurso, este usuário poderá ir para outro servidor e ainda ter sua sessão mantida. Graças ao “bendito” cookie (veja Figura 5).

<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/application-gateway-cookie.png?resize=314%2C351">
  <br>
  Figura 5. Application Gateway com afinidade de sessão por cookie habilitada
</p>

* Offload de SSL: em um modelo de host tradicional (baseado em IIS, por exemplo), para cada instância (leia-se servidor web) que queria utilizar certificados assinados para suas soluções, é preciso que o servidor web faça o offload do SSL. No caso do application gateway, se quisermos, podemos “pedir” para que ele faça este papel, liberando as máquinas que estão abaixo dele desta responsabilidade. Legal, não? Neste caso, ao invés de subir o certificado para cada IIS de instância servidora, subimos o certificado apenas para a instância de Application Gateway e delegamos essa função para ele (veja a Figura 6). 

<p align = "center">
  <img src="http://i1.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/application-gateway-ssl.png?resize=320%2C414">
  <br>
  Figura 6. Configurando o SSL no Application Gateway
</p>

Neste post não estou preocupado em mostrar a configuração de um ambiente baseado em Application Gateway (podemos até fazer isso no futuro) ou baseado em ALB ou em ambos. Estou preocupado sim em apresentar insights sobre a utilização destes recursos no dia-a-dia. Desta forma, daqui em diante, quero compartilhar alguns aprendizados que pude acumular.

###1. Quando devo optar por uma abordagem ou por outra?

Depende. Minha dica aqui é fazer uma pergunta valiosa que pode ajudar nesta decisão. “Qual é o publico alvo e o comportamento (em termos de navegação) esperado deste público?”.

Para fazermos uma análise mais assertiva, tomemos como exemplo uma aplicação que tenha uma natureza mais corporativa (isto é, uma solução SaaS que será consumida por pequenas, médias ou grandes empresas). Muito embora esta aplicação possa ser acessada não necessariamente dentro da empresa (o que provavelmente colocaria todos os usuários em uma mesma rede), precisamos concordar que existe uma chance muito grande disso ocorrer, certo? Neste caso, não podemos correr o risco de que, mesmo tendo um sistema super bem configurado com escala automática e balanceamento de carga, ter um servidor sobrecarregado com risco eminente de queda, certo? Neste caso, a abordagem mais correta seria partir para o Application Gateway por conta das características arquiteturais que já mencionei anteriormente.

De outro lado, se a aplicação que se quer balancear tem um um comportamento de navegação totalmente aleatório, isto é, hora o acesso vem de um rede, hora vem de outra, etc., muito provavelmente o ALB deverá atender de maneira satisfatória, mesmo que se queira preservar sessão, também dada a natureza do recurso. De maneira geral as aplicações que apresentam este comportamento são aquelas mais voltadas para consumidor final. Apenas alguns exemplos: Waze, Uber, etc.

###2. ALB ARM ou ALB do Cloud Service?

Existem duas diferentes abordagens para o Azure Load Balancer tradicional. Uma independente, que pode ser acoplada de maneira isolada em arquiteturas ARM (o que chamamos aqui na Microsoft de Azure 2.0) e outra que já é nativa (e automatizada) na configuração de Cloud Services (arquiteturas ASM). Já tratamos disso nesse mesmo post.

O ponto é: Um é melhor que o outro? Porque existe essa distinção?

Para responder apropriadamente a esta pergunta seria necessário descer um pouco mais fundo sobre o porque temos dois modelos arquiteuturais (ASM e ARM) no Azure. Pretendo fazer este racional em um artigo futuro mas, por hora, basta entender que não existe melhor ou pior. Existem cenários distintos e aí sim, existe a estratégia que atende melhor a um cenário e não a outro.

Em linhas gerais, se a arquitutura montada para “rodar” determinada aplicação é baseada em ASM, será impossível utilizar o ALB ARM (de maneira autônoma). Neste caso, tudo deverá estar abaixo do Cloud Service e neste caso deveremos habilitar o ALB nativo dele (e esta será a melhor solução). De outro lado, se uma nova arquitetura está sendo proposta de maneira que exista a necessidade da granularidade, ARM será o melhor caminho e portanto, o ALB ARM será a melhor solução.

###3. Custo

O aspecto “custo” é sempre importante, portanto, você deve estar atento a necessidade real de uso de um Application Gateway. Digo isso porque o primeiro (ALB) não possui custo em sua utilização, entretanto, como já vimos anteriormente, é mais limitado em termos de recursos em relação ao Application Gateway.

###4. Modelos estruturais

ALB e Application Gateway são completamente diferentes em termos arquiteturais. Este assunto também merece um estudo mais aprofundado em um post dedicado para este fim, entretato, neste momento é suficiente saber que eles são estruturalmente são distintos (o ALB é implementado diretamente em uma camada do hypervisor, enquanto o Application Gateway é um serviço implementado sobre máquinas virtuais) e que isso tem, basicamente, uma implicação prática no projeto: para que um application gateway consiga ser performático em um ambiente de missão crítica, ele precisará escalar ao longo de múltipas instâncias e isso gerará algum custo adicional.

Enfim, é isso! Espero que seja útil.

Obrigado e até o próximo.

####Fabrício Sanchez é MS Technical Evangelist, (Ex) Azure MVP, startupeiro, arquiteto de software, professor universitário, marido e nas horas vagas, escritor.
