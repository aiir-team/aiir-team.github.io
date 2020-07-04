---
layout: post
author: thieunguyen5991
categories: [video, server]
---

Update (6 April 2020): Having just updated my server, it appears that the latest version of Jitsi leads to
  some issues. I ended up reinstalling using the quick install guide, which worked without any issues
  . Best to go with that for now!
<!--more-->

Like a lot of the world, we're in "lockdown" over here, with only a few businesses allowed to open
. Obviously a lot of people are now working from home, so videoconferencing is now a key tool in everyone's
 arsenal. The defacto option appears to be Zoom, but they have a fairly sketchy record on user privacy
  which has left many people questioning whether they ought to be trusted. 

I used a Nanode from Linode, but the smallest Droplet from Digital Ocean would also be fine, and both have
 promotions with free credit which would be perfect for a project like this. In terms of OS, I opted for
  Debian 10 (buster), but the following shouldn't be too different for any similar system.

Once you're logged in, start by updating everything and installing gpg (for the Jitsi repository) and the
 nginx webserver. Installing nginx first is the main change I made from the official documentation, but
  fixed some issues I was seeing where both nginx and the Jitsi module ended up listening on the same port
  , preventing nginx from starting.