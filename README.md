> Please note that the instructions in this tutorial have been written
> for Ubuntu 16.04 Xenial Xerus. Your mileage may vary on other
> platforms. In particular, if you’re on Fedora, you’ll be unable to run
> unprivileged LXC containers (as of February 2018), as explained in
> [this issue](https://github.com/lxc/lxc/issues/1998).

# Command cheat sheet

This is for linux.conf.au 2018 attendees. It is **not** a complete
list of commands you'll be using in the tutorial, just some of the
more complex ones that you might want to copy and paste.

## Prepare `lxc-usernet`

```bash
sudo tee -a /etc/lxc/lxc-usernet <<EOF
$USER veth lxcbr0 100
EOF
```

## Install a container named `xenial-sass`

```bash
lxc-create -n xenial-sass -t ubuntu-cloud -- \
  --release=xenial \
  --tarball=https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-root.tar.xz

lxc-start -n xenial-sass
lxc-attach -n xenial-sass

apt update
apt install ruby-sass ruby-listen
```

## Enable user subuid and subgid mapping

```bash
for f in /etc/sub{u,g}id; do
  sudo tee -a $f <<EOF
$USER:100000:65535
EOF
done
```

## Enable `/home` sharing

```
vi ~/.local/share/lxc/xenial-sass/config

lxc.mount.entry = /home home none rbind

# Ubuntu 16.04 / 17.04
lxc.id_map = u 0 100000 1000
lxc.id_map = g 0 100000 1000
lxc.id_map = u 1000 1000 1
lxc.id_map = g 1000 1000 1
lxc.id_map = u 1001 101001 64535
lxc.id_map = g 1001 101001 64535

# Ubuntu 17.10
lxc.idmap = u 0 100000 1000
lxc.idmap = g 0 100000 1000
lxc.idmap = u 1000 1000 1
lxc.idmap = g 1000 1000 1
lxc.idmap = u 1001 101001 64535
lxc.idmap = g 1001 101001 64535

lxc-stop -n xenial-sass
lxc-start -n xenial-sass
```

## Get `ansible-laptop-config`

```bash
git clone https://github.com/fghaas/ansible-laptop
git clone https://github.com/fghaas/ansible-laptop-config-example
ansible-laptop-config

wget -O ansible-laptop-config/lxc_inventory.py \
  https://github.com/ansible/ansible/blob/devel/contrib/inventory/lxc_inventory.py
```

## Run ansible against localhost

```bash
cd ansible-laptop
ansible-playbook -i ../ansible-laptop-inventory/inventory local.yml
```

## Run ansible against containers

```bash
ansible -i ../ansible-laptop-inventory/lxc_inventory.py \
        -m raw \
        -a "apt-get --yes install python"

ansible-playbook -i ../ansible-laptop-inventory/lxc_inventory.py \
                 -t lxc local.yml
```
