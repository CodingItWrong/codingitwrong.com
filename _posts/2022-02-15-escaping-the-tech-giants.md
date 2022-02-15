---
title: "Escaping the Tech Giants"
---

This is a post I'll keep up-to-date with switches I've made from big-tech platforms and tools to independent and open source ones. My main motivations for doing so are data privacy and concerns with the ethics and societal effects of these companies.

I'm hoping the details will help convince and equip others to make the change themselves. If you have any questions, feel free to post a comment or contact me via any of the contact methods on [my home page](/).

Here are the changes I've made so far:

- [Browser: Chrome -> Firefox](#browser)
- [Ebooks: Apple Books -> InformIT and Kindle](#ebooks)
- [Editor: Atom -> Sublime Text](#editor)
- [Email: Apple Mail -> Fastmail](#email)
- [Image Editing: Photoshop -> Gimp](#images)
- [Laptop OS: macOS -> added Pop!_OS](#os)
- [Notes: Bear Notes -> Simplenote](#notes)
- [Office: Apple Work -> LibreOffice](#office)
- [Social Media: Twitter -> Mastodon, Feedly](#social)
- [Video Editing: iMovie -> Kdenlive](#video)

<a name="browser"></a>
# Browser: Chrome -> Firefox

Like most other web users, I used Chrome for a long time. But the last thing I want to do is for Google to know everything I do online. Instead, I switched to [Firefox](https://getfirefox.com). I don't want Microsoft or Apple to know what I'm doing either. There are some concerns about Firefox monetizing and recently getting into talks with Meta, so I may consider a non-business-based browser fork in the future. But browsers with less adoption have less compatibility with tools I need like 1Password.

<a name="ebooks"></a>
# Ebooks: Apple Books -> InformIT and Kindle

I do a lot of reading on my phone in spare moments. Apple Books is locked into Apple platforms. By contrast, DRM-free ebook providers provide books in multiple formats, including .epub (Apple Books, maybe others), .mobi (Kindle), and .pdf (platform-agnostic). [InformIT](https://www.informit.com/) is a great provider of programming books from many publishers, and they provide ebooks DRM-free. [Pragmatic Programmer](https://pragprog.com/) is one popular publisher that offers DRM-free ebooks as well. If I want a book that's only available from Apple Books and Kindle, I prefer Kindle since it can run on more than just Apple platforms.

<a name="editor"></a>
# Editor: Atom and VS Code -> Sublime Text

I prefer GUI editors rather than (at the one extreme) IDEs or (at the other extreme) CLI-based editors like vim. Atom has been my main editor for a long time. Visual Studio Code is very popular in the JavaScript world. Unfortunately, VS Code has always been owned by Microsoft, and when GitHub was bought by Microsoft Atom became theirs as well. To ensure Microsoft doesn't have access to my data, I switched to [Sublime Text](https://www.sublimetext.com/). I had used it in the past. It's paid and a bit harder to set up for my use, but worth it to support independent software developers. As a bonus, it's not implemented with Electron, so may be somewhat more performant.

<a name="email"></a>
# Email: Apple Mail -> Fastmail

When I found out [Apple scans individuals' iCloud mail data](https://9to5mac.com/2021/08/23/apple-scans-icloud-mail-for-csam/), I decided I wanted to switch off of them. I ended up going with [Fastmail](https://www.fastmail.com) as an affordable option. They're accessible via IMAP, but their first-party web and iOS clients provide more features and prevent me from providing my login info to Apple. I also use Fastmail's [Masted Email](https://www.fastmail.com/1password/) feature to easily create login email addresses for different services, so that if my login is leaked I can deactivate it and people can't correlate me with other services.

<a name="images"></a>
# Image Editing: Photoshop -> Gimp

Way back in the past I used Photoshop for image editing, but my needs are so lightweight (especially as CSS has improved) that it hasn't been worth me paying for for a long time. Instead, I use [Gimp](https://www.gimp.org/), an open-source image editor based on Photoshop.

<a name="os"></a>
# Laptop OS: macOS -> added Pop!\_OS

macOS (and its predecessor Macintosh operating systems) has been my desktop/laptop OS for my whole life. Of the tech giants, I distrust Apple the least, but since they have the ability and willingness to scan users' email accounts, I'd prefer to have an open-source option. I've used Linux for servers for years, but have just recently bought a secondhand [System76](https://system76.com/). I've found that it's easy to install the main software I need on Debian-based distros; currently I'm using System76's own [Pop!_OS](https://pop.system76.com/). Note that I'm still using macOS for some features: development of personal iOS projects, and my voice dictation software runs better on my newer macOS hardware. But I'd like to migrate as much of my computing to Linux as I can. And, at least, having the option helps ensure I don't get locked in to one platform.

<a name="notes"></a>
# Notes: Bear Notes -> Simplenote

I've been using [Bear Notes](https://bear.app/) for note taking on macOS and iOS, but because it's only available for Apple platforms it locks me in from switching to Linux. Instead, I switched to [Simplenote](https://simplenote.com/), a similar tool that is available on the web and all platforms.

<a name="office"></a>
# Office: Google Docs, Apple Work -> LibreOffice

I used Google Docs until I started getting concerned about Google's privacy violations. I switched to Apple Work, but that locks me in to Apple platforms. Instead, I switched to [LibreOffice](https://www.libreoffice.org/), an open-source office suite available for all platform including Linux.

<a name="social"></a>
# Social Media: Facebook, Twitter -> Mastodon, Feedly

I used to be a very heavy Twitter user, and lighter on Facebook. But because of their documented use of surveillance, behavior manipulation, and lack of moderation of harmful content, I deleted my Facebook account, and stopped using Twitter. Instead, I switched to Mastodon, an open-source social network. Mastodon has different servers and clients; I use the [mastodon.technology](https://mastodon.technology) server and the [Toot!](https://apps.apple.com/us/app/toot/id1229021451) iOS client. To get updates from news sources that don't have Mastodon accounts, I use [Feedly](https://feedly.com/) to follow RSS feeds.

<a name="video"></a>
# Video Editing: iMovie -> Kdenlive

I occasionally live-stream on Twitch. I do lightweight editing of Twitch stream videos and sometimes calls from work. Previously I used iMovie, but beyond Apple lock-in, it's started to be unreliable downloading, to the point that it prevents any other Mac App Store installs, even after restart. Instead, I've switched to [Kdenlive](https://kdenlive.org), an open-source video editing app.
