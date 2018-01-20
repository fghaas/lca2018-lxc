<!-- .slide: data-background-image="images/citynetwork-logo.svg"  data-background-size="10% 10%" data-background-position="10% 10%" -->
## Please Contain Me

Practical LXC on the Desktop

* * *

Florian Haas | [@xahteiwi](https://twitter.com/xahteiwi)

linux.conf.au 2018 | January 26, 2018

Note: Hi, im Florian, and it’s wonderful to be able to be here and
speak at LCA again, and thanks a bunch for coming not only on the
final afternoon of the conference, but also to the talk that is
scheduled directly opposite Sage, which I would have loved to attend
as well — I guess we’ll all catch the video, because that talk will
surely be a highly informative one on Ceph. But here, we want to talk
about something that hasn’t anything to do with storage, not even with
servers much — it’s all about managing your desktop in a manner that,
I hope you’ll agree, is simple and effective and gets the job
done.

That said, I do want to mention upfront that nothing I talk about here
is The Only Way. There are several ways to run a containerized
desktop, I’ve just found the one I’m describing here to be useful, and
I’ve been using it on a daily basis for something like a year and a
half now.


# Why?

Note: Why would you want to run applications in containers on your
desktop? Specifically, why in _system_ containers?


## Keeping `/` lean

Note: I’d like to have as few packages in my root filesystem as
possible. Now, granted, that’s still quite a few: this laptop, an
Ubuntu 16.04 box, has over 2,000 `.deb` packages installed in its root
filesystem. But it would be far more if I weren’t using
containers. All those packages need to be kept updated and patched,
and the fewer there are, the faster my daily updates get.

What isn’t installed also can’t launch a service that is running in
the background, and consume cycles, and thus eat power.


Multiple 
## parallel versions

Note: I frequently need to install multiple _packaged_ versions of
some software on my laptop.

My personal prime example is OpenStack, with which I tend to work a
lot. It’s relatively routine for me to use 2-3 different sets of
OpenStack clients in parallel, for testing: one with the Ubuntu
packages for the latest release, one with the packages for the release
prior, one with the packages that ship in the distro, and then
possibly yet another one with Python packages installed via `pip
install`.


Easy
## throw-away

Note: I often need to install something very briefly, and once I’m
done doing what I need to do, I’d like to get rid of it.

As far as Debian packages are concerned, that’s just a matter of `apt
remove --purge`. However, for anything else — something that has been
`pip` or `npm` installed, or something I’ve built locally, or
something where the vendor gave me a silly install script – I’d prefer
to actually be sure that whatever I wanted to throw away, _has_ been
thrown away.


## Non-FOSS
Skype, Zoom, non-free fonts

Note: Sometimes, I’m forced to install things that are non-free. For
example, the company I work for prefers Zoom for video
conferencing. Skype and non-free fonts are other examples of non-free
software that I sometimes need to use. I prefer to keep my root
filesystem clean of those.


Selective
## Device pass-through

Note: In relation to the aforementioned use of non-free software, I
find it inherently untrustworthy, so I’d like to be able to restrict
the access it has to my hardware.

In that situation, the software not knowing what my hardware even
looks like, because it’s presented a fake version of `/dev`, `/proc`,
and `/sys`, sounds like a beneficial thing to me. I’m certainly not
saying that this solves all of that software’s security problems, but
it just strikes me as a better idea to have this compartmentalization,
than not to have it.


## Why LXC?

(and not LXD, Docker, KVM, etc.?)

Note: Why am I doing all of this in LXC? Because it works for me, and
it suits me better than several alternatives:

* LXD, to the best of my knowledge, doesn’t expose the option of
  having the container management process run as non-`root`. Being
  able to run my containers as my own, unprivileged user account is
  one of my favorite features in LXC.

* Docker is really not built for system containers, and is geared
  toward the application container category. I could of course run
  _applications_ dockerized, but that is frequently not what my
  testing scenario is: rather, I want to see how an application
  behaves _within_ a specific environment.

* KVM is simply too heavyweight for what I need to do. I don’t need
  full hardware emulation, I want sharing parts of my host filesystem
  to be simple and easy, and none of my work is kernel related so I
  rarely have to test how something behaves under different
  kernels. And when I do, it’s usually easier to spin up a cloud
  instance in OpenStack that fire something up in local KVM.
