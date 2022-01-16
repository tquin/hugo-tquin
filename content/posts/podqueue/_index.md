---
title: "Podqueue - a simple podcast archiver, written in Python"
date: 2022-01-11T09:05:00
draft: false
tags:
  - python
  - zfs
  - podqueue
  - podcast
  - development
---

`podqueue` is a simple Python script to archive your favourite podcasts.

<!--more-->

Have you ever wanted to go back to listen to a great old episode, and it's disappeared from the internet? Some podcasts only keep a [few recent episodes][link_amt] in their feeds, or maybe the creator abandoned their project and [no one paid for the web hosting][link_abstract]. Whatever the reason, I want to keep my podcast queue archived. _(Maybe I'm just finding excuses for upgrading my ZFS pool - who can say?)_ There are other solutions to this, but honestly, I was looking for a fun project and this seemed perfect.

Enter podqueue: a simple solution to download your entire subscription list to disk, with some nice features:

* **Super simple configuration** - all you need is an input (your subscription list) and an output (directory).
* Archive the **show notes** in simple JSON and **cover art** along with all the audio files.
* **Portable** - written in Python, podqueue is available on any platform.
* **Set it and forget it** - run podqueue on a cron or systemd timer, and (hopefully) never think about it again.

After it's downloaded the back catalogue, you'll get a directory structure like this, with one `.json` and one audio file (probably `.mp3`) per episode.

```
output/
├─ Accidental_Tech_Podcast/
├── episodes/
│──├── 2021-12-30_463_No_Indication_of_Progress.json
│──├── 2021-12-30_463_No_Indication_of_Progress.mp3
│──├── 2022-01-06_464_Monks_at_Drafting_Tables.json
│──├── 2022-01-06_464_Monks_at_Drafting_Tables.mp3
│──├── ...
├── Accidental_Tech_Podcast.png
├── Accidental_Tech_Podcast.json
├─ The_Pen_Addict/
├── episodes/
│──├── 2021-12-29_494_The_Centre_is_Twisty.json
│──├── 2021-12-29_494_The_Centre_is_Twisty.mp3
│──├── 2022-01-05_495_Parter_Jocker.json
│──├── 2022-01-05_495_Parter_Jocker.mp3
│──├── ...
├── The_Pen_Addict.png
├── The_Pen_Addict.json

```

---

## Where do I get it?

Currently, podqueue is available in two places:

* https://github.com/tquin/podqueue
* https://pypi.org/project/podqueue

But I plan on releasing a docker container or an AUR package to simplify the process a little bit.

Installation can be done with Python's pip:

```
python3 -m pip install --upgrade podqueue
python3 -m podqueue --help
```

Or you can just clone the git repo directly and run it that way:

```
git clone https://github.com/tquin/podqueue
cd podqueue/
python3 src/main.py --help
```

---

## Is my podcast app supported?

_... Maybe._ Most third-party, indie-developed apps support the OPML open standard, which is how you export your subscription list for podqueue to read. Apple and Google Podcasts both don't have an official method, but there are some open source tools that _might_ be able to help you. Spotify and Stitcher, though, because they use proprietary systems and not open standards, both don't offer an export to OPML - you'll need to copy your podcasts into an app that does and export that way.

Key: ✅ _the good,_ ❌ _the bad, and_ 🔨 _the workaround_

<!--
{{< figure src="/posts/podqueue/icon_pocketcasts.png" class="icon_fig" class="icon_fig" height="50px" width="50px" alt="img_pocketcasts" >}}
-->

|Icon|Podcast App|Supported|OPML Export Options|
|---|---|---|---|
|{{< figure src="/posts/podqueue/icon_pocketcasts.png" class="icon_fig" class="icon_fig" height="50px" width="50px" alt="img_pocketcasts" >}}|Pocket Casts|✅|[OPML export](https://support.pocketcasts.com/article/exporting-an-opml/)|
|{{< figure src="/posts/podqueue/icon_overcast.png" class="icon_fig" height="50px" width="50px"  alt="img_overcast" >}}|Overcast|✅|Option available in the app's Settings page, or [here on the web.](https://overcast.fm/account/export_opml)|
|{{< figure src="/posts/podqueue/icon_castro.png" class="icon_fig" height="50px" width="50px"  alt="img_castro" >}}|Castro|✅|[Export Subscriptions](https://castro.fm/support/export-subscriptions)|
|{{< figure src="/posts/podqueue/icon_downcast.png" class="icon_fig" height="50px" width="50px"  alt="img_downcast" >}}|Downcast|✅|[Exporting Podcast Subscriptions](https://support.downcast.fm/article/vYyHP2SOOc-exporting-podcast-subscriptions)|
|{{< figure src="/posts/podqueue/icon_podcastaddict.png" class="icon_fig" height="50px" width="50px"  alt="img_podcastaddict" >}}|Podcast Addict|✅|[How can I backup and restore my subscription & data?](https://podcastaddict.com/faq/20)|
|{{< figure src="/posts/podqueue/icon_castbox.png" class="icon_fig" height="50px" width="50px"  alt="img_castbox" >}}|Castbox|✅|[OPML Export](https://helpcenter.castbox.fm/portal/en/kb/articles/settings-on-the-personal-tab-android#OPML_Export)| 
|{{< figure src="/posts/podqueue/icon_apple.png" class="icon_fig" height="50px" width="50px"  alt="img_apple" >}}|Apple Podcasts|🔨|Not available in iOS app or macOS since Catalina. However, if you sync your podcasts to your Mac, there is an [open-source workaround.](https://liujiacai.net/podcasts-opml-exporter/)|
|{{< figure src="/posts/podqueue/icon_google.png" class="icon_fig" height="50px" width="50px"  alt="img_google" >}}|Google Podcasts|🔨|Officially unavailable. There is a [Gist by @telmen](https://gist.github.com/telmen/4d67cba98ba7181424a681c1cbfc5f34) (I tested, seems to work) that can be run in your browser's Devtools if you're feeling lucky.|
|{{< figure src="/posts/podqueue/icon_spotify.png" class="icon_fig" height="50px" width="50px"  alt="img_spotify" >}}|Spotify|❌|Not available, since Spotify doesn't use open Podcast standards. Community suggestion is ['now reaching the internal teams at Spotify'](https://community.spotify.com/t5/Live-Ideas/Podcasts-Import-for-Podcasts-OPML/idi-p/4423445), as of six months ago.|
|{{< figure src="/posts/podqueue/icon_stitcher.png" class="icon_fig" height="50px" width="50px"  alt="img_stitcher" >}}|Stitcher|❌|Not available.|


[link_amt]: https://answermethispodcast.com/
[link_abstract]: https://www.instagram.com/abstractnonsenseftp/
