---
title: "FSharp, Qyoto on OS X"
description: "Setting up mono 3.0, fsharp and qyoto on OS X"
---

After playing around with [MonoMac](http://www.mono-project.com/MonoMac) I realized most people I know are running windows, most of the time thats fine. But sometimes I'd like to show something off and [Qyoto](http://techbase.kde.org/Development/Languages/Qyoto) seems like it could be the perfect cross platform solution!   

__Warning__, before I go on. I'm quite new to F# and Qt so if theres any errors give me a yell below  

I got a bit stuck going through the whole process of setting everything up, so here's what I did for anyone else looking to play with it.

### Compiling Mono
The mono packages for OS X are 32bit, where as the Qt libraries from homebrew are 64bit.
So based on [adamv's homebrew alt repo](https://github.com/adamv/homebrew-alt) I made a mono3 recipe at [aktowns/tappedbrews](https://github.com/aktowns/tappedbrews) 
to install simply tap it and brew install.

    [linenums=false]
    brew tap aktowns/tappedbrews
    brew install aktowns/tappedbrews/mono3

This will install [mono 3.0.0](http://tirania.org/blog/archive/2012/Oct-22.html) and [FSharp 3.0.11](https://github.com/fsharp/fsharp/tree/3.0.11) 64bit, next we need Qt and Qyoto

### Getting Qt and compiling Qyoto
If anyone knows how to imply --HEAD using `depends_on` in a recipe please tell me!   
These homebrew recipes are mostly based off the instructions [here](http://vivekgani.com/blog/2012/09/17/setting-up-qyoto-on-osx-lion/) with a few minor changes.

    [linenums=false]
    brew install qt
    brew install aktowns/tappedbrews/smokegen --HEAD
    brew install aktowns/tappedbrews/smokeqt --HEAD
    brew install aktowns/tappedbrews/qyoto --HEAD

### Installing monodevelop
Instead of compiling monodevelop, gtk# and numerous other dependencies its probably a lot easier to just grab the [monodevelop binaries](http://download.xamarin.com/monodevelop/Mac/MonoDevelop-3.0.4.7.dmg), install them and install [mono 2.10.9](http://download.mono-project.com/archive/2.10.9/macos-10-x86/11/MonoFramework-MRE-2.10.9_11.macos10.xamarin.x86.dmg) (to run monodevelop in) and select your default runtime as our brew compiled mono.    

![monodevelop](/content/images/monodevelop.png)

I noticed a bug with monodevelop (at least on 3.0.4.7) where if you have the binary 3.0.0 and your brew compiled 3.0.0 monodevelop wont let you set the default (it keeps defaulting to the binary 3.0.0) so for now 2.10.9 is fine.


Now lets create a new F# console project!   

### An example F# Qyoto Application
Adapted from [zetcode's c# qyoto introduction](http://zetcode.com/gui/csharpqyoto/introduction/)

    [name=qyoto.fsx]
    module Program

    open System
    open Qyoto

    let SIGNAL = QWidget.SIGNAL
    let SLOT = QWidget.SLOT

    type QyotoApp() as self = class
      inherit QWidget()
      
      do
        base.WindowTitle <- "Hello World!"
        base.ToolTip <- "Goodbye World"
        
        self.setupButton
        self.resizeAndCenter 250 150
        
        base.Show()
      
      member self.resizeAndCenter w h =
        let qdw = new QDesktopWidget()
        let x = (qdw.Width - w) / 2
        let y = (qdw.Height - h) / 2
        base.Resize(w, h)
        base.Move(x, y)
        
      member self.setupButton : unit =
        let quit = new QPushButton("Quit", self)    
        self.Connect(quit, SIGNAL("clicked()"), SLOT("quit()")) |> ignore
        quit.SetGeometry(50, 40, 80, 30)
      
      // MARK: Slots
      [<Q_SLOT("quit()")>]
      member self.quit() : unit = Environment.Exit(0)
    end

    let main(args) : unit =    
        let application = new QApplication(args)
        let qyotoApp = new QyotoApp()
        
        QApplication.Exec() |> ignore

    main(Environment.GetCommandLineArgs())


![hello world](/content/images/helloworld.png)
