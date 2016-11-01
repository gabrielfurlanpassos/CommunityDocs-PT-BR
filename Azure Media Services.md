#Azure Media Services – Conceitos iniciais
##Fabrício Sanchez

Atualmente estou trabalhando em um projeto “pesado” de migração de AWS para Azure. Estamos migrando cerca de 10.000 vídeos mais a aplicação que gerencia estes assets da AWS para que estes sejam entregues por nossa plataforma através do Azure Media Services (AMS). Até entrar neste projeto, conhecia o AMS em um nível mais alto, sem nunca ter feito algo mais “sério” usando a plataforma.

Estou relatando esta experiência para finalmente constatar que, por conta deste novo projeto, as coisas mudaram consideravelmente em relação a maneira como eu enxergava o recurso. Como se trata definitivamente um processo não trivial de migração, estou tendo que ir mais fundo nas entranhas do serviço e com isso, estou podendo atestar de forma prática o quão eficiente o AMS é hoje em dia.

Dessa maneira, com este post, quero iniciar uma nova série (sem frequência e periodicidade pré-definidas) que vai falar mostrar o AMS do básico (conceitos iniciais) até seus aspectos mais deep. Me acompanha em mais este caminho de aprendizado e troca de experiências?

###Afinal, o que é o Azure Media Services?

Quando falei que iniciaríamos do básico, não estava brincando :-). Se você está chegando agora ao universo do Azure, este tópico é pra você. Se você já tem uma noção do que é o AMS, recomendo passar para a próxima seção.

O Azure Media Services (ou simplesmente AMS) é um serviço de plataforma (PaaS, portanto) que realiza todo o processo de ingestão, codificação e entrega de conteúdos multimídia (leia-se, vídeos) para o usuário final (através de players web). Para aterrizar este conceito, imagine um cenário típico de TV pela internet, onde uma programação de vídeos é entregue de forma contínua para o usuário final, seja ao vivo (live streaming) ou através de uma playlist de vídeos armazenados (on-demand streaming) previamente.

Como é peculiar aos serviços de plataforma, o AMS absorve uma grande complexidade operacional para entregar uma experiência rica para quem projeta uma aplicação que o utiliza como backend. Processos extremamente complexos como: encoding, bit rates, proteção de conteúdo, autenticação, extração de insights nos vídeos, etc., podem ser muito facilmente executados através desta plataforma. A Figura 1 apresenta esta ideia.


<p align = "center">
  <br>
  <img src="http://i0.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/whats-ams.png?resize=620%2C173"/>
  <br>
  Figura 1. Azure Media Services (big picture)
  <br>
</p>


Quando falamos em delivery de arquivos de mídia (mais especificamente vídeos) através de uma plataforma como o AMS, necessariamente estamos falando sobre duas possibilidades: entrega de vídeos sob demanda (on-demand) ou vídeos ao vivo (live). Com o AMS você pode fazer das duas maneiras (trataremos disso no futuro em conteúdos específicos para este fim).

##Conceitos iniciais
O trabalho com AMS exige o conhecimento prévio de muitos conceitos importantes. Entender estes conceitos de maneira clara é fundamental para se construir boas soluções usando o Azure Media Services, seja de maneira completa (utilizando a plataforma como um todo) ou de maneira parcial (sim, é possível utilizar o AMS de maneira segmentada com recursos de terceiros em seu contexto). Dessa maneira, vou dedicar este post e o próximo (vou dividir em dois para não tornar a leitura cansativa) para tratar destes conceitos.

###Assets: o que são e como funcionam?

Os assets são, sem a menor sombra de dúvidas, o primeiro (e talvez mais importante) conceito a ser entendido quando o assunto é o AMS. Isso porque entender o conceito de assets significa entender a entidade principal do AMS. Gosto de dizer que um asset está para o AMS como o objeto está para as linguagens orientadas a objetos. Para que a ideia de “o que é um asset” fique clara, considere a Figura 2, a seguir.


<p align = "center">
  <br>
  <img src="http://i1.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/ams-asset.png?resize=487%2C331"/>
  <br>
  Figura 2. Um asset ilustrado]
  <br>
</p>


<p align = "center">
  <br>
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/media-services-static-packaging.png?resize=620%2C290"/>
  <br>
  Figura 5. Modelo estático de encoding
  <br>
</p>


Um asset pode ser entendido como uma unidade de agrupamento lógica de arquivos associados a um arquivo de vídeo final (que deverá ser único no contexto do asset). O conteúdo de um asset é o que será submetido posteriormente para o processo de enconding. Em relação a um asset, existem alguns aspectos importantes a serem considerados. São eles:
* Um asset deve possuir apenas um arquivo de vídeo. Conforme ilustra a Figura 2, outros formatos de arquivo que estão relacionados a este arquivo de vídeo único poderão existir, entretanto, o vídeo (que será a saída final para o encoder) deverá ser unitário no asset.
* Os arquivos de um asset são fisicamente hospedados em container em um blob storage. Dessa maneira, se você tiver por exemplo, 1.000 vídeos para serem “encodados”, a storage account que hospeda esses 1.000 vídeos terá, necessariamente, 1.000 containers.
* Asset File. Quando um arquivo de vídeo ou áudio é enviado para um asset, necessariamente deve existir um arquivo físico associado a ele no blob storage. Esse arquivo físico é o que o Azure define como Asset File.
* Shared Access Siganture (SAS). AMS implementa a estratégia SAS para os elementos armazenados em suas estruturas de storage. Em linhas gerais, SAS é um URI que engloba em seus parâmetros de consulta todas as informações necessárias para o acesso autenticado a um recurso de armazenamento. Para acessar recursos de armazenamento com a SAS, o cliente só precisa passar a SAS ao método apropriado. No caso do AMS, por conta da regra de SAS implementada por ele, não é possível ter mais que 5 locators simultâneos para um mesmo asset. Não se preocupe, falaremos dos locators daqui há pouco.

São exemplos de assets válidos: um filme completo, um trailer de algum filme, uma gravação de uma câmera, etc. São exemplos de assets inválidos: Um filme e todos os seus trailers no mesmo asset, dentre outros.

Vale mencionar também que, assim como todos os demais recursos do Azure, o AMS possui um rico conjunto de APIs, que possibilitam o acesso aos recursos de de maneira programática por qualquer linguagem de programação. Parte destas APIs referem-se aos assets. Não se preocupe, falaremos com riqueza de detalhes sobre as APIs do AMS em um futuro próximo.


Os assets podem ainda ser criptografados (por questões de segurança) através de diferentes estratégias (StorageEncrypted, CommonEncryptionProtected e EnvelopeEncryptionProtected) e ainda, podem ter políticas de acesso bem definidas (somente leitura, gravação, tempo de leitura pré-definido, etc.). Estes dois aspectos são super importantes quando pensamos no aspecto segurança dos conteúdos que serão distribuídos.

É importantíssimo mencionar aqui também um outro aspecto relacionado aos assets: Locators. Locators são a porta de entrada para os elementos (arquivos) que estão armazenados em um asset. Esta é a única maneira de acessar esses recursos. Lembra quando mencionei que é possível criar políticas de acesso aos recursos? Então, tais políticas são aplicadas sobre os locators. Por exemplo, é possível criar uma política de acesso que restrinja o acesso do cliente a determinado arquivo de vídeo apenas por algum tempo, dentre outras coisas. Veremos isso de maneira prática nos próximos posts. Ainda sobre locators e políticas de acesso, é possível ter n locators para uma única política de acesso.

###Jobs e Tasks

No contexto do Azure Media Services, um Job é um certo tipo de entidade que é alocada para realizar algum processamento mais amplo (por exemplo, Encoding). Ele possui metadados que definem a natureza de cada Task a ser executada. Um Job pode possuir uma ou várias Tasks. Cada Task é a responsável de fato, por executar uma operação atômica (por exemplo, obter determinado arquivo em um asset, fazer algum processamento com esse arquivo obtido, etc.). Se vários processos de Encoding precisarem ser disparados, vários Jobs serão criados.

A Figura 3 apresenta esta ideia.


<p align = "center">
  
  <img src="http://i1.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/job-schema.png?resize=223%2C273"/>
  <br>
  Figura 3. Uma visão conceitual de Job e Tasks
</p>


<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/media-services-dynamic-packaging.png?resize=507%2C201"/>
  <br>
  Figura 6. Dynamic package
</p>


###Encoding

O processo de encoding é vital para qualquer plataforma de entrega de aúdio e vídeos. Trata-se da capacidade que uma plataforma de mídia tem de, ao receber um vídeo bruto como entrada, realizar processos internos de conversão tendo como saída o mesmo vídeo em novos e diferentes formatos, que possam ser consumidos por diferentes dispositivos conectados a internet. São exemplos de formatos de saída: H.264, MP4, dentre outros. Além disso, o processo de encoding deve garantir a geração de diferentes dimensões de vídeos (os famosos 240p, 360p, 480p, 720p, 1080p) para que o usuário final possa escolher o melhor formato para assistir esse vídeo.

Você que acompanha esse blog já deve ter percebido que gosto de recorrer a exemplos práticos para aterrizar conceitos. Para falar do processo de encoding não será diferente. A Figura 4 apresenta um cenário típico de aplicação que poderia estar utilizando o AMS. Basicamente, temos uma aplicação web (representada pelo CMS) que envia e consome vídeos do Azure Media Services. O processo de encoding, como é possível perceber, é intermediário na arquitetura.

<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/IC721586.png?resize=552%2C471"/>
  <br>
  Figura 3. Visão geral de uma aplicação que utiliza AMS
</p>


Como estamos falando especificamente da etapa de encoding, podemos ampliar a visão especificamente sobre este processo (ver Figura 4). Como é possível visualizar, o processo de encoding no AMS implementa o conceito de pipeline. O Job criado para este fim vai disparando as Tasks de maneira sequencial. Os resultados são consolidados nas próprias tasks e na sequência, o processo de encoding consolida todos os processamentos e gera uma saída esperada.


<p align = "center">
  <img src="http://i1.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/IC721590.png?resize=593%2C240"/>
  <br>
  Figura 4. O processo de encoding
</p>


O AMS faz o processo de encoding de maneira muito eficiente e ainda, com diferentes sabores de configurações e metodologias. São vários os formatos que podem ser “inputados” para o encoder e posteriormente gerados por ele como saída. A Tabela 1 a seguir os apresenta alguns formatos de entrada suportados. Já a Tabela 2 apresenta alguns formatos de saída. Estou assumindo aqui que você conhece a diferença entre arquivos de áudio e codecs.


<p align = "center">
  <img src="http://i0.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/input-formats.png?resize=620%2C593"/>
  <br>
  Tabela 1. Alguns (não todos) formatos de arquivos aceitos como entrada pelo encoder
</p>


<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/output-format.png?resize=620%2C310"/>
  <br>
  Tabela 2. Formatos de saída gerados pelo AMS encoder
</p>


Em termos de modelos operacionais de encoders, o AMS oferece duas opções: Standard e Premium. Em linhas gerais, a diferença básica está na flexibilidade oferecida por e por outro no processo de encoding. Enquanto no modelo Standard existem uma série de configurações já pré-definidas e que não podem ser alteradas, no modelo Premium existe mais granularidade, sendo possível por exemplo, dentre outras coisas, montar um fluxo personalizado de encoding com ferramentas gráficas específicas e disponíveis para este fim. Os valores aplicados para um e para  outro também são ligeiramente diferentes. Você pode fazer a consulta dos valores através deste link.

###Dynamic Package

Outro recurso bem interessante relacionado ao processo de encoding do Azure Media Services é aquele que a Microsoft chama de “Dynamic Package”. Trata-se de um processo inteligente de entrega de vídeos sob demanda para os diferentes universos de tecnologias “entendíveis” pelos dispositivos dos clientes finais. Por exemplo, disposítivos iOS entendem “HTTP Live Streaming (HLS)” enquanto XBox, por exemplo, espera por “Smooth Streaming”. Para que possamos entender de maneira concreta o modus operandi do processo de empacotamento dinâmico, considere a Figura 5.

<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/media-services-static-packaging.png?resize=620%2C290"/>
  <br>
  Figura 5. Modelo estático de encoding
</p>

A Figura 5 apresenta o processo de encoding estático. Isto é: um asset é “encodado” e um artefato com taxa de bits bem definida é gerado e, a partir dele, são gerados arquivos físicos com diferentes formatos de saída. Este é um processo extremamente funcional, entretanto, não é preciso pensar muito para entender que o consumo de storage é consideravelmente maior (uma vez que várias versões diferentes de um mesmo arquivo serão geradas para atender os diferentes cenários de consumo).

O que acontece com o dynamic package é que esta etapa de geração de múltiplas versões de um mesmo arquivo de saída é eliminada. A partir do artefato com taxa multi-bitrate, o recurso de dynamic package do AMS consegue gerar, sob demanda, os formatos específicos de entrega para cada dispositivo. A Figura 6 apresenta este cenário.


<p align = "center">
  <img src="http://i2.wp.com/fabriciosanchez.azurewebsites.net/3/wp-content/uploads/2016/09/media-services-dynamic-packaging.png?resize=507%2C201"/>
  <br>
  Figura 6. Dynamic package
</p>


Bacana não? Como o texto está ficando grande demais, vou deixar para a próxima postagem alguns outros conceitos relacionados ao Azure Media Services. Aos poucos iremos nos aprofundando nos recursos da plataforma e não tenho dúvidas de que, em pouco tempo, teremos vários insumos técnicos criar soluções eficientes e escaláveis com o serviço.

Muito obrigado e até a próxima!

####Fabrício Sanchez é MS Technical Evangelist, (Ex) Azure MVP, startupeiro, arquiteto de software, professor universitário, marido e nas horas vagas, escritor.


