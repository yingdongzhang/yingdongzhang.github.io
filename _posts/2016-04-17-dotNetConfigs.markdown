---
layout: article
mathjax: true
title:  "Acess Another Project's Config Settings with .NET Projects"
date:   2016-04-17 11:00:30 +1000
tags: .NET
category: blog
key: dotNetConfigs
---
When working with .NET projects, you will certainly come across the need to access another project's configuration settings. Here are some tips on how to do it.

Let's say we have a solution that contains project A and B. B has a reference to A. In A we have two config fields: Config1 and Config2. In B we have two config fields as well: Config3 and Config4.

<img src="/assets/dotNetConfig/structure.png" alt="Soluction Structure" style="width: 300px;"/>

Now in B I want to access all the config settings from both A and B. This's very easy to do with B's own config settings, in B->Class1->Test() I can write:

```cs
string config3 = B.Properties.Settings.Default.Config3;
int config4 = B.Properties.Settings.Default.Config4;
```

However, when you try to access A's config settings, you will encounter the `get accessor is inaccessible` error, which makes sense since A is not exposing its config settings to the outside world.

![Inaccessible Settings](/assets/dotNetConfig/inaccessible.png)

You have two options to deal with this:

### 1. Modify the accessors of Config1 and Config2 in A by:

1. Right click on project A, select "Properties"
2. Select "Settings" in the left panel
3. On the top of the left panel, you will find the "Access Modifier"
4. By default it is set to `Internal`, you can change it to `Public`

![Modify Accessors](/assets/dotNetConfig/modify-accessor.png)

After that you can access Config1 and Config2 like normal:

```cs
string config1 = A.Properties.Settings.Default.Config1;
int config2 = A.Properties.Settings.Default.Config2;
```

### 2. Expose A's config settings

If you don't feel like modifying the accessors, which I kinda prefer not to, you can make A expose its config to the outside world by simply adding in a `ConfigExposer` (or `Enums`, whatever you prefer), and put the config settings as static readonly properties:

![Config Exposer](/assets/dotNetConfig/config-exposer.png)

Then you can access A's Config1 and Config2 like this:

```cs
string config1 = A.ConfigExposer.Config1;
int config2 = A.ConfigExposer.Config2;
```