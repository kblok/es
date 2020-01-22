---
title: Review del libro Concurrency in C#
tags: csharp books
permalink: /blog/concurrency-cookbook-review
---

[Concurrency in C# Cookbook](https://www.amazon.com/gp/product/B00KCY2CB4) de [Steve Cleary](https://twitter.com/aSteveCleary) es el primero de una serie de libros que [me propuse leer para aprender más sobre programación asincrónica](https://www.hardkoded.com/es/blog/yendo-a-las-profundidades-async).

Si no conoces a Steve Cleary te cuento, es un rock star del async. Su blog tiene unos [posts sobre programación asincrónica geniales](https://blog.stephencleary.com/2012/02/async-and-await.html), y su [reputación en StackOverflow](https://stackoverflow.com/users/263693/stephen-cleary) es impresionante. Si alguna vez hiciste, o buscaste, alguna pregunta sobre asyncs en StackOverflow, hay muchas chances que hayas leído unas de sus respuestas.

Con esta reputación, yo sabía que [Concurrency in C# Cookbook](https://www.amazon.com/gp/product/B00KCY2CB4) tenía que ser mi primer libro. 
El libro es un típico cookbook (libro de recetas), donde cada capítulo introduce un concepto y es seguido por una lista de recetas.

A grandes rasgos, el libro cubre tres tópicos:
 * Async/Parallel programming.
 * DataFlows
 * Rx (System.Reactive)

Si bien solo estaba interesado en la sección de Async programming, aprender un poco sobre DataFlows y Rx no estuvo nada mal.

# Programación asincrónica vs procesamiento en paralelo

Me encantó el primer capítulo. Este capítulo habla de cosas simples pero muy importantes a la vez. Él explica conceptos como multithreading, procesamiento en paralelo, programación asincrónica y programación reactiva (no se si así se traduce reactive programming). Tener en claro estos conceptos es super importante porque es una trampa en la que muchos (me incluyo) caemos/caímos. **La trampa está en pensar que estamos haciendo procesamiento en paralelo cuando en realidad estamos haciendo programación asincrónica**. Esta trampa se debe a que .NET usa la misma clase, la clase `Task` para trabajar tanto en paralelo como en programación asincrónica.

> La trampa es pensar que estamos haciendo procesamiento en paralelo cuando en realidad estamos haciendo programación asincrónica.

También nos recuerda que:

> Programación asincrónica es una forma de concurrencia que usa futuros (Futures) or Callbacks para evitar threads innecesarios.

Otros concepto muy interesante en este capítulo es que la programación asincrónica no se trata de ser más veloces sino de mejorar la escalabilidad y el responsiveness (les debo la traducción).

> Programación asincrónica no se trata de velocidad sino de escalabilidad y responsiveness

# Introducción a la programación sincrónica

Hay muchos conceptos importantes en este capítulo. El primero, y el más importante, es que los métodos no son asíncronos, la clase `Task` es asíncrona. El hecho que un método tenga el modificador `async` no lo hace asíncrono, el retornar una `Task` (una promesa) es lo que lo hace asíncrono.

> Los métodos no son asíncronos, la clase `Task` lo es.

Otro concepto interesante es que una llamada a un método async va a correr en forma sincrónica hasta que llegue al primer `await` de una Task incompleta. Esto lo pude probar [en este pequeño proyecto](https://github.com/kblok/async-programming-talk/blob/master/AwaitDemo/Program.cs). 
Entonces si usás `await Task.WhenAny`, podrías pensar que las Tasks que le pasas al `WhenAny` van a correr en paralelo, bueno… no.

> Las llamadas a métodos async van a ser sincrónicas hasta que lleguen al primer await de una Task incompleta.

# Synchronization Context

Leer sobre Synchronization Context era uno de mis objetivos, porque es algo que uno no ve, pero está ahí. Tus librerías pueden llegar a tener problemas en su comportamiento sino contemplás los diferentes synchronizations contexts.

> Tus librerías pueden llegar a tener problemas en su comportamiento sino contemplar los diferentes synchronizations contexts.

Encontré una explicación un poco más en profundidad en otro libro, ya vamos a tener un post, no se lo pierdan. Pero Steve nos da un consejo muy importante:

> Es una buena práctica llamar siempre al `ConfigureAwait` en los métodos de tus librerías, y solo capturar el contexto cuando lo necesitás, en los métodos que están en la interfase de usuario.

[Uno de los mandamientos](http://www.hardkoded.com/es/blog/yendo-a-las-profundidades-async) era acerca de usar `GetAwaiter().GetResult()`, pero a todo esto, ¿Qué es un awaiter?

Resulta ser que el keyword `await` funciona no solo con una Task sino con cualquier “awaitable” que siga un determinado patrón. Encontré una buena explicación sobre el keywork `await` en otro libro (TODO: Darío insertar acá el link al próximo post).

Hay muchas cosas que nosotros damos por sentado cuando usamos async/await. Una de ellas es el manejo de errores.

> Cuando un método async arroja o propaga una excepción, la excepción es asignada a la Task que se está retornando, marcando como `Faulted`, y por lo tanto `IsCompleted`, a la misma. Cuando se le hace un await a esa task, el operador await va a ver dicha excepción y la va a (re)lanzar manteniendo el stack trace original.

Vamos a hablar más en profundidad sobre cómo funciona el `await` en el próximo post.

# Conceptos básicos de la programación asincrónica

Me gustó esta sección porque nos da algunos consejos y cosas a considerar cuando usamos la librería `System.Threading.Tasks`.

Sobre `Task.WhenAll`:

> Si varias tareas arrojan una excepción, entonces todas esas excepciones van a ser asignadas a la Task que retorna el `Task.WhenAll`. Pero, cuando se le aplique el `await` a dicha Task, sólo una excepción va a ser lanzada.

Te ahorro ir a Google. `await Task.WhenAll(...)` no va a arrojar una `AggregateException`.

Sobre  `Task.WhenAny`:

> La tarea que retorna un `Task.WhenAny` nunca va a complietarse en estado `faulted` o `canceled`. Siempre va a retornar la primer Task que se complete, si dicha tarea se completó con una excepción, entonces la excepción no va a ser propagada a la Task que retorna el `Task.WhenAny`. Por esta razón, **siempre deberías aplicar el `await` a la Task que se completó**.

También tiene un tip muy interesante:

> Cuando se completa la primer Task, considera también cancelar las otras Tasks.

# Tarea para el hogar

El libro continúa hablando sobre DataFlow y Reactive. Pero te lo dejo a vos esa tarea.
Perfecto! Creo que aprendimos varios conceptos sobre programación asincrónica. En el próximo post vamos a ver qué lecciones podemos aprender de [Async in C# 5.0](https://www.amazon.com/Async-5-0-Unleash-Power/dp/1449337163).

¡No dejes de codear!

