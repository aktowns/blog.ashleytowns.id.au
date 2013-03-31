---
layout: post
title: "Running C#/F# on heroku"
description: "Setting up Nancy in F# to run via xbuild with nuget on heroku"
category: code
tags: [fsharp, xamarinstudio, heroku, csharp, xbuild]
---

Recently i decided to try and get [Nancy](http://nancyfx.org/) up and running on [Heroku](http://www.heroku.com/), heres a brief gist of things i did to get it up and working.

### The buildpack
There are [a](https://github.com/bvanderveen/heroku-mono-buildpack) [few](https://github.com/brandur/heroku-buildpack-mono) [buildpacks](https://github.com/BenHall/heroku-buildpack-mono) for mono2.x around on the internet but I was aiming for mono3 this turned out to be the longest part.
I ended up trying to use [vulcan](https://github.com/heroku/vulcan) to create a buildpack but kept hitting issues so ended up `heroku run bash`'ing and building mono3 and netcatting it over, its available [here](https://github.com/aktowns/mono3-buildpack). I ended up throwing [Nuget](http://nuget.org/) in there too.

To use the build pack, pass it in with `heroku create` on a project

    heroku create projname --buildpack https://github.com/aktowns/mono3-buildpack.git

The buildpack looks for a solution file (.sln) in root directory, and if it detects any packages.config's it will run Nuget on them and attempt to install them to packages/ this syncs up with the way [monodevelop nuget addin](https://github.com/mrward/monodevelop-nuget-addin) stores files.

### Nancy
[Nancy](http://nancyfx.org/) is a [Sinatra](http://www.sinatrarb.com/) inspired framework for building web applications. Getting nancy setup with f# was pretty straight forward, the only issue initially was finding the right host to bind to, until i discovered binding to localhost allows all hosts. Make sure to include Nancy and Nancy.Hosting.Self using monodevelop-nuget-addin.

{%highlight fsharp linenos %}
module viewfiles.Main

open System
open Nancy.Hosting

type DemoApp () =
    inherit NancyModule()
    do
        let Get = base.Get
        Get.["/"] <- fun parameters -> "Hello from F#/Nancy on Heroku!" :> obj

[<EntryPoint>]
let main args = 
    let env_port = Environment.GetEnvironmentVariable("PORT")
    let port = if env_port = null then "1234" else env_port
    
    let nancy_host = new Nancy.Hosting.Self.NancyHost(new Uri("http://localhost:" + port));
    nancy_host.Start()
    
    while true do Console.ReadLine() |> ignore
    nancy_host.Stop()
    0
{% endhighlight %}

#### The Procfile
The procfile tells heroku how to launch your application, in the case of self-hosted nancy the following works fine with the buildpack above.

    web: mono appdir/bin/Debug/app.exe

### Compiling and deploying
I had to enable build with xbuild in [xamarin studio](http://xamarin.com/studio)

![xamarin studio](/images/Screen%20Shot%202013-04-01%20at%2010.30.26%20AM2.png) 

and re-order the fsproj build order, so `Program.fs` sits last (or your file with the entrypoint) 

![re-order](/images/Screen%20Shot%202013-04-01%20at%2010.40.25%20AM2.png)

`git commit` your changes and push to heroku `git push heroku` 

{%highlight text linenos %}
Projects/viewfiles git:(master) ▶ git push heroku
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 316 bytes, done.
Total 3 (delta 1), reused 0 (delta 0)

-----> Fetching custom git buildpack... done
-----> MonoFramework app detected
-----> Copying mono to /tmp/build_1rpsx5dkzhkhv/vendor/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  103M  100  103M    0     0  1746k      0  0:01:00  0:01:00 --:--:-- 2273k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  235k  100  235k    0     0   262k      0 --:--:-- --:--:-- --:--:--  262k
-----> Setting envvars
-----> Importing trusted root certificates
Mozilla Roots Importer - version 3.0.7.0
Download and import trusted root certificates from Mozilla's MXR.
Copyright 2002, 2003 Motus Technologies. Copyright 2004-2008 Novell. BSD licensed.

Downloading from 'http://mxr.mozilla.org/seamonkey/source/security/nss/lib/ckfw/builtins/certdata.txt?raw=1'...
Importing certificates into user store...
140 new root certificates were added to your trust store.
Import process completed.

-----> Setting up nuget
Checking for updates from https://nuget.org/api/v2/.
Currently running NuGet.exe 2.2.1.
NuGet.exe is up to date.
-----> Installing dependencies with nuget
Successfully installed 'Nancy.Serialization.JsonNet 0.16.1'.
Successfully installed 'Nancy.Hosting.Self 0.16.1'.
Successfully installed 'Nancy.Authentication.Forms 0.16.1'.
Successfully installed 'Nancy.Viewengines.Razor 0.16.3'.
Successfully installed 'Nancy 0.16.1'.
Successfully installed 'System.Web.Razor.Unofficial 2.0.2'.
Successfully installed 'Newtonsoft.Json 4.5.11'.
-----> Compiling application
XBuild Engine Version 3.0.7.0
Mono, Version 3.0.7.0
Copyright (C) Marek Sieradzki 2005-2008, Novell 2008-2011.

Build started 03/31/2013 21:53:10.
__________________________________________________
Project "/tmp/build_1rpsx5dkzhkhv/viewfiles.sln" (default target(s)):
  Target ValidateSolutionConfiguration:
    Building solution configuration "Debug|x86".
  Target Build:
    Project "/tmp/build_1rpsx5dkzhkhv/viewfiles/viewfiles.fsproj" (default target(s)):
      Target PrepareForBuild:
    ... SNIP ...
    Done building project "/tmp/build_1rpsx5dkzhkhv/viewfiles/viewfiles.fsproj".
Done building project "/tmp/build_1rpsx5dkzhkhv/viewfiles.sln".

Build succeeded.
   0 Warning(s)
   0 Error(s)

Time Elapsed 00:00:04.1524600
-----> Discovering process types
       Procfile declares types -> web

-----> Compiled slug size: 110.5MB
-----> Launching... done, v25
       http://viewfiles.herokuapp.com deployed to Heroku

To git@heroku.com:viewfiles.git
   dae558e..35cd50f  master -> master
{%endhighlight%}

