---
author: "Robert LaRocca"
title: "How to Deploy an OpenVPN Server on OpenWrt 19.07"
date: 2021-06-20T03:15:00-04:00
categories:
  - server
  - security
  - networking
  - self-hosted
tags:
  - server
  - openvpn
  - openwrt
  - openssl
draft: false
---

📌 This should also work on the current `OpenWrt 21.02` release candidate.

This guide outlines how to configure an OpenWrt gateway as a OpenVPN server, which is perfect for providing secure remote access to your local network from anywhere.
<!--more-->

- Access shared files, printers and other services.
- Protect screen sharing and remote desktop connections.
- Prevent eavesdropping while using public Wi-Fi networks
- Watch Xfinity Stream (in-home) only channels from anywhere.
- Expand this list ad nauseam.

## What is OpenVPN?

> OpenVPN is a full-featured open source SSL/TLS virtual private network that accommodates a wide range of configurations, including remote access, site-to-site VPNs, Wi-Fi security, and [enterprise-scale solutions] with load balancing, failover, and fine-grained access-controls. Starting with the fundamental premise that complexity is the enemy of security. OpenVPN offers a cost-effective, lightweight alternative to other VPN technologies that is well-adapted for the [home, small business, educational] and enterprise markets.

## Packages

Before getting started it's a good idea to update the installed software on your OpenWrt device. Package updates generally resolve known issues, provide enhancements, and patch security vulnerabilities.

### Upgrade all installed packages

These commands will not upgrade the OpenWrt operating system or Linux kernel. Only packages installed using `luci` or `opkg` are upgraded.

```shell
opkg update
opkg install $(opkg list-upgradable | awk '{ printf "%s ",$1 }')
```

### Install OpenVPN and EasyRSA

Install _only_ the OpenVPN package build using OpenSSL. This version requires more storage then other OpenVPN packages, but offers better performance and supports more encryption algorithms.

```shell
opkg install openvpn-openssl openvpn-easy-rsa
```

## Certificate Authority

OpenVPN requires it's own Public Key Infrastructure (PKI) which consists of a Certificate Authority (CA) to sign server and client certificates. The certificates and private keys created during the next few steps are saved to `/etc/easy-rsa/pki`. See [setting up your own Certificate Authority (CA)](https://openvpn.net/community-resources/setting-up-your-own-certificate-authority-ca/) for more information.

### Create a certificate authority

Generate a OpenVPN certificate authority using the following commands.

```shell
easyrsa init-pki
easyrsa build-ca
```

### Create a Diffie Hellman

Generate the Diffie Hellman parameter using `easyrsa build-dh dh.pem`. This could take a long time, depending on your OpenWrt device CPU and available entropy.

### Create a TLS auth key

> The tls-auth directive adds an additional HMAC signature to all TLS handshake packets for integrity verification and provides an additional level of security above and beyond that provided by TLS alone.

Using the additional tls-auth server options can help protect against:

- DoS attacks or port flooding on the OpenVPN UDP port.
- Port scanning to determine which server UDP ports are in a listening state.
- Buffer overflow vulnerabilities in the TLS implementation.
- TLS handshake initiations from unauthorized machines are cutoff much earlier.

Generate a tls-auth key using `openvpn --genkey --secret ta.key`.

📢 This key should be copied to both the server and client devices.

### Create a server certificate

Generate a server certificate using the following `easyrsa` command. Substitute the hostname `openvpn.example.com` with your fully qualified domain name (FQDN).

```shell
easyrsa --subject-alt-name="DNS:openvpn.example.com" \
	build-server-full \
	openvpn.example.com \
	nopass
```

### Create client certificates

Generate a client certificate using `easyrsa build-client-full user1 nopass`. Substitute `user1` with your real username.

📢 This step should be repeated for every user in your environment.

🚫 Never share a single client certificate amongst multiple users. Redistributing new client certificates to every user, because access was revoked from a single bad actor is a security nightmare and allot of unnecessary work.

Now that your certificate authority, server certificate, server private key, diffie-hellman, and certificate revocation list is  generated, copy them to `/etc/openvpn/`.

## Configure

### Configure OpenVPN server

Edit the OpenVPN server configuration file `/etc/openvpn/server.conf`. You should modify these settings to match your specific environment.

```shell
# The OpenVPN configuration file

port 1194
proto udp
dev tun

ca ca.crt
cert openvpn.example.com.crt
key openvpn.example.com.key
crl-verify crl.pem
dh dh.pem

topology subnet
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /var/openvpn/persist.leases
status /var/openvpn/status.log

push "route 192.168.1.0 255.255.255.0"
push "route 10.8.0.0 255.255.255.0"
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 192.168.1.1"
push "dhcp-option DOMAIN lan"

client-to-client
max-clients 254

keepalive 10 120
persist-key
persist-tun

cipher AES-256-GCM
auth SHA384

tls-auth ta.key 0
tls-version-min 1.2
tls-cipher TLS-ECDHE-RSA-WITH-AES-256-GCM-SHA384

compress lz4-v2
push "compress lz4-v2"

user nobody
group nogroup

verb 3  # verbosity levels range from 0-low 3-default 9-devel
mute 10 # quite repeating logs after 20 messages

# Only used this option in UDP mode.
explicit-exit-notify 1
```

👀 The OpenVPN Community [reference manual](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/) explains every setting in detail.

### Create a network interface

Add the following settings to `/etc/config/network`. This will create a OpenWrt network interface used to configure the firewall.

```shell
config interface 'vpn'
	option ifname 'tun0'
	option proto 'none'
	option delegate '0'
```

### Create a firewall zone

Add the following settings to `/etc/config/firewall`. This will create a OpenWrt firewall zone.

```shell
config zone
	option name 'vpn'
	option input 'ACCEPT'
	option output 'ACCEPT'
	option forward 'ACCEPT'
	option masq '1'
	option mtu_fix '1'
	list network 'vpn'
```

### Configure firewall zone forwarding

Add the following settings to `/etc/config/firewall` to configure firewall zone forwarding. This allows OpenVPN traffic to access both `lan` and `wan` zones.

```shell
config forwarding
	option src 'lan'
	option dest 'vpn'

config forwarding
	option src 'vpn'
	option dest 'lan'

config forwarding
	option src 'vpn'
	option dest 'wan'
```

### Create an OpenVPN firewall rule

Add the following settings to `/etc/config/firewall` to configure a firewall rule to allow inbound traffic from the Internet. Client devices will be unable to communicate with the OpenVPN server without this firewall rule enabled.

```shell
config rule
	option name 'Allow-OpenVPN-Server'
	option src 'wan'
	option dest_port '1194'
	option proto 'udp'
	option target 'ACCEPT'
```

### Configure system startup script

This was once required and may no longer be required. If, OpenWrt refuses to start automatically after a reboot. Add the following to `/etc/rc.local`.

```shell
if [ -x "/usr/sbin/openvpn" ]; then
	if [ ! -d "/tmp/openvpn" ]; then
		mkdir -p /tmp/openvpn >& /dev/null
	fi
fi
```

## Enable OpenVPN server

```shell
/etc/init.d/openvpn enable
/etc/init.d/openvpn start
```

In some cases OpenVPN won't start until the OpenWrt device is rebooted. I suspect the required `openssl` kernel modules are not fully enabled.

🎗 I should really file a bug report with the OpenWrt package maintainer.

You should now have a working OpenVPN remote access server deployed on OpenWrt. If, you have any questions or would like help troubleshooting issues, contact me on [Matrix](https://matrix.to/#/@robert:laroccx.com)!
