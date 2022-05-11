---
layout: post
title: Simple questions that helped us deliver 2 weeks before deadline
date: 2014-06-28 10:20:59.000000000 +05:00
tags:
- Software Development
- YAGNI
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
Although I am in the early years of my software development career, I am often exposed to discussions where we need to make a decision on the design of the application or the database. While working for a reputed security company based in the US, we were handed a task to design a windows console application which would be scheduled through the Windows Task Scheduler. Let’s say it was supposed to do task A, B and C through some .Net Web API.

The team got into the discussions on how to design this console application according to the meagre requirements sent to us through the user stories. So let me describe it in a few points:

Task A is completely independent of both B and C. Task A makes an API call (API A) and saves some value on the database based on the response.

Task B picks up the value stored in the database by Task A and makes a call to API B. If API B is successful, API C is called to finish up the process.

Let’s assume that if any API (A, B or C) fails, the app would do NOTHING.

So, again everyone sat down and we had long and hard brainstorming sessions on the application design. We assumed that the app would run every few hours making no less than 100s of API calls.

In the end, we designed an application which would run on multiple threads and would cater to a lot of load or at least that is what we had in mind. Moreover there were proposals of dynamically controlling the threads based on the load and God knows what not. But it turned out a perfect example of YAGNI (you aint gonna need it) i.e. speculative generalization.

While everything seems to be “oh, okay that’s a great scalable design”, one fine day, one of us questioned the product owner the simplest of all questions:

1)      How often will the app run, once scheduled?

2)      How many calls are we expecting to make in a single run?

The answers were quite shocking.

1)      Once a week

2)      2 – 10 API calls

From there on it took us no more than 2 days to accomplish what we had been trying to over complicate for more than a week. And the interesting fact, we finished 2 weeks before the estimated deadline.

The reason for sharing all this is pretty obvious. As a result of studying complicated frameworks, systems and other technologies, we tend to over complicate stuff for absolutely no reason, wasting our time and effort more often than not.

Also, we need to understand the misuse of the phrases like, ‘what if the requirements change in the future’, such phrases do not give us a license to over complicate a simple tasks. Let’s try to keep things as simple as they can.

Finally, guys, use your heads...
