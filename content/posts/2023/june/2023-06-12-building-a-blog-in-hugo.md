---
title: "Building a Blog in Hugo"
date: 2023-06-12T09:33:24+07:00
draft: false
tags: ["Programming"]
---

I tried to use Hugo a couple times before but I never got it to work properly. It was probably because I was just trying out the framework by following some tutorials. This time I went straight for the [Hugo doc](https://gohugo.io/getting-started/quick-start/) to learn the basics. Then I dug around the [Poison theme](https://github.com/lukeorth/poison) Github page for more information. It's a lot easier when you go directly to the source instead of hopping around tutorials.

## Hugo

### Install Hugo

To get started, I needed to install the extended edition of Hugo on Windows. There were several ways to do it, but the easiest option was using a package manager like [Chocolatey](https://chocolatey.org/) or [Scoop](https://scoop.sh/). I opted for Scoop because I used Chocolatey before and didn't like it.

Open PowerShell terminal on Windows and run:

{{< highlight pwsh >}}
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser  # Optional: Needed to run a remote script the first time
irm get.scoop.sh | iex
scoop install main/hugo-extended
{{< / highlight >}}

### Install Git

Now I needed to install Git for version control. It's simple for Windows. I went to [Git for Windows](https://git-scm.com/download/win) to download and install it.

## Build the Site

### New Site

I ran these commands using the Git bash I just installed:

{{< highlight sh >}}
hugo new site blog
cd blog
git init
git submodule add https://github.com/lukeorth/poison.git themes/poison
echo "theme = 'poison'" >> hugo.toml
hugo server -D
{{< / highlight >}}

### New Post

Now Hugo started a development server for my blog. It's pretty barebone because there's nothing there. The next thing I needed to do was creating a post and a page. I prefer to have my posts separated by year so I'll need to create the directory structure first.

{{< highlight sh >}}
mkdir content/posts
mkdir content/posts/2023
hugo new posts/2023/2023-06-11-first-post.md
{{< / highlight >}}

Hugo generated a new markdown file with the following front matter:

{{< highlight md >}}
---
title: "2023 06 11 First Post"
date: 2023-06-11T15:27:29+07:00
draft: true
---
{{< / highlight >}}

After writing a short post, I fixed the title, changed `draft` to `false`, and the post was good to go.

### New Page

I created a new page for my CV with the same process:

{{< highlight sh >}}
mkdir content/cv
touch content/cv/_index.md
{{< / highlight >}}

The reason why I used `_index.md` was to force the Poison theme to display the page on that path later. If I used `cv.md` instead, when I go to `kietnguyen.xyz/cv/`, it will actually have the link to the `cv.md` file in there. That would makes the path `kietnguyen.xyz/cv/cv/` for my CV to display. I was having a problem with this but I was able to find out about this quirk in [issue #59](https://github.com/lukeorth/poison/issues/59).

The same Github issue also let me know to add `layout: single` to my page front matter for the sidebar menu.

### Embed PDF

Since my CV is a PDF file, I needed a way to embed it. Luckily, someone already created a Hugo shortcode for it at [hugo-embed-pdf-shortcode](https://github.com/anvithks/hugo-embed-pdf-shortcode).

{{< highlight sh >}}
git submodule add  https://github.com/anvithks/hugo-embed-pdf-shortcode.git themes/hugo-embed-pdf-shortcode
{{< / highlight >}}

For it to work, I added the following lines to my `hugo.toml` later:

{{< highlight toml >}}
theme = ["hugo-embed-pdf-shortcode", "poison"]
enableInlineShortcodes = true
{{< / highlight >}}

### Modify `hugo.toml`

This step was straightforward. I just followed the example config in the Poison theme doc and modified a few things to fit my site.

{{< highlight toml >}}
baseURL = "https://kietnguyen.xyz/"
languageCode = "en-us"
title = "Kiet Nguyen"

theme = ["hugo-embed-pdf-shortcode", "poison"]
enableInlineShortcodes = true
paginate = 10
pluralizelisttitles = false  # Remove the automatically appended "s" on sidebar entries

[params]
    brand = "Kiet Nguyen"
    brand_image = "/images/avatar.jpg"
    description = "I'm a data analyst in Ho Chi Minh City. My blog is about projects that I find interesting."

    # Sidebar menu dict keys:
        # Name:         The name to display on the menu.
        # URL:          The directory relative to the content directory.
        # HasChildren:  If the directory"s files should be listed.  Default is true.
        # Limit:        If the files should be listed, how many should be shown.
    menu = [
        {Name = "CV", URL = "/cv/", HasChildren = false},
        {Name = "Posts", URL = "/posts/", Pre = "Recent", HasChildren = true, Limit = 5}
    ]

    # Social icons
    github_url = "https://github.com/kietnguyen01"
    linkedin_url = "https://www.linkedin.com/in/kietvnguyen/"

    # Sidebar light/dark mode button color
    moon_sun_color = "#FFE87C"
{{< / highlight >}}

A couple things to note here:

- I used double quotes `" "` for everything here. I tried to use single quote `' '` but it caused some `params` to not work.
- I had to use this exact format for the avatar `brand_image = "/images/avatar.jpg"` to work properly. Found this out in [issue 68](https://github.com/lukeorth/poison/issues/68).

## Deploy Blog

I decided to use [Netlify](https://www.netlify.com/) to publish my blog because it's free and easy to set up. 

### Configure Netlify

Netlify already streamlined the process of deploying from Github. I created a `netlify.toml` file and included the following:

{{< highlight toml >}}
[build]
    publish = "public"
    command = "hugo --gc --minify"

[context.production.environment]
    HUGO_VERSION = "0.113.0"
    HUGO_ENV = "production"
    HUGO_ENABLEGITINFO = "true"
{{< / highlight>}}

### DNS Setup

To make sure that my domain works with Netlify, I needed to do a few things:

1. Point my DNS records to Netlify's load balancer.
2. Redirect any traffic from `www.kietnguyen.xyz` to `kietnguyen.xyz`. 
3. Change my domain nameservers to Netlify nameservers.
4. Select automatic SSL/TLS certificate from Let's Encrypt.

All of these steps were painless. Netlify provided simple instructions for me to follow. I waited about a day for the DNS propagation to finish and my site is live.