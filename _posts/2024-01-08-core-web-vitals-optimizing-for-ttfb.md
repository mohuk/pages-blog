---
layout: post
title: Faster Page Loads - Optimizing for Time to First Byte (TTFB)
date: 2024-02-08 11:59:17.000000000 +05:00
tags:
- Software Engineering
- AWS
- NodeJS
status: publish
type: post
published: false
author:
  login: mohuk
  email: mohammad.umair.k@gmail.com
  display_name: mohuk
  first_name: 'Muhammad Umair'
  last_name: 'Khan'
---
There are plenty of case-studies concluding faster page loads for an online business result in a much better user experience. A better and more polished user experience which eventually translates into more business. Banking on this hypothesis at Rewaa, we picked a metric which reside on the bottom on the content delivery chain, Time to First Byte (TTFB). Let's look at the text book definition of TTFB:

*TTFB measures the duration from the user or client making an HTTP request to the first byte of the page being received by the client's browser.* -- Wikipedia - [Time to first byte](https://en.wikipedia.org/wiki/Time_to_first_byte#) 

#### Why TTFB?
One might argue, why optimize TTFB ahead of user centric metrics like LCP and FCP? To answer this, we have to understand the significance of optimizing for TTFB when serving Single Page Applications (SPAs).

A SPA first has to deliver markup to the browser. The markup then requests for styles and javascript which execute to deliver meaningful content to the user. The sooner the content is delivered to the browser, the quicker user centric content can be rendered. This is why TTFB is critical when serving SPAs.

#### What is a good TTFB score?
This is simple, the quicker, the better. However most sources on the internet say anything below 800ms is good. In short, it should take less than 0.8s for the content to reach the browser so it can start doing what it needs to show meaningful content to the user.

![TTFB Score](/assets/img/2024-02-08-img-01.png)

With the case established for optimizing TTFB, the first thing we needed was to measure it across our users.

#### Measuring TTFB

*"You cannot improve what you don't measure"*.

To measure TTFB, a quick search on the internet pointed us to the following places:
- Network Tab of Chrome (and other browsers)
- Performance analysis through Lighthouse

![Chrome - TTFB](/assets/img/2024-02-08-img-02.png)

I noticed that TTFB was not too bad on my connection, so does that mean the problem doesn't exist? This is where we have to understand the traffic routing from the server to your machine varies on many factors. A faster load time for me, does not necessarily mean a faster load time for other users. To get a more realistic measure of TTFB across our user base, I turned to [Real User Monitoring (RUM)](https://docs.datadoghq.com/real_user_monitoring/) on Datadog and setup a monitor.

I setup a visualization by selecting "Query Value". Under "RUM" I configured to aggregate TTFB by the 75th percentile across all views. Rounded it up to 2 decimal places and Datadog gave me a nice and simple graph to measure my work against. I titled it "Average TTFB for all views (75pc)".

![Dashboard - TTFB](/assets/img/2024-02-08-img-03.png)

The number above prove we were certainly not in the "Poor" (>1.8s) but we had a lot of work to do to be in the good.
