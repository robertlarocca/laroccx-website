---
title: "How to Create a macOS Big Sur ISO Disk Image"
date: 2021-09-21T01:00:29-04:00
categories:
  - virtualization
tags:
  - macos
  - big sur
  - vmware
draft: false
---

This post is more or less a reminder for myself how to create a macOS disk image. The created ISO image can be used with VM ware Fusion, VM ware Workstations, or used to make a bootable USB installer.

## Download macOS Big Sur

> macOS Big Sur elevates the most advanced desktop operating system in the world to a new level of power and beauty. Experience Mac to the fullest with a refined new design. Enjoy the biggest Safari update ever. Discover new features for Maps and Messages. And get even more transparency around your privacy.

First download [macOS Big Sur](https://apps.apple.com/us/app/macos-big-sur/id1526878132) from the Mac App Store. This is the only _legal_ method to obtain a copy of the software. It's a free download if you own a Mac, otherwise just barrow a friend or family-members Mac for a couple hours.

## Create an Empty Disk Image

```shell
sudo hdiutil create \
  -o /tmp/macOS-Big-Sur \
  -size 16384 m \
  -volname macOS-Big-Sur \
  -layout SPUD \
  -fs HFS+J
```

## Mount the Empty Disk Image

```shell
sudo hdiutil attach /tmp/macOS-Big-Sur.dmg \
  -moutpoint /Volumes/macOS-Big-Sur \
  -noverify
```

## Copy Install Media to Mounted Disk Image

```shell
sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia \
  --volume /Volumes/macOS-Big-Sur \
  --nointeraction
```

## Covert DMG to ISO and Rename Disk Image

```shell
hdiutil eject -force /Volumes/Install\ macOS\ Big\ Sur
```

```shell
hdiutil convert /tmp/macOS-Big-Sur.dmg \
  -o ~/Downloads/macOS-Big-Sur
  -format Unto
```

```shell
mv -v ~/Downloads/macOS-Big-Sur.cdr ~/Downloads/macOS-Big-Sur.iso
```

## Cleanup Temporary Files

```shell
sudo rm -fv /tmp/macOS-Big-Sur.dmg
```
