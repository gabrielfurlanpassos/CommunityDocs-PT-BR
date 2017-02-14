#Trabalhando com sensores em UWP – Parte 1 – Sensor de Luz

##Bruno Sonnino

Há algum tempo, os fabricantes de computadores introduziram novas classes de dispositivos Windows – os dispositivos 2-em-1 e os tablets. Estes tipos de computadores trazem muita inovação – os computadores 2-em-1 podem ser usados como notebooks ou como tablets, sem perder poder de processamento: você pode ter dispositivos muito poderosos que podem ser usados tanto no escritório como em viagem, evitando assim a necessidade de ter um tablete e um notebook.

Mas este tipo de dispositivos traz outro tipo de inovações que não estão visíveis a olho nu: sensores. Com eles, você pode ter a mesma experiência que tem com um smartphone – você pode usar o acelerômetro para jogos ou gestos, o GPS para localização e roteamento e muitos outros sensores. Esta série de artigos irá mostrar como usar estes sensores em UWP, de maneira que você pode dar a seu usuário uma melhor experiência e permitir que ele use todos os recursos de seu dispositivo. E tudo isso com um bônus adicional, graças ao desenvolvimento UWP – o mesmo projeto pode ser usando tanto em dispositivos desktop quanto em smartphones.

##Trabalhando com sensores em UWP

Basicamente, o trabalho com sensores em UWP segue a mesma rotina:

* Obter uma instância do sensor com o método GetDefault
* Configurar suas propriedades
* Configurar um manipulador para o evento ReadingChanged

Isto é tudo o que é necessário para se trabalhar com sensores em UWP. Assim, vamos iniciar com o primeiro sensor: o sensor de luz.

##Trabalhando com o sensor de luz

O sensor de luz pode ser usado para detectar a luz ambiente e ajustar o brilho da tela de acordo com a luz externa, para que o usuário tenha a melhor visualização possível. Neste artigo irei mostrar uma maneira simples de criar um efeito usado nos dispositivos GPS: quando é dia, ele mostra a tela com um fundo branco e quando é noite, ele mostra um fundo preto.

Para obter este efeito, a maneira mais fácil é usar temas. A programação UWP traz tr6es temas, Dark (Escuro), Light (Claro) e High Contrast (Alto Contraste). Você pode configurar o tema usando a propriedade RequestedTheme do controle (em nosso caso, a janela principal). Então, vamos lá.

No Visual Studio, crie um aplicativo UWP e adicione este código na página principal:

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
       <TextBlock Text="Day" HorizontalAlignment="Center" x:Name="MainText"
           VerticalAlignment="Center" FontSize="48" />
    </Grid>
    
Então, no construtor de MainPage, inicialize o sensor de luz e processe as leituras:

    public MainPage()
    {
        this.InitializeComponent();
        var lightSensor = LightSensor.GetDefault();
        if (lightSensor == null)
        {
            MainText.Text = "There is no light sensor in this device";
            return;
        }
        lightSensor.ReportInterval = 15;
        lightSensor.ReadingChanged += async (s, e) =>
        {
            await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
            {
                if (e.Reading.IlluminanceInLux < 30)
                {

                    RequestedTheme = ElementTheme.Dark;
                    MainText.Text = "Night";
                }
                else
                {
                    RequestedTheme = ElementTheme.Light;
                    MainText.Text = "Day";
                }
            });
        };
    }
    
Devemos usar o Dispatcher para mudar o tema e o texto, pois o evento ReadingChanged pode ser chamado em uma thread diferente da thread de UI. Quando a leitura da iluminância cair abaixo de 30 lux, mudamos para o tema Dark e o texto do TextBlock para “Night”. 

Agora, ao executar o programa em um dispositivo que tem o sensor de luz, você pode ver a vista padrão para o dia, como esta aqui:

<p align = "center">
  <br>
  <img src="http://lab27.blob.core.windows.net/wordpress/2017/02/Sensores_1.jpg"/>
  <br>
</p>

Ao cobrir o sensor de luz para simular a falta de luz, você verá algo como o seguinte:

<p align = "center">
  <br>
  <img src="http://lab27.blob.core.windows.net/wordpress/2017/02/Sensores_2.jpg"/>
  <br>
</p>

##Conclusões

Como você pode ver, é muito fácil trabalhar com sensores. Com eles, você pode dar uma melhor experiência para seus usuários, de maneira que eles possam apreciar seus aplicativos. E com um benefício adicional, que você pode usar o mesmo aplicativo em um desktop ou em um Windows Phone.

O código fonte para este projeto está em https://github.com/bsonnino/LightSensor e, se você quiser ver um vídeo com o conteúdo deste artigo, pode dar uma olhada em meu canal no Channel9: https://channel9.msdn.com/Series/Windows-Development/Trabalhando-com-Sensores-em-UWP-Parte-1-Sensor-de-Luz.




