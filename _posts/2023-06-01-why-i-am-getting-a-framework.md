---
title: Why I am getting a Framework
date: 2023-06-01T14:28:00+02:00
last_modified_at: 2023-06-01T16:49:00+02:00
categories:
  - blog
tags:
  - computers
---

First things first. This is my first post to my newish personal website, which
I'm hosting on GitHub Pages. It has all kinds of bells and whistles like
[LetsEncrypt][1] for SSL/TLS, pagination, and is fully statically generated
thanks to [Jekyll][2]. Big shout-out to [Michael Rose][3] for a really nice
template I'm using called [Minimal Mistakes][4]! Thanks a bunch.

I used to have a personal website back in the day to share ideas and thoughts,
and I figured I should pick that up again.. so here we are..


## Some background

Anyways, I've been on a quest for a new personal computing device for a while
now. I use a beefy desktop at home and at my office, but when I'm out and about
I usually have a laptop with me to hack around on.

Historically I've used ThinkPads (T43, X220, X230, W520, T14s), Macs (PowerBook
G4, Apple Cube, G5, MacBook Pro 14 2017), and even a few Surface devices (Surface
Pro 4, Surface Pro X).

## ThinkPads nowadays

I'm really not happy about the slow deterioration of build quality in ThinkPads.
It's been a slow ride downwards essentially since Lenovo took over the ThinkPad
line from IBM back in 2005ish.

Fast forward to today and it's a plasticky, flimsy shadow of its former self.
Next to that, configurations as stated in [PSREF][5] are not available in all
regions and you have to jump through hoops to get a configuration you like/need.

My most recent ThinkPad, a [T14s AMD gen3][6] is great feature-wise: It has
32GiB of RAM, a fast processor, 2TiB of storage and a 4K screen. But the build
quality is horrible - it squeeks when you lift it from the edges, the screen has
at least 5 dead pixels (within the year!) and the sound quality (which,
admittedly, has historically been bad with ThinkPads) is below average.

## Apple stuff

I've had a couple of Macs and the experience has mostly been great. Up till
about a little more than a year ago, I used a [MacBook Pro 13 2017][7] and I
was really happy with it. It was becoming a bit slow though.

I absolutely *adore* Apple build quality. It's really another dimension
compared to other manufacturers. Next to that, recent developments with regards
to the introduction of the M1 and M2 processors are astoundingly great too.
My only gripe with Apple is the slowly-but-surely dumbing-down of macOS, the
in-my-opinion-uneccessary removal of 32-bit support (which Mojave still had)
and a general feeling of uneasiness I have with the App Store being pushed more
and more as the only "safe" way of installing software (in general, a feeling
of being vendor-locked in).

I have been following [Asahi Linux][8] with much interest. I'm amazed how far
the project has come in such a short time! I don't think it is fully ready yet
for daily-driving, but as soon as it is, I would definitely hop aboard.


## The surfaces

I really love the 2-in-1 setup of the Surface Pro product line. I read a lot,
usually digitally, and the fact that you can use one device as a tablet and as
a computer is really nice.

Last year I've been using a [Surface Pro X][9] and it really is a cool little
device. Most applications nowadays have Windows-on-ARM native binaries, like
JetBrains, Python, Java, Node.js, etc, so, while not super-fast, it has been
fast enough. At night I use it to read books, comics and magazines.

The problem with Microsoft is not so much build quality as a whole, but
component quality. The build quality of Microsoft Surface devices is really
nice, no argument there. But they can't stand the test of time. After about 18
months issues start to appear, which turn out to be really common. I had the
following issues on both my Surface devices:

  * Kick-stand loosening due to regular wear
  * White/lighter spots on display due to normal handling
  * Type-cover stops working due to folding

The first issue is arguably understandable. Issues two and three are really
design and quality control issues. It is totally unacceptable for a tablet
device to develop screen pressure damage from simply holding the device. And
before you go and think this must be me, go do a search and see how many people
are suffering from this issue.

I've had three (3!) type covers in 2 years time, both broke in the same way
(the fold corner contains wires that go to the magnetic connector, which break
eventually due to wear-and-tear).


## The Framework

Ever since I've first heard of [Framework][10], I wanted to get one. First they
didn't sell in the Netherlands, where I'm at, and initially there were quite a
few growing pains, bugs, and little issues especially when running Linux
(according to brave early-adopters in the Framework forums at least). Nowadays
most of these issues are fixed and for common grievances fixes and workarounds
seem to be readily available.

Framework has recently opened pre-order for their 3rd generation devices and so
I bit the bullet and ordered an Intel 13th gen variant with 64Gib RAM and an
8TiB NVME (ordered seperately).

Actually, as I'm typing this, I just got a message in the mail telling me they
are preparing my batch!

By the way, you might be asking yourself why not the Ryzen variant? I love AMD,
their Linux support is super good, especially amdgpu in the Linux kernel is very
good. But since the last generation, AMD has built in support for the Microsoft
[Pluton][11] platform, which I find a bit too much customer-as-adversary for my
tastes. Pluton is woefully under-documented, but the best source for
understanding it is [a talk][12] from Tony Chen about Xbox One security, which
is the predecessor of Pluton.
{: .notice--info}

So I guess the biggest reason for me to go with Framework is the openness of the
platform, consequent repairability, and overal positive reviews with regards to
build quality. I think this might just be the ultimate Linux device for me!

I'll be running [Arch Linux][13] on mine. In a future post, I'll share how the
experience has been so far.

[1]: https://letsencrypt.org/
[2]: https://jekyllrb.com/
[3]: https://mademistakes.com/about/
[4]: https://mmistakes.github.io/minimal-mistakes/
[5]: https://psref.lenovo.com/
[6]: https://www.lenovo.com/us/en/p/laptops/thinkpad/thinkpadt/thinkpad-t14s-gen-3-(14-inch-amd)/len101t0015
[7]: https://support.apple.com/kb/SP754
[8]: https://asahilinux.org/
[9]: https://www.microsoft.com/en-us/d/surface-pro-x/8xtmb6c575md
[10]: https://frame.work/
[11]: https://learn.microsoft.com/en-us/windows/security/information-protection/pluton/microsoft-pluton-security-processor
[12]: https://www.youtube.com/watch?v=U7VwtOrwceo
[13]: https://archlinux.org/
