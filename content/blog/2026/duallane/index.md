---
title: "DualLane — A macOS App Switcher That Works the Way You Do"
weight: 1
tags: ["macOS", "Swift", "SwiftUI", "indie"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://mayonesso.com/blog/2026/duallane/"
disableHLJS: true
disableShare: true
hideSummary: false
summary: "DualLane is a free macOS app switcher that replaces Cmd+Tab with a two-lane interface — primary apps up top, everything else below."
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: true
---

## • What is DualLane?

So, a weird update — a long time without any activity here.
But just passing by to share a little experiment I've been working on (more than the AI of the day, right? nowadays everything seems to be under the hood of agents).

So, kind of, if you don't find the solution out there, fix it yourself. The idea is simple: not all open apps are equally important. You have a handful you switch to constantly — your editor, browser, terminal — and then a long tail of everything else, but you still need them to be 'there', like your music app.

If your hands are close to the keyboard most of the time, it's highly probable that you're familiar with `Cmd+Tab`.
But the Mac default switcher treats all apps the same, which means more scanning and more keystrokes.

So the app I'm sharing here is [DualLane](https://duallane.app) — a macOS app switcher built to replace the default `Cmd+Tab` experience with something more intentional (and efficient).

DualLane splits things into two rows:

- **Primary lane** — the apps you put there. Your go-to tools, always at the top.
- **Secondary lane** — everything else, out of the way but reachable.

![DualLane in action](demo.gif)

