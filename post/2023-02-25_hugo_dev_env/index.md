---
title: A Hugo Development Environment
date: 2023-02-25
tags: ["hugo", "docker", "git"]
draft: false
---

## Introduction

As I've mentioned before, this site is built with [Hugo](https://gohugo.io/) as a static site generator.  Hugo is great and if you follow the quickstart guide you'll have a site up and running right quick.  However, if you're like me and you're developing your own Hugo themes and want to keep your content in a separate repo because you post from multiple devices, etc. then you'll need something a little more advanced.  

Previously, I did this with git submodules with the help of [Andrew Hoog's Guide](https://www.andrewhoog.com/post/git-submodule-for-hugo-themes/).  But recently Hugo has incorporated Golang's module feature as [Hugo Modules](https://gohugo.io/hugo-modules/) which has made things much easier.  You could use Hugo Modules locally of course, but what I've found works best for me is to leverage Docker Compose locally and use Modules for pushing the site to Production.  This takes a bit of work upfront but makes the development and maintenance of my Hugo sites much nicer in the long run.

## Setting up the Hugo Environment

Create a main directory for your project.

{{< highlight bash >}}
mkdir tryfossdotcom
{{< /highlight >}}

Create a .gitignore file.
{{< highlight bash>}}
### Hugo ###
# Generated files by hugo
/src/public/
/src/resources/_gen/
/src/assets/jsconfig.json
/src/hugo_stats.json

# Temporary lock file while building
/src/.hugo_build.lock

# Docker compose dev
/themes/
/content/
{{< /highlight>}}

Then create a docker-compose.yaml file.
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

Execute ASH to get to the command line on the "serve" container:
{{< highlight bash >}}
docker compose run serve /bin/ash
{{< /highlight >}}

Then, in the "serve" container, create your new site in /src and add some content:
{{< highlight bash >}}
hugo new site /src
cd /src
hugo new post/my-first-post.md
exit
{{< /highlight >}}
Add some content to my-first-post.md and change draft to false.

Back outside of the container, clone a theme into the ./themes directory.  In my case, I'll be using my own theme.
{{< highlight bash >}}
cd themes
git clone https://github.com/yankiwi/tryfossdotcom-theme ./tryfossdotcom
cd ..
cp -r ./themes/tryfossdotcom/exampleSite/config ./src
rm config.toml
{{< /highlight >}}

My theme is already configured for a [configuration directory](https://gohugo.io/getting-started/configuration/#configuration-directory).

You can check it out at: https://github.com/yankiwi/tryfossdotcom-site/tree/main/src/config

But the key are elements are this.  You need to remove theme = from the _default/config.toml file.  Place this is the development/config.toml file.  You don't want it in production as you'll be using a module in production.

#### ./config/_default/config.toml
{{< highlight toml >}}
baseURL      = "/"
languageCode = "en-nz"
timeZone     = "Pacific/Auckland"
title        = "TryFOSS.com"
paginate     = 24
copyright    = "2023 TryFOSS.com.  All rights reserved."

[Author]
name  = "TryFOSS.com"
email = "patrick@tryfoss.com"
github = "tryfossdotcom"

[params]
rss              = true
comments         = true
readingTime      = true
wordCount        = true
useHLJS          = true
socialShare      = true
delayDisqus      = true
showRelatedPosts = true

[params.hero]
    title    = "TryFOSS.com"
    subtitle = "Trying to use FOSS where ever possible."
[params.hero.logo]
    src     = "/img/_logo.svg"
    height  = "64"
    width   = "64"
    alt     = "TryFOSS.com"

[[menu.main]]
    name       = "Home"
    url        = "/"
    weight     = 1

[[menu.main]]
    name       = "Posts"
    url        = "/post/"
    weight     = 2

[[menu.main]]
    name       = "About"
    url        = "/about/"
    weight     = 3

[[menu.main]]
    name       = "Tags"
    url        = "/tags/"
    weight     = 4

[[menu.main]]
    name       = "Subscribe"
    url        = "/subscribe/"
    weight     = 5

[module]
    [[module.imports]]
        path = "github.com/ionic-team/ionicons"
        disable = false
        [[module.imports.mounts]]
            source = "src/svg"
            target = "assets/svg/ionicons"
{{< /highlight >}}

#### ./config/production/config.toml
{{< highlight toml >}}
[module]
    [[module.imports]]
        path = "github.com/yankiwi/tryfossdotcom-theme"
        disable = false
        [[module.imports.mounts]]
            source = "archetypes"
            target = "archetypes"
        [[module.imports.mounts]]
            source = "assets"
            target = "assets"
        [[module.imports.mounts]]
            source = "data"
            target = "data"
        [[module.imports.mounts]]
            source = "layouts"
            target = "layouts"
        [[module.imports.mounts]]
            source = "static"
            target = "static"
    [[module.imports]]
        path = "github.com/yankiwi/tryfossdotcom-content"
        disable = false
        [[module.imports.mounts]]
            source = "."
            target = "content"
    [[module.imports]]
        path = "github.com/ionic-team/ionicons"
        disable = false
        [[module.imports.mounts]]
            source = "src/svg"
            target = "assets/svg/ionicons"
{{< /highlight >}}

#### ./config/development/config.toml
{{< highlight toml >}}
theme        = "tryfossdotcom"

[module]
    [[module.imports]]
        path = "github.com/ionic-team/ionicons"
        disable = false
        [[module.imports.mounts]]
            source = "src/svg"
            target = "assets/svg/ionicons"
{{< /highlight >}}

Run the build compose service once to create any modules needed in the development environment, such as [our Ionicons]({{< ref "2023-02-12_ionicons_in_hugo" >}} "our Ionicons") setup.
{{< highlight bash >}}
docker compose up build
{{< /highlight >}}

## Run the Development Container

{{< highlight bash >}}
docker compose up serve
{{< /highlight >}}
