---
title: "Photoprism and Running a Stateful Mariadb"
date: 2022-05-21T16:20:51+02:00
categories: ["homelab"]
tags: ["nomad", "mariadb", "photoprism"]
draft: true
---
A while ago my Synology NAS its adapter died and since then I've never really trusted that piece of hardware anymore. This also got me thinking about redundancy since with a two-bay NAS the propability of failure was kind of meh. So ordered some more harddisks, grab some older redundant motherboard and oversized case I had lying around and get a fresh install of Truenas on there. After quickly setting up zome ZFS pools, one for homelab data and one for my home NAS data, I went about and started a long-running rsync operation to get all the data off the Synology NAS. Pfew that went well and without any real hickups but now the next challenge came into view.

We used our Synology NAS also as photo organiser through DS Photo... As expected, my home user-base (i.e. my wife) was not amused. Luckily I found Photoprism through the homelab community and went my way to dive into this awesome piece of software. I decided to run this in my Nomad homelab cluster which is made up of a number of Lenovo ThinkCentre Tiny nodes and a tower server, first running this in my testing "datacenter". In its most simple form Photoprism leverages a SQLite database and can index all your photos and metadata into that database. The other option was to host a separate MariaDB database. Keep in mind that both these databases should store their data somewhere to be stateful, which seemed to be a perfect candidate for hosting in my Truenas server. After quickly setting up some storage in Truenas and exposing this through NFS I ran into the next challenge. How 
