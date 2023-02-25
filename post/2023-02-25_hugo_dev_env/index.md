---
title: A Hugo Development Environment
date: 2023-02-25
tags: ["hugo", "docker", "git"]
draft: true
---

## Introduction

As I've mentioned before, this site is built with [Hugo](https://gohugo.io/) as a static site generator.  Hugo is great and if you follow the quickstart guide you'll have a site up and running right quick.  However, if you're like me and you're developing your own Hugo themes and want to keep your content in a separate repo because you post from multiple devices, etc. then you'll need something a little more advanced.  

Previously, I did this with git submodules with the help of [Andrew Hoog's Guide](https://www.andrewhoog.com/post/git-submodule-for-hugo-themes/).  But recently Hugo has incorporated Golang's module feature as [Hugo Modules](https://gohugo.io/hugo-modules/) which has made things much easier.  You could use Hugo Modules locally of course, but what I've found works best for me is to leverage Docker Compose locally and use Modules for pushing the site to Production.  This takes a bit of work upfront but makes the development and maintenance of my Hugo sites much nicer in the long run.

## Getting Started

Create a main directory for your project.

{{< highlight bash >}}
mkdir paddysprojects
{{< /highlight >}}

Create a docker-compose.yaml file.

{{< highlight yaml >}}
version: "3.4"

services:

  build:
    image: klakegg/hugo:ext-alpine
    entrypoint: ""
    command: >
      sh -c "rm go.mod &&
             rm go.sum &&
             hugo mod init dummy-mod &&
             hugo mod tidy &&
             hugo mod get -u &&
             hugo --environment='production'"
    volumes:
      - "./src:/src"
    environment:
      - "TZ=Pacific/Auckland"

  serve:
    image: klakegg/hugo:ext-alpine
    entrypoint: ""
    command: >
      ash -c "hugo mod get -u &&
              hugo server --environment='development' --baseURL '' --disableFastRender"
    environment:
      - "TZ=Pacific/Auckland"
    volumes:
      - "./src:/src"
      - "./themes:/src/themes"
      - "./content:/src/content"
    ports:
      - "1313:1313"
{{< /highlight >}}

What's nice about this is that we don't have to install Go and Hugo to get the project started.  We can use the klakegg/hugo Docker image.



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