#Introdução à Programação Reativa com Rx.NET

##Bruno Sonnino

###Introdução

Em algum ponto do dia, você está aborrecido e o demônio da procrastinação toma conta de você. Você vai ao Facebook (ou ao Twitter, você pode escolher J) e começa a ver as atualizações. Após algum tempo, você já checou todas as postagens novas e começa a apertar F5 para refrescar a página e obter as atualizações, na esperança de ter algo interessante ali. Após algum tempo, você começa a pensar se não há alguma maneira automática de refrescar as postagens quando uma nova mensagem chega, de modo que você não tenha que teclar F5 e ficar frustrado porque não há nada de novo para ler.
Eu sei que isso nunca acontece a você, mas você pode imaginar a ideia: não seria melhor que houvesse algo que mostrasse as novas postagens sem a necessidade de refrescar os dados? Na realidade, isso já existe e é chamado de Programação Reativa (Reactive Programming).
Na nossa programação normal, tendemos a fazer o mesmo que nosso personagem fictício e perguntar por novos dados, como no código a seguir:

    while (true)
    {
      // Obtém novos dados
      // Processa os dados
    }


Isto é muito ineficiente e irá drenar os recursos de seus dispositivos: seu consumo de CPU irá para o alto e sua bateria será drenada em pouquíssimo tempo. Com Programação Reativa, ao invés de você perguntar por novos dados, seu programa será notificado quando houver novas informações: sem dúvida, esta é uma maneira mais eficiente de se obter os dados!
Então, vamos mergulhar neste novo paradigma de programação. Novo? Nem tanto. Programação Reativa é baseada num padrão de projeto chamado Observable. Em .NET, isto é implementado por duas interfaces, IObserver<T> e IObservable<T>, que foram introduzidas no .NET 4.0, em 2010. Logo após isso a Microsoft lançou um framework que implementa Programação Reativa, chamado de Rx.NET ou Reactive Extensions for .NET.

###Introdução a IObservable/IObserver

O Padrão Observer tem duas partes: um Observable, que irá enviar os dados e um (ou mais) Observer, que irá assinar as notificações de novos dados e irá exercer alguma ação sobre eles. Toda vez que penso nisso, eu me lembro de uma esteira industrial, como aquelas que você pode ver no filme “Tempos Modernos” do Charles Chaplin ou no episódio “A Fábrica de Chocolate” de “I Love Lucy”.

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/1.png.jpg">
<br>
Charles Chaplin – Tempos Modernos
</p>


<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/2.jpg">
<br>
I Love Lucy – A Fábrica de Chocolate
</p>

O Observable está liberando peças (ou chocolates) e os Observers estão agindo sobre eles, apertando as porcas ou embrulhando os chocolates. Para representar estas sequencias, podemos usar diagramas como os seguintes:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/3.jpg">
<br>
</p>

A primeira sequência mostra uma série de quatro pontos e não está terminada (os Observers podem continuar recebendo os dados). A segunda é uma série de três pontos que foi terminada. Os Observers não receberão mais dados a partir deste ponto. A terceira mostra três pontos e um erro. Como a sequência terminada, os Observers não receberão mais dados após o erro/

As duas interfaces são muito simples:

    public interface IObservable<out T>
    { 
      IDisposable Subscribe(IObserver<T> observer); 
    } 

    public interface IObserver<in T> 
    { 
      void OnCompleted(); // Fonte terminou de enviar mensagens.
      void OnError(Exception error); // Um erro ocorreu.
      void OnNext(T value); // Mais dados a processar.
    }
    
Na maior parte do tempo, você não implementará estas interfaces manualmente, mas usará as classes Observable e Observer para implementá-las. A classe Observer tem métodos de ajuda para criar observables que gerarão dados e têm um método Subscribe que retorna um IDisposable (que deve ser liberado quando não se quer mais dados) e a classe Observer tem métodos de construção que ajudam a criar instâncias que têm três delegates, para receber o próximo dado, processar erros ou terminar o processamento. Este código mostra um programa simples que processa os Observables (você deve incluir o pacote NuGet System.Reactive para compilar o projeto):

    static void Main(string[] args)
    {
        IObservable<int> source = Observable.Range(1, 10);
        IObserver<int> obsvr = Observer.Create<int>(
            x => Console.WriteLine("OnNext: {0}", x),
            ex => Console.WriteLine("OnError: {0}", ex.Message),
            () => Console.WriteLine("OnCompleted"));
        using (source.Subscribe(obsvr))    
        {
            Console.WriteLine("Press ENTER to unsubscribe...");
            Console.ReadLine();
        }
    }
    
Ele cria um observable que emite os inteiros entre 1 e 10 e um observer que escreve os valores recebidos na console. O resultado é algo semelhante a isso:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/4.jpg">
<br>
</p>

Neste código, eu usei um bloco “using” no resultado do método Subscribe, mas na maior parte do tempo isto não é o ideal – normalmente você armazena o resultado do método Subscribe em uma variável e chama Dispose mais tarde, quando você não precisa mais da assinatura. Isto é mais importante quando você usa observables assíncronos; com este código você não observaria nada, a assinatura seria dispensada antes de se observar alguma coisa.

###Filtrando sequências observáveis com LINQ

Nesta hora você deve estar pensando: “isto não é muito diferente do que eu estou acostumado a fazer, porque usar isso?”. A primeira resposta pode ser LINQ. Você pode usar LINQ para filtrar a sequência, como neste programa:

    static void Main(string[] args)
    {
        IObservable<int> source = Observable.Range(1, 10)
            .Where(x => x % 2 == 0)
            .Select(x => x * x);
        IObserver<int> obsvr = Observer.Create<int>(
            x => Console.WriteLine("OnNext: {0}", x),
            ex => Console.WriteLine("OnError: {0}", ex.Message),
            () => Console.WriteLine("OnCompleted"));
        using (source.Subscribe(obsvr))
        {
            Console.WriteLine("Press ENTER to unsubscribe...");
            Console.ReadLine();
        }
    }
    
Este programa é semelhante ao primeiro, mas o observable está filtrado, emitindo os quadrados dos números pares. O resultado aqui é:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/5.jpg">
<br>
</p>

Agora, eu estou certo que você está convencido das vantagens (sério?), mas ainda há mais: as sequências podem ser geradas de outras maneiras, como sequências temporizadas:

    static void Main(string[] args)
    {
        var source = observable.Timer(TimeSpan.FromSeconds(2),
         TimeSpan.FromSeconds(1))
            .Select((t,i) => new
            {
                Date = DateTime.Now,
                Item = i
            });
        using (source.Subscribe(
            x => Console.WriteLine($"OnNext: {x.Date} - {x.Item}"),
            ex => Console.WriteLine($"OnError: {ex.Message}"),
            () => Console.WriteLine("OnCompleted"))) 
        {
            Console.WriteLine("Press ENTER to unsubscribe...");
            Console.ReadLine();
        }
    }
    
Esta sequência inicia a emitir após 2 segundos e, após isso, emite novamente a cada segundo. Eu não criei explicitamente um Observer, mas usei uma sobrecarga do método Subscribe, que passa três lambdas como parâmetros. Quando você executa o programa, vê algo semelhante ao seguinte:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/6.jpg">
<br>
</p>

Você pode ver que o código não é sequencial: o Observer é criado e a mensagem “Press ENTER to unsubscribe” é mostrada. Então, após 2 segundos, o programa começa a escrever na console. Legal, não? Agora a programação reativa começa a fazer sentido. O programa irá continuar a saída até que você pressione Enter. Não há o evento Completed e ele continuará a execução até você terminar o programa.

Mas isto não é tudo o que podemos fazer aqui. Podemos criar observáveis a partir de manipuladores de eventos e combiná-los, de maneira a poder mover uma imagem em uma janela WPF.

###Combinando eventos observáveis 

Crie um projeto WPD e adicione o pacote NuGet System.Reactive. Adicione uma imagem ao projeto e, em MainWindow.xaml, mude o elemento raiz para um Canvas e adicione a imagem a ele:

    <Canvas>
        <Image Source ="Louvre.JPG" Width="150" x:Name="Image"/>
    </Canvas>
    
Então, em MainWindow.xaml.cs, adicione o código para arrastar a imagem:

    private void SetupDrag()
    {
        var mousedown = Observable.FromEventPattern<MouseButtonEventArgs>(
          Image, "MouseLeftButtonDown")
                .Select(evt => evt.EventArgs.GetPosition(Image));
        var mouseup = Observable.FromEventPattern<MouseButtonEventArgs>(
          this, "MouseLeftButtonUp");
        var mousemove = Observable.FromEventPattern<MouseEventArgs>(
          this, "MouseMove")
                .Select(evt => evt.EventArgs.GetPosition(this));
        var drag = from start in mousedown

from move in mousemove.TakeUntil(mouseup)
            select new
            {
                X = move.X - start.X,
                Y = move.Y - start.Y
            };
        drag.Subscribe(value =>
        {
            Canvas.SetLeft(Image, value.X);
            Canvas.SetTop(Image, value.Y);
        });
    }

Neste código, estamos combinando Observables que vêm de três eventos, MouseLeftButtonDown, MouseLeftButtonUp and MouseMove em um Observable “drag”, para posicionar a imagem no Canvas. Note que os eventos não vêm do mesmo componente: MouseDown vem da imagem (não queremos que a imagem seja arrastada se o usuário não clicar na imagem) e os outros vêm da janela. Se fossemos desenhar um diagrama, ele seria assim:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/7.jpg">
<br>
</p>

Para ilustrar este processo, usarei um add-in para o Visual Studio, chamado OzCode (http://www.oz-code.com), que auxilia a depuração de programas. Ele tem uma feature que permite listar os eventos e os dados que chegam a eles. Vamos criar três novas assinaturas para os eventos, para que possamos verificar quando elas são chamadas:

    private void SetupDrag()
    {
       var mousedown = Observable.FromEventPattern<MouseButtonEventArgs>(
         Image, "MouseLeftButtonDown")
                 .Select(evt => evt.EventArgs.GetPosition(Image));
         var mouseup = Observable.FromEventPattern<MouseButtonEventArgs>(
           this, "MouseLeftButtonUp");
         var mousemove = Observable.FromEventPattern<MouseEventArgs>(
           this, "MouseMove")
                 .Select(evt => evt.EventArgs.GetPosition(this));
         var drag = mousedown.SelectMany(start => mousemove.TakeUntil(mouseup),
           (start, end) => new
         {
             X = end.X - start.X,
             Y = end.Y - start.Y
         });
         mousedown.Subscribe(p =>
         {
         });
         mouseup.Subscribe(e =>
         {
         });
         mousemove.Subscribe(p =>
         {
         });
         drag.Subscribe(value =>
         {
             Canvas.SetLeft(Image, value.X);
             Canvas.SetTop(Image, value.Y);
         });
   }

As assinaturas não afetam o resultado, são apenas pontos de análise do que está acontecendo. Coloque um breakpoint ao final do método Subscribe de MouseMove:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/8.jpg">
<br>
</p>

Então, execute o programa. Quando você inicia o movimento do mouse sobre a janela, o breakpoint será ativado. Então, usando a varinha mágica, crie um Tracepoint aqui:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/9.jpg">
<br>
</p>

Não queremos listar nada de especial aqui, apenas o nome do evento, Mouse Move. Você poderia também listar valores de variáveis usando chaves (como em {variavel}), mas neste caso queremos apenas saber quando o evento é observado e apenas o string é suficiente:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/10.jpg">
<br>
</p>

Faça o mesmo para o final de cada assinatura (incluindo a assinatura de drag) e execute o programa. Após terminar a execução, vá para a janela de Tracepoints, clicando no número de eventos de trace em baixo da janela do Visual Studio:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/11.jpg">
<br>
</p>

Quando você olha para a janela, verá algo como o seguinte:

<p align="center">
<img src="http://lab27.blob.core.windows.net/wordpress/2017/02/12.jpg">
<br>
</p>

Como você pode ver, há alguns MouseMove. Então acontece um MouseDown, que iniciará a emitir o Observable Drag para cada MouseMove. Se você for mais para baixo no Trace, você verá que, quando um MouseUp ocorre, o drag termina, até que ocorra um MouseDown novamente.

Legal, não? Com poucas linhas de código, pudemos fazer um programa para arrastar uma imagem em uma aplicação WPF. Pudemos combinar eventos usando LINQ, observar o resultado e mover a imagem.

Com esta introdução à programação reativa e sabendo o que pode ser feito com os observables, podemos ir à segunda parte desta série, onde veremos como usar os Observables para criar um programa usando MVVM, com Reactive UI. Até lá.


    







