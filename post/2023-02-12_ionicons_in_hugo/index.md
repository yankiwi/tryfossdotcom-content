---
title: Using Ionicons in Hugo
date: 2023-02-12
tags: ["hugo", "ionicons"]
draft: false
---

## Introduction

[Ionicons](https://ionic.io/ionicons/) is a great icon set created by the [Ionic](https://ionic.io/) team.  Whereas a lot of sets have pulled back from including logos, Ionicons has a large set builtin, it's MIT licensed, and looks great on web, Android and iOS.  Because it supports both SVG and web font there are a lot of ways to use Ionicons in your project.  This site uses [Hugo](https://gohugo.io/) so my goal is make Ionicons available both as a partial and shortcode without Javascript.

{{< ionicon icon="logo-ionic" >}}{{< ionicon icon="arrow-forward-outline" >}}{{< ionicon icon="happy-outline" >}}

## Get the Icons

To get the Ionicons SVG files, I'll be using [Hugo Modules](https://gohugo.io/hugo-modules/).  Modules are nice because they let you specify a single file or directory within a git repository.  In our case, we just want the svg directory.  So I add the following to my config.toml file.

{{< highlight toml >}}
[module]
    [[module.imports]]
        path = "github.com/ionic-team/ionicons"
        disable = false
        [[module.imports.mounts]]
            source = "src/svg"
            target = "assets/svg/ionicon"
{{< /highlight >}}

## Create Partial

I've created a file called ionicon.html in my theme's layout/partials directory with the following content:
{{< highlight html >}}
{{ $icon := .icon | default "logo-ionic" }}
{{ $width := .width | default "24"}}
{{ $height := .height | default "24" }}
{{ $color := .color | default "currentColor" }}
{{ $iconres := resources.Get (printf "/svg/ionicons/%s.svg" $icon) }}
{{ $svg := ($iconres.Content) }}
{{ $svg := (replaceRE `^.*?>(.*?)<\/svg>$` "$1" $svg) }}
{{ (printf `<svg
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 512 512"
    width="%s"
    height="%s"
    fill="%s"
  >%s</svg>` $width $height $color $svg) | safeHTML }}
{{< /highlight >}}
{{< elink "https://github.com/yankiwi/tryfossdotcom-theme/blob/main/layouts/partials/ionicon.html" >}}
View on: {{< ionicon "logo-github" "16" "16" >}}
{{< /elink >}}

Which I can then call in other layout files...
{{< nohighlight >}}{{ partial "ionicon" (dict "icon" "logo-ionic" )}}{{< /nohighlight >}}
...or...

{{< nohighlight >}}{{ partial "ionicon" (dict "icon" "logo-ionic" "width" "32" height "32" color "#450000" )}}{{< /nohighlight >}}

## Create Shortcode

But I also want to be able to call it from content files such as this post.  So I've also create a files called ionicon.html in my theme's layout/shortcodes directory with the following content.

{{< highlight html >}}
{{ $icon := "logo-ionic"}}
{{ $width := "24"}}
{{ $height := "24" }}
{{ $color := "currentColor" }}
{{ if .IsNamedParams }}
    {{ if .Get "icon" }}
        {{ $icon = (.Get "icon") }}
    {{ end }}
    {{ if .Get "width" }}
        {{ $width = (.Get "width") }}
    {{ end }}
    {{ if .Get "height" }}
        {{ $height = (.Get "height") }}
    {{ end }}
    {{ if .Get "color" }}
        {{ $color = (.Get "color") }}
    {{ end }}
{{ else }}
    {{ if .Get 0 }}
        {{ $icon = (.Get 0) }}
    {{ end }}
    {{ if .Get 1 }}
        {{ $width = (.Get 1) }}
    {{ end }}
    {{ if .Get 2 }}
        {{ $height = (.Get 2) }}
    {{ end }}
    {{ if .Get 3 }}
        {{ $color = (.Get 3) }}
    {{ end }}
{{ end }}
{{ $iconRes := resources.Get (printf "/svg/ionicons/%s.svg" $icon) }}
{{ $svg := ($iconRes.Content) }}
{{ $svg := (replaceRE `^.*?>(.*?)<\/svg>$` "$1" $svg) }}
{{ (printf `<svg
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 512 512"
    width="%s"
    height="%s"
    fill="%s"
  >%s</svg>` $width $height $color $svg) | safeHTML }}
{{< /highlight >}}
{{< elink "https://github.com/yankiwi/tryfossdotcom-theme/blob/main/layouts/shortcodes/ionicon.html" >}}
View on: {{< ionicon "logo-github" "16" "16" >}}
{{< /elink >}}

Which I can then call in content files...
{{< nohighlight >}}{&lbrace;< ionicon "logo-ionic" >&rbrace;}{{< /nohighlight >}}
{{< ionicon "logo-ionic" >}}

{{< nohighlight >}}{&lbrace;< ionicon icon="logo-ionic" >&rbrace;}{{< /nohighlight >}}
{{< ionicon icon="logo-ionic" >}}

{{< nohighlight >}}{&lbrace;< ionicon "logo-ionic" "32" "32" "#ff0000" >&rbrace;}{{< /nohighlight >}}
{{< ionicon "logo-ionic" "32" "32" "#ff0000" >}}

{{< nohighlight >}}{&lbrace;< ionicon icon="logo-ionic" width="32" height="32" color="#00ff00" >&rbrace;}{{< /nohighlight >}}
{{< ionicon icon="logo-ionic" width="32" height="32" color="#00ff00" >}}