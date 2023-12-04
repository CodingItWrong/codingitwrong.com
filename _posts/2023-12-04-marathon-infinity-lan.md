---
title: How to Play Marathon over a LAN on Classic Macs
---

**tl;dr: to play the original Classic Mac Marathon games over a LAN, enable AppleTalk and set it to connect via Ethernet. No TCP/IP configuration is needed.**

***

One of my favorite memories growing up a Mac user was LAN gaming with my brothers playing Marathon. When I started collecting Macs again, being able to play Marathon on a LAN was one of my main goals. I had trouble getting it working, and now that I’ve found the answer, I wanted to share it so it’ll show up in others’ search results.

Note that another option to play Marathon over the network is [Aleph One](https://alephone.lhowon.org/), a project for running Marathon on modern machines. But in my case I wanted the authentic original experience on original hardware.

First, get Macs with classic Mac OS and a working Ethernet port. All the Macs I used had on-board Ethernet:

- Power Macintosh 7500
- Power Macintosh 8500
- Twentieth Anniversary Macintosh
- Powerbook G3 Pismo
- Power Mac G3 Tower
- Power Mac G4 Tower (MDD)

Get the Macs connected over Ethernet. I used a Netgear WPN824 router.

Next, enable AppleTalk and configure it to run over Ethernet.

If you have an AppleTalk control panel, open it and choose “Ethernet” from the dropdown.

![The Mac OS 9 AppleTalk control panel, with Ethernet selected from the "Connect via" dropdown.](/img/posts/marathon-lan/appletalk-over-ethernet.png)

On some computers running pre-Open Transport, you may need to do this in the Network control panel instead of AppleTalk.

Next, make sure AppleTalk is enabled. You can do this one of two places. The first is in Chooser:

![The Mac OS 9 Chooser, showing AppleTalk as Active](/img/posts/marathon-lan/chooser-appletalk-active.png)

And the second is in the AppleTalk Control Strip module:

![The Mac OS 9 AppleTalk control strip module, with AppleTalk Active selected](/img/posts/marathon-lan/control-strip-appletalk-active.png)

Note that you do *not* need to do any TCP/IP configuration at all. 2023 me equated Ethernet with TCP/IP, but I forgot that TCP/IP seems to be an *alternative* to AppleTalk.

At this point, you can test the AppleTalk connectivity without needing to go into Marathon using file sharing. Open the File Sharing control panel, make sure there is an owner name, password, and computer name, and click Start File Sharing.

![The Mac OS 9 File Sharing control panel, with File Sharing on](/img/posts/marathon-lan/file-sharing-on.png)

Then open Chooser and pick AppleShare. If the machines are connected, they should all see each other in the list of servers.

![The Mac OS 9 Chooser, with AppleShare selected and a file server named "Josh's TAM" showing in the list](/img/posts/marathon-lan/chooser-appleshare.png)

Next you'll need one of the Marathon games installed on each machine. There is a Marathon [serial number generator](https://marathon.bungie.org/maraserialgen/) that Bungie is not opposed to, so you can get a different serial number for each machine. I went with Marathon Infinity.

![The Marathon Serial Generator, with generated serial numbers blurred out](/img/posts/marathon-lan/serial-generator.jpg)

Start the games up. Pick one machine to be the server (probably makes sense to choose the highest-powered one) and click Gather Network Game on it. Make sure Ethernet is chosen from the Network dropdown. Enter a name, color, team, and any game settings you want. Then click OK.

![The Marathon Infinity Setup Network Game dialog, configured to use Ethernet as the Network](/img/posts/marathon-lan/setup-network-game.png)

On the other machines, choose Join Network Game instead. Enter a name, color, and team for each.

Once you're done, click Join on each. Back on the Gather machine, all the machines' names should appear in the "Players On Network" list.

![The Marathon Infinity Gather Network Game dialog, with a player named Pismo in game and a player named TAM on the network](/img/posts/marathon-lan/gather-network-game.png)

Click each name and click Add.

![The Marathon Infinity Gather Network Game dialog, with players named Pismo and TAM in game](/img/posts/marathon-lan/gather-network-game-added.png)

Then click OK and the game will start!

*Thanks to cheesestraws and treellama on 68kmla.org for additions and corrections!*
