---
published: true
title: Automatic Integration: VisualStudio & ConfuserEx
layout: post
tags: [VisualStudio, ConfuserEx, Obfuscation]
categories: [Automatic, Integration]
---
In this post I'll demonstrate how to integrate VisualStudio build process with obfuscation.

Recently I've been playing with [ConfuserEx](https://yck1509.github.io/ConfuserEx/). I decided to use obfuscation with ConfuserEx in my project. It's fairly easy to obfuscate a built assembly. But for integrating with VisualStudio there are some steps. Let's dive in!

I assume you are familiar with using ConfuserEx, if not hop on [Project Wiki](https://github.com/yck1509/ConfuserEx/wiki) to get started.

_Note: If you are using EntityFramework in your assembly you are going to have a bad time. I suggest move your data access components to a separate project._

After creating and configuring our ConfuserEx project we are ready to roll. Run VisualStudio and open your project. Go to your project properties and open Build Events tab. Type the following commands inside Post-build event command line:
    
    if $(ConfigurationName) == Release (
    ...\ConfuserEx_bin\Confuser.CLI.exe $(SolutionDir)confuser.crproj
    copy /y $(TargetDir)Confused\*.* $(TargetDir)
    rmdir $(TargetDir)Confused /s /q
    )

Let's examine these commands:
`if $(ConfigurationName) == Release` is to make sure we only obfuscate Release binaries. If your project have more than Debug/Release configuration you can specify which configuration you want to obfuscate.

`...\ConfuserEx_bin\Confuser.CLI.exe $(SolutionDir)confuser.crproj` is the path to Confuser.CLI.exe binary on your system followed by path of ConfuserEx project file. Remember to replace `...` this with actual path in your system.

`copy /y $(TargetDir)Confused\*.* $(TargetDir)` copies all files (confused binaries) back to their original folder.

Finally, `rmdir $(TargetDir)Confused /s /q` removes Confused folder along with it's sub-folder (if any).

That's it, now build your project with Release configuration. If everything goes as expected your assemblies are now obfuscated. To confirm you can use any Disassemble tool.
