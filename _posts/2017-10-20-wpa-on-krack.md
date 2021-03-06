---
layout: post
title: '0-Day: WPA2 On Krack!'
date: '2017-10-20 10:25:06 -0700'
comments: true
published: true
---
A couple of days ago, [Mathy Vanhoef](https://twitter.com/vanhoefm) of [imec-DistriNet](https://distrinet.cs.kuleuven.be/) has published his scientific [paper](https://papers.mathyvanhoef.com/ccs2017.pdf) which explained how WPA2's algorithm mechanism could be manipulated so an attacker could decrypt the WPA2-encrypted traffic between an Access Point and a client.<!--break-->
<br><br>

### **What is WPA2?**

WPA2 (Wi-Fi Protected Access II) is the protocol used in the encryption of wireless network communications (Wi-Fi). It has superseded the [WEP](https://en.wikipedia.org/wiki/Wired_Equivalent_Privacy) (Wired Equivalent Privacy) algorithm which was proven to be [extremely weak and easily broken](https://eprint.iacr.org/2007/120.pdf).
<br><br>

### **What is the Bug?**

Typically, after the authentication process followed by association between the client (your computer) and the AP (Wi-Fi router), a _4-way handshake_ takes place. This handshake results in the generation and exchange of [ different keys](https://security.stackexchange.com/a/149236) that are used in the encryption process of all further traffic between the client and the AP.

Since the radio waves are not 100% reliable, the protocol ensures that the key exchange is performed correctly by repeating the _Step 3_ the 4-way handshake several times. The 3rd step contains what is known by the `nonce`, _which is a random integer generated for a single purpose_, and in our case, it is used in the mathematical process of key(s) generation. Every time _Step 3_ is sent, the key is _reinstalled_.

**Note**
> This is not a flaw in the encryption method used in the protocol. This is simply a flaw in the implementation of the handshake process, which makes it a logic bug rather than a cryptographic one.

<br>
<img src="https://raw.githubusercontent.com/fusionx3/fusionx3.github.io/master/images/WPA2-Personal%2C%2B4-way%2Bhandshake.jpg" width="65%" height="65%" />
<br><br>

### **The Impact of the Bug**

An attacker could take advantage of the key reinstalling function to reinstall/reset the key by manipulating the value of the nonce. In case of providing a zero `nonce`, Linux and Android-based devices reset the entire key to **zero**, making the attack extremely easier. However, you might need to perform further cryptoattacks if you don't know the AP's password on other different Operating Systems/devices.
<br><br>

### **Affected Devices**

You can find a list of affected devices and Operation Systems with their respective vendors at the [CERT's official website](https://www.kb.cert.org/vuls/byvendor?searchview&Query=FIELD+Reference=228519&SearchOrder=4). The list is frequently updated.
<br><br>

### **Proof of Concept**

The author of the paper wanted to wait as long as possible before publishing the tools used in the [demonstration video](https://www.youtube.com/watch?v=Oh4WURZoR98) so vendors can have a reasonable window of time to patch their implementations of WPA2. However, one of the scripts got leaked. That gave the author no choice but to release the script which was partially leaked.
The script could be found at [Mathy's Github repository](https://github.com/vanhoefm/krackattacks-test-ap-ft).
<br><br>
 
### **Suggested Solution**

Interestingly, the suggested solution was a simple `boolean value`, which is set to true when the key is received and successfully installed, skipping any further attempts to reinstall the key. So when the attacker repeats the Step 3 packet, it will be ignored. An example pseudo-code:

```c
bool Step3 = false;
temp_key = recv_key();
if (Step3)
  //Skip reinstalling
else
  //Install the key
  key = temp_key;
```
<br>

### **Fixes**
As a regular user, best you can do is hope that your AP/Computer's vendor patches its own implementation of WPA2 as quick as possible. We know that [Microsoft has silently rolled a security patch last week](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2017-13080). However, if you're an advanced user, you can take advantage of the [released manual patches](https://github.com/kristate/krackinfo).
<br><br>
  
### **References**
[KrackAttack's Official Website](https://www.krackattacks.com)

[Proof of Concept video](https://www.youtube.com/watch?v=Oh4WURZoR98)

[Vulnerable Products and their vendors](https://www.kb.cert.org/vuls/byvendor?searchview&Query=FIELD+Reference=228519&SearchOrder=4)

[Patching Status for different products](https://github.com/kristate/krackinfo)
<br><br>

**Tags:** `#wifi` `#krack` `#0day` `#exploit` `#hack` `#wpa2`
<br><br>