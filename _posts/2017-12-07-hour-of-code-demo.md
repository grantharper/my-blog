---
layout: post
title:  "Hour of Code"
date:   2017-12-07
categories: software
---

Today for hour of code, I will demo a simple Alexa app (skill) using the Amazon Echo. While the app seems really simple from the user's point of view, it is really cool to think about everything that is going on behind the scenes.


![](/img/alexa-service-diagram.png)

As a user, you simply ask the service for something and it responds back to you.

In reality, the request you make is processed by a lot of different computers before the echo responds back to you.
1. You talk to the echo
2. The echo processes your voice and sends your commands over the internet to the Alexa service
3. The Alexa Skill interprets your commands and calls the Lambda "serverless" computer
4. This computer calls the file system where the responses are located and sends the response back to the Alexa service
5. The Alexa service responds to your echo
6. The echo responds back to you

Pretty cool!