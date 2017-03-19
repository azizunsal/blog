+++
date = "2017-03-18T21:38:43+03:00"
title = "Hugo v0.19 baseURL Problem"
description = "It is easy to set up a personal blog with a static site generator like Hugo."
draft = false
tags = [ "hugo", "static-site-generator", "github", "github-pages"]
categories = [
  "hugo",
  "github"
]

+++

### Introduction

It is easy to set up a personal blog with a static site generator like [Hugo] (https://gohugo.io/). But there are a few tricky points when you want to use Hugo with Github Pages.

I chose the use a single repository for codes and posts. In that approach your site will have a URL that looks like https://username.github.io/projectname. For more information please go and check the Github Pages [documentation] (https://help.github.com/articles/user-organization-and-project-pages/).

After setting up your Hugo site, you can serve the site on localhost by running : `hugo server --buildDrafts`.

Now the site should be running on `http://localhost:1313/`.

The complete instructions for others steps can be found on the Hugo [website] (https://gohugo.io/overview/quickstart/).

Ok, so far so good.

### The Problem

As I said in the beginning we want to serve out site with Github Pages and we decided to apply the [Project Pages approach] (https://help.github.com/articles/user-organization-and-project-pages/#project-pages). The most important part in this approach is to have a URL like  https://username.github.io/projectname. For this to happen we should use `baseURL` property in our `config.toml` configuration file.

`baseURL = https://azizunsal.github.io/blog`

After adding this configuration, if you run `hugo server -D`, you will see that your page doesn't load CSS files!


After a lot of digging, I realized that the site couldn't load CSS files was because I had updated Hugo to version 0.19!

Other people has the same problem and there is an [issue](https://github.com/spf13/hugo/issues/2194) for this situation.

### Resolution
* Downgrade the Hugo to version 0.15
* Regenerate the site

And you're done.
