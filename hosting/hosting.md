---
title: Hosting a Server at Home
---


Hosting a Server at Home

I will now start a new series of posts about how I set up my home server. It hosts several applications, is accessible via the web (but only via HTTPS), and solves some problems for me in a nice way.

First of all, I had to figure out what hardware to use. If you already have a PC lying around, fine. If not, there are several options. My main requirements were that it shall run stably (and to a
point perform well) and not use too much power.

I started off with a Raspberry Pi 3+, which I had already, but soon learned that it wouldn't fulfill the first requirement. I think I burned three SD cards before giving up. There were most likely
just too many read/write accesses.

I then did a little research and figured out that a mini PC would best fulfill my requirements. Now, there are lots of them out there, and which one to choose mostly depends on what you want to do with
it. I decided to go for the [NiPoGi AK2 Plus](https://amzn.to/3Wm7Gkb). It was not too expensive and has enough power for what I need. I updated it by adding a second 512GB SSD, which I already owned.

Later, I added an external USB HDD as a backup and storage drive for all the data. It is a [5TB USB-3.0 drive from Western Digital](https://amzn.to/4aFJeyc). After figuring out how to get the HDD to go to sleep when not actively
used, I was very satisfied with this setup.

I usually run the PC headless, but I can connect it to the HDMI input of one of my monitors when needed. Most likely, a TV would also work for these rare cases.

Now that the hardware was there, I could start setting it up. I'll describe the setup in the sequence I would now consider best, but for sure, I experimented a lot, and it was not as straightforward when
I installed things. 😀

/info This page contains affiliate links to Amazon. Feel free to order your hardware anywhere, but it would be nice of you to use these links so I get a small reward for all this information. 😉

* [Ubuntu Installation](/Hosting%20a%20Server%20at%20Home/Ubuntu%20Installation/)
* [Get Idle Hard Drives to Sleep](/Hosting%20a%20Server%20at%20Home/Get%20Idle%20Hard%20Drives%20to%20Sleep/)
* [Setting up your own Domain](/Hosting%20a%20Server%20at%20Home/Setting%20up%20your%20own%20Domain/)
* [Extend Default Ubuntu LVM Partition](/Hosting%20a%20Server%20at%20Home/Extend%20Default%20Ubuntu%20LVM%20Partition/)
* [Docker Installation](docker)
