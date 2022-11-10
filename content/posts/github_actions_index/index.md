---
title: "Github Actions: How To Check an Index File was Updated"
date: 2022-11-10
draft: false
tags:
  - github
  - actions
  - automation
  - ci/cd
---

I\... am forgetful, and need a reminder to update my biscuit ranking every now and then. How can we achieve that with CI/CD automation?

<!--more-->

---

## The Problem

But first, let's go into a little more detail on the exact problem I'm trying to solve here to see if it applies to you.

A large chunk of this website is not the few-and-far-between technical posts you're here reading, but instead is a tribute to core Australiana: biscuit reviews. As part of that, I maintain [the ranking list](/arnotts/) with every single bickie included -- at least in theory. The trouble is, I can be forgetful, and push new reviews live but forget to rank it first. And that sounds like a thing we can automate!

{{< figure src="/posts/github_actions_index/action_example.png" alt="img_action_example" >}} <!-- class="smaller_img" -->

## Lights, Camera: Trigger the Action

I'm sure most people are familiar with GitHub Actions by now, but in short, it's a CI/CD automation platform like Jenkins or Ansible. You develop a workflow and tell GitHub what to trigger it on, and it can kick off scripts and do a bunch of cool things for you. For example, [CodeQL][link_codeql] is a common one to check for common code vulnerabilities before you merge, so you don't accidentally commit your AWS key to prod.

In this example, we're going to be kicking this off based on new pull requests, but the trouble is that _not every pull request is new content._ Sometimes, I'm just fixing a typo, editing a theme, doing backend stuff, yadda yadda yadda. Or maybe, I'm pushing to a dev branch, in which case I don't really care whether all the T's were crossed and I's dotted. It might also be a `content/posts` change, rather than a `content/arnotts` one, where these technical sorts of posts obviously don't require the index to be updated. So step one, filter and trigger the action:

* For new pull requests
* To the main branch
* Which change some file in my Arnott's directory

```
on:
  # Triggers the workflow on push or pull request events but only for the "main" branch
  pull_request:
    branches: [ "main" ]
    paths:
      - 'content/arnotts/**'
```

## Get to the Goods

Now that we've found an applicable PR to run against, we need something that actually detects if the index file changed or not. `git diff` would be an easy way to do this; the only complication is that we'd need to be able to access the details of the commit from within the action environment. Luckily, that's a freebie; GitHub has their own `checkout` action that provides you with exactly that. After calling it, we can just run a diff and `grep` for the index file we want. Since that will already either output something to stdout or not, which is effectively a true/false output, that's all we need to cause the action to fail or succeed.

```
steps:

  - name: Checkout code
    uses: actions/checkout@v1
    with:
      ref: ${{ github.sha }}

  # Runs a set of commands using the runners shell
  - name: Verify that the index file has been edited in this pull req compared to the base
    run: git diff --name-only ${{ github.event.pull_request.head.sha }} ${{ github.event.pull_request.base.sha }} | grep content/arnotts/_index.md
```

Full code is available on [GitHub][link_action] if you want a direct copy + paste, but the snippets here should give a good overview of the key bits and pieces. Good luck!



[link_action]: https://github.com/tquin/hugo-tquin/blob/main/.github/workflows/arnotts_index_checker.yml
[link_codeql]: https://github.com/github/codeql-action