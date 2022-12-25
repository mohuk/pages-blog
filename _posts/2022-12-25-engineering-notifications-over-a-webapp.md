---
layout: post
title: Engineering Notifications over a Web app
date: 2022-12-25 04:59:17.000000000 +05:00
tags:
- Software Engineering
- AWS
- NodeJS
status: publish
type: post
published: true
author:
  login: mohuk
  email: mohammad.umair.k@gmail.com
  display_name: mohuk
  first_name: 'Muhammad Umair'
  last_name: 'Khan'
---
Up until the world exploded into mobile apps, communication between the schools and children’s parents was simple. Parent's expected a message in the children's diary. Fast forward today, diaries are no longer in fashion. Every school has an app to "keep you updated". The problem with apps is that it should have an amazing notification system. It turns out, my son’s school did have an app but did not have a functional notification system. My son kept missing school events and that’s when I decided to tinker around to see if I could make a notification system on top of the schools apps.

#### Bird's eye view
The school had both mobile and web apps. Having the right credentials, information was accessible over the internet. A quick skim through the “Network Tab” gave me the HTTP call to fetch the required data. The fetch request returned an HTML document which had everything required. All I wanted now was to parse the HTML, extract what I wanted from the DOM and send a notification if there is something "new". 

I decided that notification should be an SMS to avoid complicating the system. The content of the SMS can be kept simple, e.g. “New notification on ABC School Portal. Please login to https://parent.abc.edu”.

![High level flow](/assets/img/2022-12-25-img-01.png)

Running the above cycle few times a day means we’d have to be super lazy to miss them. Next step, Code!

#### Engineering Details:
The entire process just need to run once every few hours and was completely independent of state. Such workflows are tailor made for serverless apps. A Lambda function, scheduled to run through CloudWatch Events and sending an SMS through SNS should suffice.

![AWS flow](/assets/img/2022-12-25-img-02.png)

*I will not be diving into the how to’s as there is a lot of content on the internet on setting up Lambda functions, CloudWatch alarms and sending SMS via SNS.*

#### Keeping it under free-tier:
While I want things to be easy for me, I don’t want to spend money for a feature the school engineering team should have provided out of the box. Free tier limits for AWS Lambda, SNS and CloudWatch are very gracious for a use case like this. The only challenge for me was the SMS which is never free. AWS SNS sandbox provides upto 10 numbers on which SMS can be sent as part of testing which is more than enough for me. I added a couple of phone numbers and started receiving SMS-es at not cost.

#### Final outlook:
CloudWatch events make the Lambda run every few hours to check for updates and notifies me and my wife via an SMS. This is definately helping us stay up to date on the school conversations. 

The code can be found [here](https://github.com/mohuk/fps-connect-notifications). One fine day, I will move this to AWS CDK.