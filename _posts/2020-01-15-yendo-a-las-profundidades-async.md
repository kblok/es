---
title: Yendo a las profundidades de la programación asyncrónica en C#
tags: csharp books
permalink: /blog/yendo-a-las-profundidades-async
---

_Don't speak Spanish? Checkout the [English version](https://www.hardkoded.com/blog/going-deeper-async)_

Cuando empezás a hacer applicaciones asincrónicas en C# todo parece muy simple, casi mágico. Ponés un async acá algunos awaits allá y listo!

![bob loves async](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/deeper-async/bob-loves-async.jpg)

Pero luego, cuando empezás a usar asyncs más y más, empiezan los problemas. 

![nuevo lenguaje](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/deeper-async/newlanguage.jpg)

Como no conocés muy bien la tecnología empezás a adoptar reglas, reglas que no entendés, pero que seguís. Encontrás estas reglas en StackOverflow (preferiblemente en las respuestas y no en preguntas!) o en blogs. Entonces empezas a crear tu propia versión de los diez mandamientos:

![commandments](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/deeper-async/ten-commandments.jpg)

 * Usarás `ConfigureAwait(false)`.
 * No usarás `async void`.
 * No usarás `.Result`.
 * Usarás `GetAwaiter().GetResult()` en lugar de `.Result`.
 * Deberás retornar siempre una `Task`.
 * etc.

No hay nada malo en seguir reglas. Pero yo me cansé de ellás. Quería tener una explicación para cada una. Porque, al final y al cabo, si queres dominar una tecnología, tenés que ir más profundo. Entonces decidí empezar a leer más sobre programación asincrónica. Tratando de encontrar respuestas a preguntas como _”¿Qué demonios sigifica el false en la llamada a ConfigureAwait?”_ o _”¿Por qué necesito llamar a GetAwaiter()?”_, o aún mejor _”¿Qué demonios es un awaiter?”_, _”¿SyncronizationContext verdad o mito?”_, _”¿Si una Task falla en el bosque y nadie está ahí, emite un sonido?”_… ok tal vez no esa.

Así que agregué estos tres libros a mi lista de lectura:
 * [Concurrency in C# Cookbook, Stephen Cleary](https://www.amazon.com/gp/product/B00KCY2CB4). [Go to the review](https://www.hardkoded.com/blog/concurrency-cookbook-review)
 * [Async in C# 5.0, Alex Davies](https://www.amazon.com/gp/product/B0099BJ4DU)
 * [Pro Asynchronous Programming with .NET](https://www.amazon.com/gp/product/B00I01FWGS)

En las próximas semanas voy a estar subiendo las lecciones aprendidas de cada libro, así que te invito a esta aventura asincrónica.

¡No dejes de codear!


