---
layout: post
title: From Docker API To Root Access in 60 Seconds
date: '2017-10-09 16:25:06 -0700'
comments: true
published: true
---
I was tasked with performing a penetration testing on a server which hosted multiple websites and services. Most of these services ran on _Docker containers_, and in this article, I'm going to show how could an overlooked misconfiguration lead to a complete takeover of the **host** system.

**Disclaimer:** This article was written for educational purposes, and the author is not responsible for any misuse of any of the tools or techniques demonstrated in the article.

## **What is Docker?**


## **Information Gathering**
Initial reconnaissance showed an open port that was apparently serving some sort of an API that supports JSON objects. Default response for any unknown requests was a JSON object like this:

	{"message":"page not found"}

By appending `/info` to the URL of the service to be like this: `http://10.0.5.10:4000/info`, we were able to view further information about the Docker service behind the API:

![Information about the Docker Daemon]({{ "/images/docker_1.png" | absolute_url }})


So, the system administrator hasn't restricted access to the Docker API (A big mistake), and apparently, since the Docker daemon runs as a privileged user, I doubt that the system administration bothered

To enumerate the version of the API, we simply appended `v1.30/info` to the URL and the API explicitly told us its version:

`{"message":"client is newer than server (client API version: 1.30, server API version: 1.24)"}`

## **Gaining access to a container**
After some quick reading through the [Docker API Documentation](https://docs.docker.com/engine/api/v1.24/), creating a custom image and starting it as a container is simply a matter of a few POST requests.

Since Docker allows us to create an image from a Docker file, it shouldn't be too different if performed through the API, right? 






