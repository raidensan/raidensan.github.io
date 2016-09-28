---
published: true
title: Multilanguage DevExpress components
layout: post
tags: [Multilanguage, DevExpress, .Net]
---
By default [DevExpress](https://www.devexpress.com/) components come with English, German, Spanish, Japanese & Russian language support.  To change component language one needs to specify `CultureInfo` by doing:

    Thread.CurrentThread.CurrentCulture = new CultureInfo("en-US");
    Thread.CurrentThread.CurrentUICulture = new CultureInfo("en-US");

Of course, resource files of desired language should be present alongside your binary.

So what is needed to use components in a language other than default language support?
DevExpress provide translated resources - AKA SatteliteAssembly - on demand. Go to [Localization](https://localization.devexpress.com), signup or login with your account. Now, press Add Translation and select your desired component version and Culture. Press Download; A download link will be sent to registered e-mail.

Download these files from provided link and put them alongside your assembly inside a folder with culture name, e.g. `tr`,`kr` & etc. Add these files to your project as Existing Items and set `Copy to output` to `Always`.

Now you are all set to use these resources, and to use, you only need to set `CurrentCulture` and `CurrentUICulture` to desired culture as shown above.




