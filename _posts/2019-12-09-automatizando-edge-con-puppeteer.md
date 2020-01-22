---
title: Automatizando Microsoft Edge con Puppeteer/Puppeteer-Sharp
tags: puppeteer-sharp csharp
permalink: /blog/automatizando-edge-con-puppeteer
---

Posiblemente hayas leído que Microsoft hizo pública la versión beta de su nuevo [Microsoft Edge](https://www.microsoftedgeinsider.com/es-es/), apodado Edgium, ya que está basado en Chromium. Tal vez esta no sea una buena pregunta para una entrevista técnica pero:

>Si Puppeteer/Puppeteer-Sharp automatiza Chromium, y este nuevo Microsoft Edge está basado en Chromium, estoy podría significar que...

![Idea](https://media.giphy.com/media/B5AVgxf0OzlyE/giphy.gif)

Cuando llamás a `Puppeteer.LaunchAsync`, unas de las opciones que se pueden configurar en las `LaunchOptions` es el [ExecutablePath](https://www.puppeteersharp.com/api/PuppeteerSharp.LaunchOptions.html#PuppeteerSharp_LaunchOptions_ExecutablePath).

Entonces, ¿Qué pasaría si instanciamos Puppeteer/Puppeteer-Sharp pasándole el path de este nuevo Microsoft Edge?

```cs
var browser = await Puppeteer.LaunchAsync(new LaunchOptions
{
    Headless = true,
    ExecutablePath = "C:\\Program Files (x86)\\Microsoft\\Edge Dev\\Application\\msedge.exe"
});
```

Vamos a escribir una app super simple:

```cs
var browserOptions = new LaunchOptions
{
    Headless = false,
    ExecutablePath = "C:\\Program Files (x86)\\Microsoft\\Edge Dev\\Application\\msedge.exe"
};

var browser = await Puppeteer.LaunchAsync(browserOptions);
var page = await browser.NewPageAsync();

await page.SetContentAsync("<div>Testing</div>");
```

Voilà!

![demo running](https://github.com/kblok/kblok.github.io/raw/master/img/microsoft-edge-puppeteer/demo-running.png)

Intentemos correr todos los tests de Puppeteer-Sharp usando Microsoft Edge.

![tests running](https://github.com/kblok/kblok.github.io/raw/master/img/microsoft-edge-puppeteer/test.png)

Ok, casi casi. De los 691 tests sólo 6 están fallando. Lo bueno es que el equipo de Edge [se puso a disposición para revisar estas fallas](https://twitter.com/EdgeDevTools/status/1201978063015333889). Esperemos que podamos ver todo en verde en las próximas versiones :)

![Yeah](https://media1.giphy.com/media/TdfyKrN7HGTIY/giphy.gif?cid=790b76115cbda5016531733341f0d371)

¡No dejes de codear!


