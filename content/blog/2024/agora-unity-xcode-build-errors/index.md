---
title: "[LiP] Upwork Adventures; Fixing a Unity/XCode Build Error"
# date: 2020-09-15T11:30:03+00:00
# description: "Brief guide to integrate Rust code into Unity"
weight: 20
# aliases: ["/p22"]
tags: ["LiP", "unity", "xcode", "agora", "build errors"]
# author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://mayonesso.com/blog/2024/agora-unity-xcode-build-errors"
disableHLJS: true # to disable highlightjs
disableShare: true
hideSummary: false
summary: "Errors on XCode when building Unity Projects with Agora Plugin"
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: true
cover:
    image: "cover.webp" # image path/url
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---
## • Introduction

So, inspired by [learn in public](https://www.swyx.io/learn-in-public) (yep, from that comes the \[LiP\] tag…) I am here sharing a little adventure that I came up with last week exploring UpWork trying to find something that my _zero-rating-zero-stars_ status could be enough for.

So, looking into Unity stuff I saw a problem in the signing process that someone has faced when building an OSX build.

{{< figure
    alt="Dog with war memories..."
    align=center
    caption="Yep, XCode build errors…"
    src="nam.webp"
>}}

But the good thing (and a smart move) from the job description was that all the repro steps were there. “Is this Unity version...”, “Is this specific plugin”, “These are the repro steps”, “And this is the error that we are getting”.

## • Digging into it…

While my gf was fighting with her landlord over emails before we went dining out. I downloaded the plugin, set the specific Unity version, and got the same error with the delightful repro steps.

{{< figure
    alt="XCode error"
    align=center
    caption="The easy part."
    src="error_output.webp"
>}}

Then I started to look at what could be the error, by the stack trace I saw that some files from the specific plugin that was described ([Agora](https://github.com/AgoraIO-Extensions/Agora-Unity-Quickstart)) weren’t signed properly, and the build would be completed if these files from the plugin were removed. So, yes, it was the plugin.

Further investigation, I saw some people having the same issue on the [plugin repo](https://github.com/AgoraIO-Extensions/Agora-Unity-Quickstart/issues/224), and some people having the same issue on other plugins or files in Stackoverflow it was clear that had something to do with the signature and a re-signature had to be made.

But then, exploring the plugin files I saw two scripts called `prep_codesign.sh` and `signcode.sh` ! In a careless move ( 🤞 no `rm -rf` pls!), I ran the scripts fixing the parameters / ENV variables that were asking me, but no luck.

The error on Xcode was still there and my girlfriend was almost done with the current round. I jumped back to the console and started to read the scripts a little bit more carefully, and then I recognized part of a path that they were using. But the pattern was slightly different. What if… swapped the line to match the structure that I had, ran the scripts, and voila! A green light on XCode!

{{< figure
    alt="XCode build result"
    align=center
    caption=""
    src="success.webp"
>}}

## • What's next?

This wasn’t a theoretical exercise, but a **weird** attempt to try to get my first job on UpWork, so I went to the platform, took some screenshots of before/after (I had to create a new project & fix it again), and wrote a “presentation” letter **completely different** from my other attempts where I try to, somehow with words, convince someone that I actually can do the job at hand.  
This time I just said that I was able to reproduce the error, that I found a fix, and that I was attaching screenshots of it.

**“Show, don’t tell”** in its maximum expression.

The next day I receive a message from the client and we start working on the solution. A _few_ simple steps that in the end I put together in a PostBuild script + a ScriptableObject to set up the required parameters ([repo here](https://github.com/mayo-nesso/AgoraPostBuildFix)).

## • Some lessons learned…

Maybe the most important one is that **if you don’t have a reputation in places like UpWork, you will have to work for reduced fees or even for free**.  
But the point is to do it smartly. Pick a ~job~ task that you can finish in a reasonable amount of time and just do it. Before even applying.

Two things could happen;</br>
a.- once you are done you offer the solution (with proof!), or</br>
b.- the job was taken and at least you learned something (hopefully).

Even with if the latter happens, try to achieve a balance in the karmic receive/give thing...
Make a post like this, put the solution to Github, share your experience…  
I don’t have good arguments on this one, but I think it does more good than harm.
