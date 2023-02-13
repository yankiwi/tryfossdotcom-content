---
title: Using Ionicons in Hugo
date: 2023-02-12
tags: ["hugo", "ionicons"]
draft: false
---

## Introduction

[Ionicons](https://ionic.io/ionicons/) is a great icon set created by the [Ionic](https://ionic.io/) team.  It's MIT licensed, looks great on web, Android and iOS.  Because it supports both SVG and web font there are a lot of ways to use Ionicons in your project.  This site uses [Hugo](https://gohugo.io/) so my goal is make Ionicons available both as a partial and shortcode without Javascript.

## Get the Icons

To get the Ionicons SVG files, I'll be using [Hugo Modules](https://gohugo.io/hugo-modules/).  Modules are nice because they let you specify a single file or directory within a git repository.  In our case, we just want the svg directory.  So I add the following to me config.toml file.

{{< highlight toml >}}
[module]
    [[module.imports]]
        path = "github.com/ionic-team/ionicons"
        disable = false
        [[module.imports.mounts]]
            source = "src/svg"
            target = "assets/svg/ionicon"
{{< /highlight >}}

{{< ionicon "sunny" "16" "16" "#ff0000" >}}

##### FROM:
{{< highlight javascript >}}
var n={"service-worker-url":"upup.sw.min.js"}
...AND...
{scope:"./"}
{{< /highlight >}}
##### TO:
{{< highlight javascript >}}
var n={"service-worker-url":"/upup.sw.min.js"}
...AND...
{scope:"/"}
{{< /highlight >}}
I also slightly modified the header script to include Hugo generated pages...
{{< highlight html >}}
<script src="/upup.min.js"></script>
<script>
    UpUp.start({
        'content-url': '{{ .Page.Permalink }}',
        'assets': ['/css/main.css', '/css/bulma.min.js']
    });
</script>
{{< /highlight >}}

## Home Screen Icons:

Setting up home screen icons was almost too easy thanks to [realfavicongenerator.net](https://realfavicongenerator.net/).  They claim they'll have you sorted in 5 minutes and that's pretty close even when importing into Hugo.  They deliver all of the icon files and the head code required.  I just copied the icon files into my theme's static folder and pasted the code into my theme's header partial file. 

## Results:

![Lighthouse Score](lighthouse.png)

Live: [pagespeed.web.dev/report?url=https://tryfoss.com](https://pagespeed.web.dev/report?url=https%3A%2F%2Ftryfoss.com)