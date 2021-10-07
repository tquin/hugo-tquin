---
title: "How I Build This Site: Working with Hugo on WSL"
date: 2021-10-07T18:07:00
draft: false
tags:
  - hugo
  - wsl
  - linux
  - bash
  - github
  - netlify
---

I build the test environment for this site on WSL - which comes with it's own [quirks and features][link_doug] working with Hugo.

<!--more-->

---

## Wait, what's Hugo?

[Hugo][link_hugo] is a _static content blogging engine_ - buzzwords aside, it's just a fun little program that lets you write simple Markdown files and renders them out into a fancy website like this one! It supports a bunch of community-made themes, easy integrations with things like images, tweets, Github Gists - you get the idea. You just write your files in a [folder structure][link_folder_struct], run `hugo`, ???, profit. Dead simple, and the static HTML approach makes things **fast.**

For example, here's what the first few lines of my [Splunk tstats and PREFIX][link_prefix] post looks like:

{{< figure src="/posts/hugo_on_wsl/markdown.png" class="smaller_img" alt="img_markdown" >}}

Pretty simple, right? To publish, I just push these files up to [Github][link_github], and Netlify picks them up and hosts them for free.

When you're writing a new post, though, before you publish it to the world, you can run it in a local webserver to see what it looks like. And there's the catch - for me, using WSL (Windows Subsystem for Linux), I couldn't get this test site to work. 

---

### The issue with WSL

Since the Linux box running in WSL is completely independent of your Windows install, it has it's own local IP. That means that going to the default Hugo server URL of `http://localhost:1313` in my Windows browser wouldn't be able to reach it - it would look for that webserver on the Windows machine, not in the WSL environment.

The solution to this is a pretty simple bash function that uses Hugo's `bind` CLI parameter. And, well, here it is:

{{< gist tquin 98eaa40004df298a5ee5a26e8f5cb300 >}}

This is just finding your WSL environment's LAN IP _(it's likely to change with every reboot)_ and tells Hugo to open the dev webserver up there, instead of the default `127.0.0.1`. The other options here are pretty minor - show draft or future posts, clean up the cache, blah blah blah.

So that's it! Running `hugo_server` in WSL will give you a nice clicky button to the right URL to access the test site.

{{< figure src="/posts/hugo_on_wsl/hugo_server.png" class="smaller_img" alt="img_markdown" >}}

Adding another alias to `cd` into the right directory makes my build process as simple as `hugo_tquin` -> `hugo_server`, and I'm good to go!

Hopefully, this helped you if you were madly googling for a solution. My thanks to [Jan Reilink][link_saotn] for the initial inspiration.






[link_doug]: https://www.youtube.com/c/DougDeMuro
[link_hugo]: https://gohugo.io/
[link_folder_struct]: https://gohugo.io/content-management/page-resources/
[link_prefix]: {{< ref "/posts/exploring_splunk_prefix" >}}
[link_github]: https://github.com/tquin/hugo-tquin
[link_saotn]: https://www.saotn.org/hugo-development-environment-in-wsl-2/