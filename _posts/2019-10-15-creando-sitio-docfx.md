---
title: Creando un sitio para tu proyecto usando DocFX
tags: docfx dotnet
permalink: /blog/creando-sitio-docfx
cross-site-link: https://www.hardkoded.com/blog/creating-docfx-site
---

# Motivación

Hay muchos tutoriales tratando de explicar cómo solucionar un problema siguiendo una lista de pasos, el problema es que **asumen que vos compartís el mismo contexto que el autor, y aún más importante, asumen que vos tenes éxito en cada uno de los pasos**.

Yo quiero hacer algo distinto, quiero hacer un post en pseudo tiempo real, explicando como crear un sitio usando [DocFx](https://dotnet.github.io/docfx/). Nuestro contexto va a ser diferente, por su puesto, pero espero poder ayudarte a configurar tu sitio, explicándote no solo los paso correctos, sino también los problemas que voy encontrando, y que vos también podes llegar a encontrar.

# ¿Muy largo para leer? !Mirá el video!

<iframe width="560" height="315" src="https://www.youtube.com/embed/UZM7h4GE8bM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Introducción

Toda la API de [Puppeteer-Sharp](https://github.com/hardkoded/puppeteer-sharp/) está documentada utilizando [Comentarios XML](https://docs.microsoft.com/es-es/dotnet/csharp/programming-guide/xmldoc/). Lo que queremos hacer es crear un sitio utilizando [DocFX](https://dotnet.github.io/docfx/) y [Github Pages](https://pages.github.com/). Por último, queremos que el sitio esté disponible en http://www.puppeteer-sharp.com.

# ¿Qué es lo que sabemos?

 * Sabemos que le podemos pedir al compilador que genere un archivo XML con la documentación. [Ya estamos haciendo eso](https://github.com/hardkoded/puppeteer-sharp/blob/master/lib/PuppeteerSharp/PuppeteerSharp.csproj#L40).
 * También sabemos que queremos usar [DocFX](https://dotnet.github.io/docfx/tutorial/walkthrough/walkthrough_create_a_docfx_project.html).
* Y que queremos que sea un proceso automático en nuestro CI.

# Pomodoro 1 - Setup

Como necesitamos correr el CI muchas veces y no queremos ensuciar el repositorio de Puppeteer, vamos a crear un nuevo repositorio. [docfx-playground](https://github.com/kblok/docfx-playground) suena ~~genial~~ bien.

![Repo Image](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/1_repo.png)

También necesitamos configurar un CI para este proyecto.
![App Veyor](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/2_appveyor.png)

Copiemos todos nuestros archivos a este nuevo repository.

## DocFX

Bien, ahora vamos a necesitar 2 carpetas:

* Sabemos que DocFX va a necesitar una carpeta para su proyecto. Una carpeta llamada `docfx_project`.
* También sabemos que vamos a intentar generar un sitio usando la carpeta `_site`, pero como 
[Github Pages va a buscar una carpeta llamada docs](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/), vamos a crear nuestro sitio allí.

## docfx_project

Si corremos `docfx init`, este debería crear una carpeta `docfx_folder` en nuestro proyecto.
 
>docfx init -q

![Dcofx](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/3_docfxproject.png)

Fabuloso!

Ahora simplemente necesitamos setear la ruta de esa carpeta en el archivo `docfx.json`:


```
"metadata": [
    {
      "src": [
        {
          "cwd": "../",
          "files": [
            "lib/PuppeteerSharp/**.csproj"
          ]
        }
      ],
      "dest": "api",
      "disableGitFeatures": false
    }
  ],
```

Bien. Ahora si llamamos a `docfx metadata` debería… hacer algo...

>docfx metadata docfx_project/docfx.json 

BOOM!
![DocFx metadata](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/4_docfx_metadata.png)

No se que son esos archivos todavía, pero es algo.
Tratemos de buildear el sitio.

>docfx build docfx_project/docfx.json -o docs 

Bueno, podría haber sido peor. Primero, creo una carpeta `_sites` dentro de mi carpeta `docs`. No quería hacer eso seteando como output la carpeta `docs`.

A parte, la home page y la API index page podrían haber tenido más links conectando a las nuevas páginas...

_Home del sitio_
![Primera Home](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/5_first_home.png)
_La Home de la API_
![FirstAPI](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/6_first_api.png)
_La documentación de una clase_
![Doc Page](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/7_first_doc.png)

#### Fin del pomodoro 1

## Pomodoro 2 -  Dándole forma al sitio

Necesitamos deshacernos de la carpeta `_sites`, para ello vamos a cambiar la propiedad `dest` en el `docfx.json`.

>"dest": "."

Perfecto, ahora nuestro sitio está usando la carpeta `docs`.
Ahora vamos a intentar mejorar nuestro sitio:
Voy a trabajar en el estilo.
Agregar algo de contenido a la home page.
Ver como agregar links a la documentación de la API.

Creí que iba a perder todo un pomodoro con esto, pero resulta que lo único que necesitas para tener el sitio corriendo es simplemente llamar a `docfx serve`. 

>docfx serve

![Good API Page](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/8_first_good_api.png)

#### Fin del Pomodoro 2

## Pomodoro 3 - GitHub Pages y AppVeyor

Bien, ya tenemos nuestros sitio corriendo, vamos a hacer push y configurar Github Pages.

![Githubpage](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/9_set_github_page.png)

Tenemos un sitio!

![Site](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/10_first_site.png)

Si bien hay algunas cosas que mejorar, vamos a configurar AppVeyor. La idea es simple:
Necesitamos compilar docfx solo cuando un Tag (release) es creado.
También necesitamos actualizar la metadata de DocFx y el sitio.
Por último, necesitamos hacer un push del sitio compilado nuevamente al repo.

AppVeyor tiene [un muy buen documento](https://www.appveyor.com/docs/how-to/git-push/) para ayudarnos a configurar git, de modo que podamos hacer un push del contenido. Basicamente necesitamos [configurar un personal access token en Github](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/), encriptarlo y usarlo en nuestro script. La sección de configuración de git debería ser algo como esto:

```
- git config --global credential.helper store
- Add-Content "$HOME\.git-credentials" "https://$($env:git_access_token):x-oauth-basic@github.com`n"
- git config --global user.email "dariokondratiuk@gmail.com"
- git config --global user.name "Darío Kondratiuk"
```

Luego generamos la documentación

```
- docfx metadata docfx_project/docfx.json
- docfx build docfx_project/docfx.json -o docs
```

Finalmente, hacemos un commit a nuestro repo. 

```
- git checkout master
- git add docfx_project/*
- git add docs/*
- git commit -m "Docs for"
- git push origin
```

Ouch… DocFx me está tirando este error:

>[18-06-08 12:17:14.994]Warning:ExtractMetadata(C:/projects/docfx-playground/lib/PuppeteerSharp/PuppeteerSharp.csproj)Workspace failed with: [Failure] Msbuild failed when processing the file 'C:\projects\docfx-playground\lib\PuppeteerSharp\PuppeteerSharp.csproj' with message: C:\Program Files\dotnet\sdk\2.1.300\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.GenerateAssemblyInfo.targets: (161, 5): The "GetAssemblyVersion" task failed unexpectedly.

Y este cuando intento hacer un push a GitHub:

>fatal: unable to access 'https://github.com/kblok/docfx-playground.git/': The requested URL returned error: 403
550

#### Fin del Pomodoro 3
 
## Pomodoro 4 - Branch gh-pages 

Cambio de planes! Encontré un [artículo en GitHub](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/) el cual nos muestra como podemos publicar un GitHubPage usando un branch llamado `gh-page`. Es genial porque **los commits relacionados con docfx no se van a mezclar con los commits del branch de master**, tambien **voy a ser capáz de mantener mi [branch master protegido](https://help.github.com/articles/configuring-protected-branches/)**. 

Encontré [este gist](https://gist.github.com/ramnathv/2227408) con un script para crear un branch `gh-page` vacío.

```
cd /path/to/repo-name
git symbolic-ref HEAD refs/heads/gh-pages
rm .git/index
git clean -fdx
echo "My GitHub Page" > index.html
git add .
git commit -a -m "First pages commit"
git push origin gh-pages
```

Genial! Tenemos un branch llamado `gh-pages`vacío y listo para usar.
Vamos a volver a nuestro código en AppVeyor. Para poder hacer un push a GitHub, necesitamos agregar `https` a nuestro remote.

```
git remote add pages https://github.com/kblok/docfx-playground.git
``` 

Entonces vamos a poder usar `git subtree push` para hacer un push de nuestro sitio a este nuevo branch remoto.

```
git subtree push --prefix docs pages gh-pages
```

Ok, ahora AppVeyor me está tirando este error:

```
"GetAssemblyVersion" task failed unexpectedly.
331System.NullReferenceException: Object reference not set to an instance of an object.
```

#### Fin del Pomodoro 4 :(

# Pomodoro 5 - Git subtree

[Este post](https://dev.to/letsbsocial1/deploying-to-gh-pages-with-git-subtree) me muestra que tenemos que registrar el subtree antes de hacer un `git subtree push`.

```
git subtree add --prefix docs gh-pages
```

También aprendí que después de crear el subtree tenemos que hacer un commit y un push a ese subtree:

```
git checkout master
git subtree add --prefix docs gh-pages
docfx metadata docfx_project/docfx.json
docfx build docfx_project/docfx.json -o docs
git add docs/
git commit -m "Docs"
git subtree push --prefix docs pages gh-pages
```

Pero el comando push está trabado :(

#### Fin del Pomodoro 5

# Pomodoro 6 - Configurando un GitHub Access Token

Encontré un [buen script](https://github.com/dustinchilson/dustinchilson.com/blob/master/_tools/before_build.ps1) para configurar git.

El error 403 mientras tratamos de hacer un push a Github se debe a que tenemos que chequear la opción “Repo” cuando creamos el Token.

![Repo check](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/11_repo_check.png)

También aprendí que esta es la manera correcta de registrar un subtree:

```
git subtree add --prefix docs pages/gh-pages
```

Listo! Ahora necesitamos poner todo este código dentro de un if de modo tal que solo se ejecute cuando estamos creando un nuevo Tag/Release en GitHub. 

Este sería el script final:

```
if($env:APPVEYOR_REPO_TAG -eq 'True'){
    git config --global credential.helper store
    Add-Content "$env:USERPROFILE\.git-credentials" "https://$($env:git_access_token):x-oauth-basic@github.com`n"

    git config --global user.email "dariokondratiuk@gmail.com"
    git config --global user.name "Darío Kondratiuk"
    git remote add pages https://github.com/kblok/docfx-playground.git
    git fetch pages
    git checkout master
    git subtree add --prefix docs pages/gh-pages
   docfx metadata docfx_project/docfx.json
   docfx build docfx_project/docfx.json -o docs
    git add docs/* -f
    git commit -m "Docs build $($env:APPVEYOR_BUILD_VERSION)"
    git subtree push --prefix docs pages gh-pages
}
```
Ahora tenemos que configurar nuestro dominio y ya estamos listos!

![Setup domain](https://raw.githubusercontent.com/kblok/kblok.github.io/master/img/docfx-tutorial/12_setup_domain.png)

Si te estás preguntando qué será este error:

>[18-06-08 12:17:14.994]Warning:[ExtractMetadata](C:/projects/docfx-playground/lib/PuppeteerSharp/PuppeteerSharp.csproj)Workspace failed with: [Failure] Msbuild failed when processing the file 'C:\projects\docfx-playground\lib\PuppeteerSharp\PuppeteerSharp.csproj' with message: C:\Program Files\dotnet\sdk\2.1.300\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.GenerateAssemblyInfo.targets: (161, 5): The "GetAssemblyVersion" task failed unexpectedly.

La verdad que ni idea, pero la metadata se está generando a pesar de eso.

#### Fin del Pomodoro 6

# Conclusión

Fue un camino bastante divertido, aunque un poco más largo de lo que esperaba. Bueno, si bien seis pomodoros son solamente dos horas y media, como invertí solamente un pomodoro por día, todo esto me llevó casi una semana.
Espero que este post te ayude a configurar DocFx en tu solución y te simplifique la vida.

Te estarás preguntando, por qué AppVeyor y no Github Actions? No te preocupes, ya lo vamos a implementar con Github actions ;)

No dejes de codear!