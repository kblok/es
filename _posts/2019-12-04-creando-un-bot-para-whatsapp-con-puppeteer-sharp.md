---
title: Creando bot para WhatsApp con Puppeteer-Sharp
tags: puppeteer-sharp csharp
permalink: /blog/creando-un-bot-para-whatsapp-con-puppeteer-sharp
---

_Don't speak Spanish? Checkout the [English version](https://www.hardkoded.com/blog/creating-whatsapp-bot-puppteer-sharp)_

# Había una vez

[Not enough friends? Get a bot!](https://www.hardkoded.com/blogs/not-enough-friends-get-a-bot) fue mi segundo post en inglés. En esos días, mientras estaba aprendiendo algo (alguito) de Machine Learning, ví que muchos devs estaban usando [Marcovify](https://github.com/jsvine/markovify) y pensé que sería divertido hacer un bot para que hablara con mis amigos. Pero encontré que crear un bot para WhatsApp no era tan simple. Necesitabas configurar una [cuenta de negocios](https://www.whatsapp.com/business/), y pagar por ella. Entonces lo hice en Telegram.

Pero hay un problema con Telegram: **No todos lo usan como su aplicación de mensajería principal**. Todos amamos Telegram, pero usamos WhatsApp...

Entonces hice un bot en Telegram, lo publiqué en un docker local, y desde entonces vamos cada tanto a Telegram para divertirnos un poco con nuestro bot.

# Un dev automatizando VS Code

Hace unos días encontré un video donde [Jarrod Overson](https://twitter.com/jsoverson) estaba automatizando VS Code usando Puppeteer!! Fabuloso!!  

<iframe width="560" height="315" src="https://www.youtube.com/embed/VDGiQ2cwFP4?start=500" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Esta fue mi cara después de mirar el video:
![Idea](https://media.giphy.com/media/B5AVgxf0OzlyE/giphy.gif)

>Si el fue capaz de automatizar VS Code porque la app estaba hecha en Electron, ¿No deberíamos entonces ser capaces de automatizar la app de WhatsApp?

Pero después me acordé que...

>No necesitamos hackear Electron, WhatsApp tiene una aplicación web!

¡Manos a la obra!

# El bot

Queremos crear un chatbot muy simple, Tiene que esperar por un “disparador”, una palabra clave, y tratar de responder algo.
Veamos la página de WhatsApp.

![main page](https://github.com/kblok/kblok.github.io/raw/master/img/whatsappbot/main-whatsapp.png)

Esto es lo que necesitamos hacer:
 * Abrir esa página.
 * Buscar a un grupo o a una persona.
 * Seleccionar a ese grupo o persona.
 * Empezar a escuchar los mensajes.
 * Escribir y enviar el mensaje cuando se detecte un disparador.

# Explorando la página

Necesitamos saber cómo buscar un grupo, hacer click en ese grupo y escribir un mensaje. DevTools va a ser nuestro mejor amigo para esta tarea.  

![DevTools](https://github.com/kblok/kblok.github.io/raw/master/img/whatsappbot/devtools.png)

Después de explorar el DOM, encontramos que:
 * Toda la página está dentro de un div `#pane-side`.
 * El input de búsqueda tiene una clase `jN-F5`.
 * Cada persona o grupo en la lista tiene una clase  `_2wP_Y`.
 * El input para ingresar un mensaje es un DIV editable con la clase  `_2S1VP`.
 * El botón para enviar un mensaje tiene la clase `_35EW6`.
 * Hay un DIV que contiene todos los mensajes y tiene la clase `_9tCEa`.
 * Cada línea en el chat es un DIV con la clase `vW7d1`.

# ¡A codear!

Vamos a crear una aplicación de consola común y corriente, usando el paquete de NuGet de Puppeteer-Sharp ¡Por supuesto!

Antes que nada, vamos a crear una clase para poner ahí todos selectores CSS que encontramos en la sección anterior.

```cs 
internal class WhatsAppMetadata
{
    public const string WhatsAppURL = "https://web.whatsapp.com/";
    public const string MainPanel = "#pane-side";
    public const string SearchInput = ".jN-F5";
    public const string PersonItem = "._2wP_Y";
    public const string MessageLine = "vW7d1";
    public static string ChatContainer = "._9tCEa";
    public static string ChatInput = "._2S1VP";
    public static string SendMessageButton = "._35EW6";
}
```

Bien, recordemos el To-Do que habíamos hecho antes:
 * Abrir esa página.
 * Buscar a un grupo o a una persona.
 * Seleccionar a ese grupo o persona.
 * Empezar a escuchar los mensajes.
 * Tipear y enviar en mensaje cuando se detecte un disparador.

## Abrir la página

Primero necesitamos un browser.

```cs
await new BrowserFetcher().DownloadAsync(BrowserFetcher.DefaultRevision);
_browser = await Puppeteer.LaunchAsync(new LaunchOptions
{
    UserDataDir = Path.Combine(".", "user-data-dir"),
    Headless = false
});
```

También necesitamos configurar el `UserDataDir` para almacenar ahí toda nuestra información, como cookies, localStorage, etc.

Vamos a configurar el `Headless` en `false` más que nada porque necesitamos escanear el código QR con nuestro teléfono la primera vez. Podríamos ponerlo en `true` después. 

Bien, vamos a navegar la página..

```cs
_whatsAppPage = await _browser.NewPageAsync();
await _whatsAppPage.GoToAsync(WhatsAppMetadata.WhatsAppURL);
await _whatsAppPage.WaitForSelectorAsync(WhatsAppMetadata.MainPanel);
```

`.WaitForSelectorAsync(WhatsAppMetadata.MainPanel);` va a esperar hasta que el sitio esté cargado, este va a ser el momento de escanear el código QR en caso de ser necesario.

## CommandLineParser

Antes de empezar a buscar personas o grupos, vamos a agregar [CommanLineParser](https://github.com/commandlineparser/commandline) a nuestro proyecto. Me encanta este proyecto porque valida todo los argumentos que recibimos por línea de comandos y luego crea una instance de una clase basada en esos argumentos.

Esta es la clase BotArguments:

```cs
public class BotArguments
{
    [Option('t', "trigger", Required = true, HelpText = "Trigger word.")]
    public string TriggerWord { get; set; }
    [Option('c', "chat", Required = true, HelpText = "Chat name.")]
    public string ChatName { get; set; }
    [Option('r', "response", Required = true, HelpText = "Response template.")]
    public string ResponseTemplate { get; set; }
    [Option('l', "language", Required = true, HelpText = "Language.")]
    public string Language { get; set; }
    [Option('f', "file", Required = true, HelpText = "Source text file.")]
    public string SourceText { get; set; }
}
```

Y así es como se ve ahora el método `Main`:

```cs
static async Task Main(string[] args)
{
    await Parser.Default.ParseArguments<BotArguments>(args).MapResult(
        async (BotArguments result) => await LaunchProcessAsync(result),
        _ => Task.FromResult<object>(null));
}
```

Ahora que tenemos nuestra clase BotArgument volvamos a nuestra app.

## Buscar una persona o grupo

Basándonos en la persona que recibimos como argumento, nosotros podemos hacer esto:

```cs
var input = await _whatsAppPage.QuerySelectorAsync(WhatsAppMetadata.SearchInput);
await input.TypeAsync(args.ChatName);
await _whatsAppPage.WaitForTimeoutAsync(500);
```

¡Fabuloso!
Buscamos un elemento usando `QuerySelectorAsync`, escribimos en ese elemento, y luego esperamos un poco para que el DOM se actualice.

## Seleccionar el elemento

Si asumimos que la persona que estamos buscando va a aparecer primero, vamos a estar seguros que esa persona va a ser el segundo elemento en esa lista, porque “CHATS” va a ser nuestro primer elemento.

![Primer elemento](https://github.com/kblok/kblok.github.io/raw/master/img/whatsappbot/first-item.png)

Ahora que sabemos esto, podemos hacer:

```cs
var menuItem = (await _whatsAppPage.QuerySelectorAllAsync(WhatsAppMetadata.PersonItem)).ElementAt(1);
await menuItem.ClickAsync();
```

Hacemos un `QuerySelectorAllAsync` de todos los elementos en la lista, y nos quedamos con el segundo.

## Empezar a escuchar mensajes

Empieza lo divertido, y creo que es algo que vas a querer aprender.
**¿Cómo hacemos para escuchar nuevos mensajes?**

Vamos a necesitar dos cosas:
Una función que funcione como callback de nuestro lado.
Un DOM observer del lado del browser, con la capacidad de llamar a nuestra función

### ExposeFunctionAsync al rescate

`ExposeFunctionAsync` nos permite registrar un método C# del lado de Chromium.

![What?](https://media3.giphy.com/media/lsU7mOh76j4QM/giphy.gif?cid=790b76115caf2a587652616e2e96ddfa)

```cs
await _whatsAppPage.ExposeFunctionAsync("newChat", async (string text) =>
{
    Console.WriteLine(text);

    if (text.ToLower().Contains(args.TriggerWord) && !text.Contains(args.ResponseTemplate))
    {
        await RespondAsync(args, text);
    }

    text = text.Replace(args.ResponseTemplate, string.Empty);
    await File.AppendAllTextAsync(args.SourceText, text + "\n");
});
```
¡Perfecto!
Ahora tenemos una nueva función llamada `newChat` en Javascript.
Cuando alguien (o algo) llama a `newChat` nosotros vamos a:
Loguear el mensaje.
Verificar que el mensaje contenga el disparador (la palabra clave).
Verificar que el mensaje no contenga nuestro template de respuesta, básicamente no queremos escuchar lo que nosotros mismos enviamos.
Las últimas líneas no son tan importantes por ahora, pero lo que hacen es guardar el mensaje en un archivo, cosa que podamos tener más datos para generar nuevos mensajes en el futuro. 

### Escuchando nuevos mensajes

Si `ExposeFunctionAsync` es nuestro mejor amigo del lado de C# , [MutatorObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) va a ser nuestro mejor amigo del lado de Javascript/Chromium.

```cs
await _whatsAppPage.EvaluateFunctionAsync($@"() => {
    var observer = new MutationObserver((mutations) => {
        for(var mutation of mutations) {
            if(mutation.addedNodes.length &&
               mutation.addedNodes[0].classList.value === '{WhatsAppMetadata.MessageLine}') {
                newChat(mutation.addedNodes[0].querySelector('.copyable-text span').innerText);
            }
        }
    });
    observer.observe(
        document.querySelector('{WhatsAppMetadata.ChatContainer}'),
        {{ attributes: false, childList: true, subtree: true }});
}");
}
```
_Nota: El [código real](https://github.com/kblok/WhatsAppBot/blob/master/WhatsAppBot/Program.cs#L72) tiene doble llaves. Las tuve que remover ~~porque rompen a jekyll~~ para que sea más claro._

Lo que estamos haciendo ahí es observando cambios en `childList`de nuestro elemento `WhatsAppMetadata.ChatContainer`. Dentro de este observer, vamos a filtrar los ítems que tengan la clase que está en nuestra constante `MessageLine`.
Si tenemos un match, vamos a llamar a `newChat` enviando el innerText de dicho elemento.

## Escribiendo un mensaje de respuesta.

[MarkovSharp](https://github.com/chriscore/MarkovSharp) puede ayudarnos a divertiros un poco y construir algunas respuestas basadas en los chats que tenemos exportados.

Configurar `MarkovSharp` es tan fácil como esto:

```cs
var chat = await File.ReadAllLinesAsync(args.SourceText);
_model = new StringMarkov(5);
_model.Learn(chat);
```

Entonces nuestro `RespondAsync` podría hacer lo siguiente:

Vamos a procesar el mensaje que recibimos y vamos a quedarnos con una lista curada de palabras. Para ellos vamos a usar [dotnet-stop-words](https://github.com/hklemp/dotnet-stop-words).

```cs
string response = null;
var words = text.RemoveStopWords(args.Language).RemovePunctuation().Replace(args.TriggerWord, string.Empty).Split(' ');
```

Ahora vamos a recorrer la lista de palabras en orden inverso, tratando de encontrar un mensaje válido de nuestro modelo de Markov.

>La idea acá es la siguiente: Si recibimos un mensaje como “Hey este bot chat es genial!”. Vamos a obtener como válidas las palabras [bot, chat, genial], y vamos a tratar de hacer un mensaje basándonos, primero en “genial”, luego en “chat” y por último en “bot”.

```cs
for (var index = words.Length - 1; index >= 0; index--)
{
    response = _model.Walk(1, words[index]).First();

    if (response == words[index])
    {
        response = null;
    }
}

if (response == null)
{
    response = _model.Walk(1).First();
}
````

Una vez que tengamos un mensaje “divertido”. Lo vamos a enviar como respuesta.

```cs
await WriteChatAsync(args.ResponseTemplate + " " + response);
```

Nuestro método sería algo así:

```cs
var chatInput = await _whatsAppPage.QuerySelectorAsync(WhatsAppMetadata.ChatInput);
await chatInput.TypeAsync(text);
await (await _whatsAppPage.QuerySelectorAsync(WhatsAppMetadata.SendMessageButton)).ClickAsync();
```

¡Genial! ¡Tenemos un bot!

# Palabras finales

Espero que hayas disfrutado este tutorial. Vas a encontrar [el repo con el código en Github](https://github.com/kblok/WhatsAppBot)
La idea de este post no era solamente mostrar este bot, pero también mostrarte algunas técnicas y herramientas que podés usar para automatizar un browser. 

¡No dejes de codear!
