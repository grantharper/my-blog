---
layout: post
title:  "Audio Chef Part 1"
date:   2018-01-07
categories: software
---

I like cooking (and eating), but it can be a messy business. What better application for the Amazon Echo than a kitchen assistant? Through voice commands, Alexa can guide users through recipes alleviating the problem of consulting cookbooks or unlocking smart phones. That's what I set out to do with [The Audio Chef][theaudiochef]. Below is a diagram that illustrates the architecutre, focused solely on the user experience with the Alexa Skill and the various supporting AWS components.

![](/img/audio-chef-diagram-1.png)

Main Points:
* Session state is stored in the DynamoDB table AudioChefAlexaSession
* Recipe data is stored in the DynamoDB table AudioChefRecipeUrl
* Sample recipe lookup data is stored in the S3 bucket audiochefcode for ease of updating without having to perform code changes
* The request/response from/to the Alexa Skill service is handled by the Lambda function AudioChefAlexaCustom
* The skill configuration is managed through the Amazon Developer Console Alexa Skill section

[theaudiochef]: https://theaudiochef.com