## A simple example

`ruby-sass`


The presentation contains a live screen demo at this stage.

Note: This is a container that simply includes a Ruby runtime, and the
`ruby-sass` package in Ubuntu (to serve the same purpose, I could also
have included the `sass` gem in the container.)

* `lxc-info -n xenial-sass`
* `lxc-attach -n xenial-sass`
* `lxc-attach -n xenial-sass -- sudo -Hu florian sass --watch -t expanded -E utf-8 css:css`


## Sharing `$HOME`

Note: So far, so simple. But you may have noticed that in order to be
able to do what I just did, a few conditions had to be met (besides
the `sass` binary being available, of course:

* My username, `florian`, must map to a user available in the
  container.
* My `$HOME` directory must be available inside the container.
* That `$HOME` directory must be _writable_ to that user inside the
  container.


The presentation contains a live screen demo at this stage.

Note: 
* `vi ~/.local/share/lxc/xenial-sass/config`

We do those things by adding subuid/subgid mappings, and a bind
mount.

The required `lxc.config` entries for the uid/gid mappings are
unfortunately a bit convoluted, but in essence what this means is:

* give the user a uid/gid namespace that is shifted by
  100,000 (user 0 in the container becomes 100000 in the host, etc.)
* however, _exclude_ uid/gid 1000 (the user and group `florian`) from
  that shift, so it alone maps to uid/gid 1000 in the guest.

`lxc.id_map = u 0 100000 1000` means “for 1000 uids starting with 0,
shift them by 1000000,” that is, map uids 0-999 to 100000-100999.


## X applications

Note: Now let’s turn it up a small notch. We’d like to run an X
application in a container. That’s actually not that difficult with X,
which is what I use here on Ubuntu 16.04, and this approach works for
Wayland as well, provided Wayland runs the X compatibility layer
(`XWayland`). As for native Wayland, I’m sure there’s a way to make
this work too, I just haven’t looked into it yet.

Now of course we could do this by running `ssh -X` into the container,
but that requires that we always have a running SSH server, and I
prefer a more direct method. 


The presentation contains a live screen demo at this stage.

Note:
* `vi ~/.local/share/lxc/xenial-firefox-java/config`
* `lxc-attach -n xenial-firefox-java`
* `firefox`


## Sound

Note: Sometimes, I do want sound in my container as well. Now there
are a couple of ways to do that, with Pulseaudio.

I could have Pulseaudio put its listening socket in a non-standard
location, and then share that with the container via a bind-mount
again (the default in `/run/user` doesn’t work in a container because
the container `systemd` mounts over that).

However, I find it much easier to just start a Pulseaudio TCP socket
on my host, and then have applications in the container connect to
that, instead.


The presentation contains a live screen demo at this stage.

Note:
* `pactl load-module module-native-protocol-tcp`
* `lxc-attach -n xenial-firefox-java`
* `export PULSE_SERVER=10.0.3.1`
* `firefox https://hpr.dogphilosophy.net/test/`


## USB device pass-through

Note: By default, containers don’t see USB devices plugged into the
host, because they get their own fake `/sys` and `/dev`
tree. Sometimes however, I do want to make USB devices accessible to
my container. One such example is my webcam which I want to be able to
use in conjunction with Zoom meetings. Here, what LXC helps me do is
pass through _just_ the devices that I want to, and no others.


The presentation contains a live screen demo at this stage.

Note:
* `vi ~/.local/share/lxc/xenial-zoom/config`
* `lxc-attach -n xenial-zoom`
* `ls /dev/video*`
* `export PULSE_SERVER=10.0.3.1`
* `zoom`


## Cloning,
## snapshots,
and
## subvolumes

Note:
Clearly, with all these containers that all share a very common
baseline configuration (in my case, just 1-3 Ubuntu releases), it
would be silly and time-consuming to always create containers from
their templates. Luckily, we can do things much more easily (and much
faster!) by using LXC clones in combination with btrfs snapshots.

When we use `-B btrfs`, then `lxc-create` creates a new btrfs
_subvolume_ in either `/var/lib/lxc` or `~/.local/share/lxc` to hold
the `rootfs` of the container. If we subsequently use `lxc-copy` with
the `-s` option, then the lxc userland creates a new _snapshot_ off
that subvolume. The latter is particularly useful, because it can
create a container almost instantaneously (that is, in under a
second), which makes container spin-up and tear-down a breeze.


The presentation contains a live screen demo at this stage.

Note:
* `time lxc-create -B btrfs -n xenial-test -t ubuntu-cloud -- --release xenial --tarball https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-root.tar.xz`
* `time lxc-copy -B btrfs -n xenial-test -N xenial-clone`
* `sudo btrfs subvolume show ~/.local/share/lxc/xenial-test/rootfs`


## Automation
Ansible ftw!


The presentation contains a live screen demo at this stage.

Note:
* `cd /tmp`
* `git clone https://github.com/fghaas/ansible-laptop`
* `git clone https://github.com/fghaas/ansible-laptop-config-example`
* `cd ansible-laptop-config-example`
* `vi host_vars/localhost`
* `cd ../ansible-laptop`
* `ansible-playbook -i '../ansible-laptop-config-example/inventory' -t lxc local.yml`
* `cd -`
* `ln -s ~/git/ansible/contrib/inventory/lxc_inventory.py`
* `cd -`
* `ansible-playbook -i '../ansible-laptop-config-example/lxc_inventory.py' -t lxc local.yml`
