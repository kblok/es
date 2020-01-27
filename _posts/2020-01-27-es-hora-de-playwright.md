---
title: ¡Es hora de Playwright!
tags: playwright
permalink: /blog/es-hora-de-playwright
---

Si usás puppeteer o puppeteer-sharp seguro que debes estar tan emocionado como yo con el anuncio de [Playwright](https://github.com/microsoft/playwright). Playwright va a llevar la automatización de browsers a un nuevo nivel.

_Descripción del proyecto en GitHub_
> Playwright está enfocado en la automatización multi-browser y multi-plataforma, de una manera confiable, completa y veloz. Nuestro primer objetivo con Playwright es mejorar el testing de UI automatizado, eliminando inestabilidades y mejorando los tiempos de ejecución, ofreciendo detalles sobre el browser que se está utilizando.

El [equipo](https://github.com/microsoft/playwright/graphs/contributors) que tienen es impresionante, no veo la hora de ver todo lo que van a hacer con este proyecto.

Ahora vamos a lo importante ¿Vamos a tener un Playwright**Sharp**?

![yes](https://media2.giphy.com/media/10Jpr9KSaXLchW/giphy.gif?cid=790b7611f42ab6c5c6cf35c80cf2a14aff294637a1a6ef38&rid=giphy.gif)

Si, vamos a hacer un port a .NET de Playwright (esta oración está agregada simplemente para mejorar el SEO del post).

Ya tenemos lo más importante:

 * Un repo [https://github.com/kblok/playwright-sharp](https://github.com/kblok/playwright-sharp)
 * Un paquete NuGet [https://www.nuget.org/packages/PlaywrightSharp/](https://github.com/kblok/playwright-sharp). Aunque seguramente vamos a tener varios paquetes (un paquete de abstracción, y varios paquetes por browser).
 * Un dominio [http://www.playwrightsharp.com/](http://www.playwrightsharp.com/) (ni siquiera te molestes en hacer click en el link).
 * ¡Y un plan! [https://github.com/kblok/playwright-sharp/projects/1](https://github.com/kblok/playwright-sharp/projects/1)

# Filosofía del proyecto

Así como hicimos con Puppeteer-Sharp, el objetivo principal es crear una API lo más cercana posible a Playwright. Un developer tiene que ser capaz de moverse facilmente entre Playwright y PlaywrightSharp.

Teniendo en claro ese objetivo principal, Playwright Sharp tiene que tener sabor a .NET/C# (creo que suena horrible en español, tiene que tener un .NET flavor). Un developer tiene que ser capaz de inyectar los objetivos usando un inyector de dependencias, las funciones getters de Playwright tienen que ser traducidas como propiedades, Los métodos asyncs van a tener el sufijo Async, etc, etc.

No vamos (tecnicamente hablando) a respetar [semver](https://semver.org/). Siempre vamos a tratar de que una versión de PlaywrightSharp tenga los mismos features que la misma versión en Playwright.

# La arquitectura

Si bien esto no está definido aún, PlaywrightSharp va a ser mas o menos así:

 * **PlaywrightSharp**, con las clases base. 
 * **PlaywrightSharp.Abstractions**, with interfaces, usando el namespace `Playwright`.
 * **PlaywrightSharp._#BrowserEngine#_**. Paquetes específicos para cada browser. Por ejemplo PlaywrightSharp.Chromium.

# El plan

Alcanzar a un proyecto activo es todo un desafío. Siempre estás un paso atrás. Pero, si una vez fuimos capaces de alcanzar a Puppeteer, vamos a ser capaces de alcanzar a Playwright.

Ya tenemos nuestro [primer proyecto](https://github.com/kblok/playwright-sharp/projects/1). 157 issues que nos van a ayudar a llegar al tag [first-snapshot](https://github.com/kblok/playwright/tree/first-snapshot) que creé el 25 de enero.

Como [aprendí en el pasado](http://www.hardkoded.com/blogs/how-to-start-an-oss-project), los tests van a ser nuestro faro. Van a hacer lo primero que implementemos (bueno, técnicamente lo segundo), y van a ser nuestra guia para saber que componentes tenemos que implementar.

# Palabras finales

Siempre hablo del “poder de la estrella”. Esas estrellas en GitHub no significan mucho para algunos, para para los desarrolladores de las librerías es una linda palmada en la espalda. Así que si estás interesado en este proyecto, una estrella [en el repo](https://github.com/kblok/playwright-sharp) me va a robar una sonrisa :)

¡No dejes de codear!

