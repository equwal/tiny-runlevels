Using OpenRC runlevels to have lots of instances of simple (network)
programs like quark for multiple http serving locally or ii (or sic)
for IRC server connections.

To keep programs simple, they should not do the same process for
unrelated uses. It is much more complicated to configure and manage
multiple webservers in the same process (like how nginx does it) than
just one. Init systems are designed to manage processes, and should be
used instead. Do one thing etc. etc.

This is how to use the OpenRC runlevels for quark and ii runlevels
as examples.

## The Tree (on my machine)

```sh
/etc/runlevels
├── boot
│   └── ...
├── default
│   └── ...
├── irc-base
│   ├── oftc -> /etc/init.d/oftc
│   ├── liberachat -> /etc/init.d/liberachat
│   └── ...
├── net-base
│   ├── default -> ../default
│   └── quark -> ../quark
├── net-offline
│   ├── default -> ../default
│   └── net-base -> ../net-base
├── net-dns
│   ├── dnscrypt-proxy -> /etc/init.d/dnscrypt-proxy
│   └── unbound -> /etc/init.d/unbound
├── net-wifi
│   ├── dhcpcd -> /etc/init.d/dhcpcd
│   ├── net-base -> /etc/runlevels/net-base
│   ├── net-dns -> /etc/runlevels/net-dns
│   └── wpa_supplicant -> /etc/init.d/wpa_supplicant
├── net-wifi-irc
│   ├── irc-base -> ../irc-base
│   └── net-wifi -> ../net-wifi
├── quark
│   ├── quark-docs -> /etc/init.d/quark-docs
│   ├── quark-equwal -> /etc/init.d/quark-equwal
│   └── ...
├── shutdown
│   └── ...
└── sysinit
    └── ...
```
as you can see, IRC is connected after the real internet on wifi
(net-wifi), but quark runs on any network runlevel using net-base.

## How To:

OpenRC has its own commands for making the runlevels, but I find them
confusing. Let's just edit the files.

```sh
# make runlevel
mkdir /etc/runlevels/quark
# add services (as many as needed)
ln -s /etc/int.d/quark-whatever /etc/runlevels/quark
# add runlevel to parent
ln -s /etc/runlevels/default /etc/runlevels/quark
# Reload since openrc is in default
openrc default
```

That works, but what about for networked IRC on a custom networking runlevel? Just add it to the networking runlevel.
```sh
# (first, make a network runlevel that contains default! Without default the system will crash, obviously.)
mkdir /etc/runlevels/net-wifi
ln -s /etc/runlevels/net-wifi /etc/runlevels/default
ln -s /etc/init.d/wpa_supplicant /etc/runlevels/net-wifi
# etc.
# Now add the irc-base to the network runlevel
mkdir /etc/runlevels/irc-base
ln -s /etc/init.d/liberachat irc-base
ln -s /etc/init.d/oftc irc-base
ln -s /etc/runlevels/irc-base /etc/runlevels/net-wifi
openrc net-wifi
```

## Tool: restart-runlevel
Openrc wants switch the whole stack `openrc runlevel` so I made a little
tool to just restart a specific runlevel. Nice for debugging.

```sh
#!/bin/bash

for link in /etc/runlevels/${1}/*
do [[ -L "$link" && "$(readlink "$link")" == /etc/init.d/* ]] && \
    rc-service "$(basename "$link")" restart
done
```
