---
title: Una historia sobre async voids, eventos y manejo de errores
tags: puppeteer-sharp csharp
permalink: /blog/cuento-async-void
cross-site-link: https://www.hardkoded.com/blog/async-void-fairy-tale
---
 
Dejame contarte una historia sobre async voids, SynchronizationContext y programación asincrónica.  
Hace un tiempo recibí [un issue en Puppeteer-Sharp](https://github.com/hardkoded/puppeteer-sharp/issues/717) describiendo dos problemas:
Puppeteer-Sharp crasheaba con excepciones que no podían ser atrapadas.
Se reportaba un KeyNotFoundException tratando de obtener un [Frame](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Frame.cs). 

El código era bastante simple:

```cs
var launchOptions = new LaunchOptions() { Headless = true };
var sites = new List<string>()
{
    "somesites.com",
}
// Act
try
{
    await new BrowserFetcher().DownloadAsync(BrowserFetcher.DefaultRevision);
    using (var browser = await Puppeteer.LaunchAsync(launchOptions))
    {
        var page = await browser.NewPageAsync();

        foreach (var site in sites)
        {
            try
            {
                await page.GoToAsync($"http://{site}");
                Console.WriteLine(await page.GetTitleAsync());
                await page.ScreenshotAsync($"D:\\bin\\screenshots\\{site}.png");
            }
            catch (Exception exception)
            {
                // Catches most exceptions such as timeouts but does not catch others, see below.
                Console.WriteLine($"Unable to take screenshot of: {site}. Exception: {exception.Message}");
            }
        }
    }
}
catch (Exception exception)
{
    // Never enters the catch.
    Console.WriteLine($"Unable to proceed: {exception.Message}");
}
```

## ¿Cómo era posible que un bloque try-catch no atrape una excepción?

Es imposible, simplemente imposible. Para eso son los try-catch ¿no?
Bueno, [Ben Adams](https://twitter.com/ben_a_adams) me dió una pista [aquí](https://github.com/dotnet/roslyn/issues/13897#issuecomment-248098377).

![benadamscomment.png](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/async-void-fairy-tale/benadamscomment.png)

>Si se ignora lo que retorna una función `async Task`, una excepción no manejada dentro de dicha función aparecerá en el `TaskScheduler.UnobservedTaskException`. Pero si lo mismo suceder en una función `async void`, esta terminará el proceso.

Creo que la mayoría de nosotros leyó al menos una vez esta regla:

>¡No uses async void! Uno puede tener un método async void, pero sólo deberías usarlo si estás escribiendo un event handler async. Una función async normal debería siempre retornar una Task, nunca void. 

_[Cleary, Stephen. Concurrency in C# Cookbook](https://www.amazon.com/dp/B00KCY2CB4)_

El tema es que a veces nos encontramos en una situación donde tenemos que escribir un event handler y hacerlo asincrónico. Entonces tenemos la excusa perfecta: “Se que usar async void está mal. Pero tengo que hacer un event handler, no me queda otra. Estoy siguiendo las reglas, todo va a estar bien”

Bueeeeno…. no.


>Cuando un método `async void` propaga una excepción, esa excepción es lanzada al SynchronizationContext que estaba activo en el momento que el método `async void` comenzó su ejecución. Si tu entorno provee un SyncronizationContext, este generalmente tiene una forma de manejar estas excepciones a nivel global, por ejemplo, WPF tiene un Application.DispatcherUnhandledException, WinRT tiene un Application.UnhandledException y ASP.NET tiene un Application_Error.

_[Cleary, Stephen. Concurrency in C# Cookbook](https://www.amazon.com/dp/B00KCY2CB4)_

Pero, como Ben dijo, en una aplicación de consola estas excepciones van a ser propagadas hasta el ThreadPool sin ser atrapadas, terminando el proceso.

## ¿Dónde están estos async voids?

Debes estar pensando: “Ok, pero en tu código de ejemplo no hay async voids, ¿Dónde están esos famosos async voids?

Bueno, resulta ser que hay mucho eventos dentro de Puppeteer que estaban siendo manejados usando async voids. 
Cuando un nuevo mensaje llega desde Chromium, un [IConnectionTransport](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Transport/IConnectionTransport.cs) evalúa el mensaje y lo propaga usando el evento [MessageReceived event](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Transport/IConnectionTransport.cs#L32). Muchas clases como [Page](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Page.cs#L2082) escuchan estos mensajes y realizan tareas asincrónicas.

Y aquí es donde encontramos nuestros **async voids**.
Fin de la historia.

No mentira, aún no terminamos...

## ¿Son los async voids el verdadero problema?

Me obsesioné con este tema de los async voids. Incluso consideré reemplazar estos eventos con otras herramientas como [DataFlows](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/dataflow-task-parallel-library) o [System.Reactive](https://github.com/dotnet/reactive), pero no encontré ninguna solución que me pareciese la correcta.

Cuando terminé de leer [Concurrency C# cookbook](http://shop.oreilly.com/product/0636920030171.do), decidí contactar a [Steve Cleary](https://twitter.com/aSteveCleary) en Twitter. El, amablemente respondió a mi tweet, y para mi sorpresa, esto fue lo que dijo en medio de la conversación:

![benadamscomment.png](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/async-void-fairy-tale/stevetweet.png)

>¿Por qué querés eliminar los eventos async void?

Yo me quedé así:

![What](https://media.giphy.com/media/91fEJqgdsnu4E/giphy.gif)

¿Cómo es posible que la persona que más sabe sobre programación asincrónica me diga eso?
¿No era que `async void` era la raíz de todos los males?

Pero esta situación me ayudó a comprender que no estaba entendiendo muy bien cual era el problema. Vamos ver bien qué sucede cuando tenemos un código como este:

```cs
try
{
    using (var browser = await Puppeteer.LaunchAsync(options))
    using (var page = await browser.NewPageAsync())
    {
            await Page.ClickAsync("body");
    }
}
catch(Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

Si hacemos un diagrama simplificado, sería algo así:

![diagram](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/async-void-fairy-tale/puppeteerworkflow.png)

Si nosotros fallamos al intentar procesar ‘Target.targetCreated`. ¿Qué línea de código fallaría?
Fácil, `browser.NewPageAsync`.
Pero ¿Qué pasaría si fallamos al intentar procesar `Target.targetInfoChanged`? No hay forma que podamos enviar esa excepción al usuario, simplemente por el hecho que el usuario no disparó esa acción.

Esto fue un problema bastante feo para Puppeteer-Sharp, **no poder comunicar errores internos al usuario**.

>Si tu libraría consume eventos desde algún otro origen, tenés que ser muy cuidadoso al momento de manejar las excepciones y diseñar tu API de manera tal que esas excepciones se comuniquen de alguna manera.

## ¿Cómo lo resolví?

Puppeteer-Sharp tiene dos tipos de conexiones. Una conexión general por browser y una para cada uno de los targets (tabs de un browser).
Lo primero que hice fue agregar un `try-catch- en cada `MessageReceived` que tenía en la librería. Entonces, en caso de capturar una excepción, lo que hice fue cerrar la conexión y agregar un `close reason` a la misma.

```cs
private async void Client_MessageReceived(object sender, MessageEventArgs e)
{
    try
    {
        //Message Processing
    }
    catch (Exception ex)
    {
        var message = $"NetworkManager failed to process {e.MessageID}. {ex.Message}. {ex.StackTrace}";
        _logger.LogError(ex, message);
        _client.Close(message);
    }
} 
```

Próximo issue.

## KeyNotFoundException tratando de obtener un [Frame](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Frame.cs) … Qué?


Si volvemos a observar nuestro diagrama de secuencias, vamos a poder ver que el primer mensaje que obtenemos de Chromium es `Target.targetCreated`. ¿Cómo es posible que recibamos un KeyNotFoundException?

Te voy a dar una pista, comienza con `async` y termina con `void`.

Un [IConnectionTransport](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/Transport/IConnectionTransport.cs) va a comenzar a recibir un stream de mensajes de Chromium, los va a parsear y luego va a emitir el evento `MessageReceived`.

El problema acá es que el `MessageReceived?.Invoke` va a actuar como un “fire and forget” si el event handler es un `async void` (se llama “fire and forget” cuando uno ejecuta una tarea asincrónica y no espera a que esta finalice).
**Por supuesto!** no hay ningún `await` ahi. Ese evento **va a ser** un fire and forget.

Ahora tiene sentido cuando uno lee esto en [Using Asynchronous Methods in ASP.NET 4.5 post](https://docs.microsoft.com/en-us/aspnet/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45)

>El problema con los eventos async void es que los developers pierden el control del orden de los eventos. Por ejemplo, si una página aspx y su .Master definen un Page_Load, y alguno de ellos es asyc, el orden de ejecución no puede ser garantizado. Lo mismo aplica con los eventos que siguen, como por ejemplo un Button_Click.

Por supuesto, el orden de ejecución no puede ser garantizado, porque **el método `invoke` no espera los event handlers asíncronos.

## Esperando que los frames sean creados.

Ahora que entendemos la situación, el problema con los frames va a ser fácil de resolver. Será cuestión de reemplazar todos los:

```cs
Frames[someFrame];
```

Con

```cs
await _frameManager.GetFrameAsync(someFrame);
```

# Palabras finales

Si bien este post tiene muchas cosas muy específicas de Puppeteer-Sharp, creo que puede ofrecer algunos conceptos interesantes al momento de consumir eventos en forma asincrónica, y cómo resolverlo en tu librería.

¡No dejes de codear!
