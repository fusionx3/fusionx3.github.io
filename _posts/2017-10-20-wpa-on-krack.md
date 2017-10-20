---
layout: post
title: '0-Day: WPA2 On Krack!'
date: '2017-10-20 10:25:06 -0700'
comments: true
published: true
---
A couple of days ago, [Mathy Vanhoef](https://twitter.com/vanhoefm) of [imec-DistriNet](https://distrinet.cs.kuleuven.be/) has published his scientific [paper](https://papers.mathyvanhoef.com/ccs2017.pdf) which explained how WPA2's algorithm mechanism could be manipulated so an attacker could decrypt the WPA2-encrypted traffic between an Access Point and a client.<!--break-->
<p>

## **What is WPA2?**

WPA2 (Wi-Fi Protected Access II) is the protocol used in the encryption of wireless network communications (Wi-Fi). It superseded the [WEP](https://en.wikipedia.org/wiki/Wired_Equivalent_Privacy) (Wired Equivalent Privacy) algorithm which proved to be [extremely weak and easily broken](https://eprint.iacr.org/2007/120.pdf).
<p>

## **What is the Bug?**

Typically, after the

## **The Impact of the Bug**

An attacker co
<p>
## **Affected Devices**

You can find a list of affected vendors at the [CERT's official website](https://www.kb.cert.org/vuls/byvendor?searchview&Query=FIELD+Reference=228519&SearchOrder=4). The list is frequently updated.
<p>
## **Proof of Concept**

The author of the paper wanted to wait as long as possible before publishing the tools used in the [demonstration video](https://www.youtube.com/watch?v=Oh4WURZoR98) so vendors can have a reasonable window of time to patch their implementations of WPA2. However, one of the scripts which was used in the video got leaked. That gave the author no choice but to release the script which was partially leaked.
The script could be found at [Mathy's Github repository](https://github.com/vanhoefm/krackattacks-test-ap-ft).
<p>
  
## **Fixes**
As a regular user, best you can do is hope that your AP/Computer's vendor patches its own implementation of WPA2 as quick as possible. However, if you're an advanced user, you can take advantage of the manual patches which were released 
<p>
  
## **References**
www.
