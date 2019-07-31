---
title: Trabajando con stored procedures - Parte 1
tags: sqlserver
permalink: /blog/trabajando-con-stored-procedures-parte1
---

## ¿Muy largo para leer? !Mirá el video!

<iframe width="560" height="315" src="https://www.youtube.com/embed/UX8kcg34nGo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

_Don't speak Spanish? Checkout the [English version](https://www.hardkoded.com/blogs/how-to-work-with-stored-procedures-and-not-die-trying)_

## ¿Cuál es el problema con las bases de datos?

No voy a intentar venderte la idea de que los stored procedures son lo mejor, que todos deberían tirar sus ORMs a la basura y mover todo a la base de datos. Creo que cada equipo tiene que elegir la mejor herramienta que no solo los ayuda a resolver sus problemas sino que también los ayuda a ser más productivos.

Ya sea que hayas caído en un proyecto lleno de stored procedures, que seas un amante de las bases de datos o que simplemente consideras que los stored procedures son la herramienta adecuada para tu trabajo, si usas stored procedures te vas a encontrar con varios desafíos al momento de mejorar la productividad de tu equipo.

He notado que a veces, incluso cuando tiene sentido usar un stored procedure, un developer no va a ir por ese camino simplemente porque no se siente productivo usando stored procedures.

Generalmente esto sucede por alguno de los siguientes motivos:

### Bases de datos compartidas
Hay equipos que solo tienen una instancia de SQL Server con muchas versiones de la misma base de datos. Por ejemplo: Dev, Test, CI, etc. 

Entonces, si uno de los devs hace un cambio en una de las bases de datos, esto afectaría **inmediatamente** a los otros miembros del equipo. 
Algunos equipos tratan de mitigar este problema alterando lo menos posible la base de datos, y avisando a todo el equipo cada vez que lo hacen.

### Falta de control de versiones

Tal vez nunca lo pensaste, pero el control de versiones juega un papel muy importante en la productividad. Mientras más fácil sea integrar tu código, controlar cambios y switchear entre tareas, más productivo vas a ser.
Muy pocos equipos ponen a la base de datos dentro del control de versiones. ¿Cómo haces eso? Tranquilo, ya vamos a llegar allá.

### No hay debugging

¿Me estás diciendo que hay gente que debuggea stored procedures? Si, Creo que hay solo dos developers en el mundo que hacen eso, pero sí, es posible.


# Descentralizar las bases de datos

Empecemos a hablar sobre productividad. En este primer post vamos a hablar del primero de los problemas, que son las bases de datos compartidas.

Lo que deberíamos hacer es tener un base de datos por persona, donde tenga la libertad de actualizar o incluso recrear la base de datos si lo necesita. Podemos lograr esto de 3 maneras, ok pueden haber más, pero estas son las 3 maneras que recomiendo:

## Bases de datos en un servidor compartido
Si ya estás usando un servidor que está siendo compartido por tus devs, esta sería la solución más sencilla. Simplemente deberías usar una convención para identificar cada base de datos, podrías hacer algo super inteligente como `<Proyecto>DEV<NombreDev>`.

### Ventajas
* No hay mucho setup para hacer. Simplemente crear una base para cada uno.
* Los miembros del equipo puede acceder a la base de otro dev, haciendo más fácil la colaboración.

### Contras
*Si no estás usando un servidor compartido ahora, vas a necesitar configurar uno y pensar cómo vas a implementar la seguridad. 
* Puede ser lento, comparado con un entorno local.
* Un dev puede tirar abajo el servidor si está ejecutando una tarea pesada.

## LocalDB

>LocalDB es una versión liviana de SQL Server Express que está orientada al desarrollo.

Tal vez no lo sepas, pero si tenés instalado Visual Studio, hay altas probabilidades que ya tengas instalado LocalDB. Podés verificarlo abriendo el tab **SQL Server Object Explorer**.

<img src="https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/working-with-stored-procedures/SQLServerObjectExplorer.png" height="200px">

Si no encontrás un LocalDB ahí, podés ver 
[acá](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/sql-server-2016-express-localdb) cómo instalarlo. Hay otras versiones que podés instalar, por ejemplo: podes instalar una instancia completa del [Microsoft SQL Server Developer Edition](https://blogs.technet.microsoft.com/dataplatforminsider/2016/03/31/microsoft-sql-server-developer-edition-is-now-free/) de forma gratuita.

### Pros

* Ya tenés instalado LocalDB, y está listo para usar.
* Como el nombre lo dice, es local, no necesitas VPN ni acceso a un servidor remoto.
* Una base de datos local va a hacer más rápido tu entorno de desarrollo, en especial si trabajás remoto.

### Cons

* Ya sea que uses LocalDB o SQL Server Developer Edition, estamos hablando de un proceso más corriendo en tu computadora, cosa que podría ponerla un poco más “pesada”.

## Un servidor local en Docker

Esta solución es similar a usar un LocalDB, pero es genial si usás MacOS. Si usás Visual Studio en una máquina virtual, tiener un SQL Server en Docker corriendo en MacOS, va a acelerar tu máquina virtual significativamente. Te recomiendo esta opción si sos usuario del sistema operativo de la manzana.

Podés leer sobre como correr SQL Server en Docker [aquí](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker).

# Palabras finales

Espero que este post te haya dado un poco de esperanzas si estás trabajando con Stored Procedures. Pero no te pierdas el próximo post (Darío agregá el link acá una vez que lo hayas subido). En la parte dos vamos a ir al plato principal, vamos hablar sobre cómo integrar la base de datos en tu código con Visual Studio.

No dejes de codear!

