---
layout: post
title: "FSharp, Qyoto on OS X"
description: ""
category: messingaround
tags: [fsharp, qt, qyoto, code]
---
__Warning__, before i go on. I'm quite new to F# and Qt so if theres any errors give me a yell below   

After playing around with [MonoMac](http://www.mono-project.com/MonoMac) decided it'd be nice to have a cross platform UI solution.

### Compiling Mono
The mono package for OS X are 32bit, where as Qt from homebrew is 64bit
so based on [adamv's homebrew alt repo](https://github.com/adamv/homebrew-alt) i made a mono3 recipe at [aktowns/tappedbrews](https://github.com/aktowns/tappedbrews) 
to install simply tap it and brew install

    brew tap aktowns/tappedbrews
    brew install aktowns/tappedbrews/mono3

This will install [mono 3.0.0](http://tirania.org/blog/archive/2012/Oct-22.html) and [FSharp 3.0.11](https://github.com/fsharp/fsharp/tree/3.0.11) 64bit, next we need Qt and Qyoto

### Getting Qt and compiling Qyoto
If anyone knows how to imply --HEAD using `depends_on` in a recipe please tell me!

    brew install qt
    brew install aktowns/tappedbrews/smokegen --HEAD
    brew install aktowns/tappedbrews/smokeqt --HEAD
    brew install aktowns/tappedbrews/qyoto --HEAD

### Installing monodevelop
Instead of compiling monodevelop, gtk# and numerous other dependencies its probably a lot easier to just grab the [monodevelop binaries](http://download.xamarin.com/monodevelop/Mac/MonoDevelop-3.0.4.7.dmg), install them and install [mono 2.10.9](http://download.mono-project.com/archive/2.10.9/macos-10-x86/11/MonoFramework-MRE-2.10.9_11.macos10.xamarin.x86.dmg) (to run monodevelop in) and select your default runtime as our brew compiled mono.    

![monodevelop](/images/monodevelop.png)

I noticed a bug with monodevelop (at least on 3.0.4.7) where if you have the binary 3.0.0 and your brew compiled 3.0.0 monodevelop wont let you set the default (it keeps defaulting to the binary 3.0.0) so for now 2.10.9 is fine.



After monodevelop is installed, set the default .net runtime to the brew installed version. 
and create a new F# console project!   

### An example F# Qyoto Application

{% highlight fsharp linenos %}
module Program

open System
open Qyoto

type QyotoApp() as self = class
  inherit Qyoto.QWidget()
  
  do
    let width = 250
    let height = 150
    base.WindowTitle <- "Hello World!"
    base.ToolTip <- "Goodbye World"
    
    self.InitUI()
    
    let qdw = new QDesktopWidget()
    
    let x = (qdw.Width - width) / 2
    let y = (qdw.Height - height) / 2
    
    base.Resize(width, height)
    base.Move(x, y)
    base.Show()
  
  [<Q_SLOT("quit()")>]
  member self.quit() : unit = Environment.Exit(-1)
    
  member self.InitUI() =
    let quit = new QPushButton("Quit", self)    
    self.Connect(quit, QWidget.SIGNAL("clicked()"), QWidget.SLOT("quit()")) |> ignore
   
    quit.SetGeometry(50, 40, 80, 30)
end
  
let main(args) : unit =
    new QApplication(args) |> ignore
    new QyotoApp() |> ignore
    QApplication.Exec() |> ignore

main(System.Environment.GetCommandLineArgs())
{% endhighlight %}

![hello world](/images/helloworld.png)
