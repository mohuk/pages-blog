---
layout: post
title: Faster Page Loads - Optimizing Time to First Byte (TTFB)
date: 2024-02-08 11:59:17.000000000 +05:00
tags:
- Software Engineering
- AWS
- Angular
- Performance
- Cloud
- SPA
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
There are plenty of case-studies concluding faster page loads for an online business result in a much better user experience. A better and more polished user experience which eventually translates into more business. Banking on this hypothesis at Rewaa, we picked a metric which reside on the bottom on the content delivery chain, Time to First Byte (TTFB). Let's look at the text book definition of TTFB:

*TTFB measures the duration from the user or client making an HTTP request to the first byte of the page being received by the client's browser.* -- Wikipedia - [Time to first byte](https://en.wikipedia.org/wiki/Time_to_first_byte#) 

## Why TTFB?
One might argue, why optimize TTFB ahead of user centric metrics like LCP and FCP? To answer this, we have to understand the significance of optimizing for TTFB when serving Single Page Applications (SPAs).

A SPA first has to deliver markup to the browser. The markup then requests for styles and javascript which execute to deliver meaningful content to the user. The sooner the content is delivered to the browser, the quicker user centric content can be rendered. This is why TTFB is critical when serving SPAs.

## What is a good TTFB score?
This is simple, the quicker, the better. However most sources on the internet say anything below 800ms is good. In short, it should take less than 0.8s for the content to reach the browser so it can start doing what it needs to show meaningful content to the user.

![TTFB Score](/assets/img/2024-02-08-img-01.png)

With the case established for optimizing TTFB, the first thing we needed was to measure it across our users.

## Measuring TTFB

*"You cannot improve what you don't measure"*.

To measure TTFB, a quick search on the internet pointed us to the following places:
- Network Tab of Chrome (and other browsers)
- Performance analysis through Lighthouse

![Chrome - TTFB](/assets/img/2024-02-08-img-02.png)

I noticed that TTFB was not too bad on my connection, so does that mean the problem doesn't exist? This is where we have to understand the traffic routing from the server to your machine varies on many factors. A faster load time for me, does not necessarily mean a faster load time for other users. To get a more realistic measure of TTFB across our user base, I turned to [Real User Monitoring (RUM)](https://docs.datadoghq.com/real_user_monitoring/) on Datadog and setup a monitor.

I setup a visualization by selecting "Query Value". Under "RUM" I configured to aggregate TTFB by the 75th percentile across all views. Rounded it up to 2 decimal places and Datadog gave me a nice and simple graph to measure my work against. I titled it "Average TTFB for all views (75pc)".

![Dashboard - TTFB](/assets/img/2024-02-08-img-03.png)

The number above prove we were certainly not in the "Poor" (>1.8s) but we had a lot of work to do to be in the good.

## Exploring Content Delivery
As a starting point, I simply  It turns out that to facilitate SPA routing, any route within the application was returned with an `index.html` file with a HTTP 404. I will not quote posts, but there is actual content online which advocates to serve `index.html` on a 404, trust me and never do that.

![Static Content Delivery 1.0](/assets/img/2024-02-08-img-04.png)

Chalking this on a whiteboard, I realized there are several problems with our setup:

- **All initial page loads were 404s with `index.html`**: To enable SPA routing, application routes resulted in Http 404 with `index.html`. I will not quote posts, but there is actual content online which recommends serving `index.html` on a 404 to enable SPA routing because it works and is easy to configure (few clicks). However, it has side effects which maybe unknown. One such example is, running "Lighthouse" page load performance refuses to compute results because `index.html` has a 404 - Not Found status.

- **No compression**: Lighthouse also indicated no compression is applied on content being served resulting in high volumes of data transfer.

- **All requests were resulting in a cache miss**: Each cache miss resulted in content being delivered directly from the S3 bucket in the region. No advantage was being taken from the intermediate caches. The increased hops resulted increased latency as all the content was being served directly from the deployment region (Mumbai: ap-south-1) to the entire world.

![Static Content Delivery 1.0](/assets/img/2024-02-08-img-05.png)

## Solutions

### Routing fix
The first step was to get rid of the 404s and fix the routing. All we need is a place to rewrite a route to `index.html` if it is not a file i.e. have an `.` followed by an extension. A few searches resulted in something known as `Lambda@Edge` which can execute code before the request reaches CloudFront. 

![Lambda@Edge - AWS Docs](/assets/img/2024-02-08-img-06.png)

The ability to intercept a request before it reaches CloudFront Edge AKA `Viewer Request` is all I needed. In the following few lines, I was able to rewrite the incoming requests to `index.html`.

```javascript
export const handler = async (event, context, callback) => {
  const { request } = event.Records[0].cf;

  const re = /(?:\.([^.]+))?$/;

  if (!re.exec(request.uri)[1]) {
    const newuri = request.uri.replace(/.*/, '/index.html');
    request.uri = newuri;
  }
  callback(null, request);
};
```

Steps:
- Create a Lambda Function with the above code in the region `us-east-1`
- Deploy the code at `Lambda@Edge`
- Copy the ARN of the deployed `Lambda@Edge`
- On CloudFront, Edit `Behaviors`. In the `Function associations` section, attach `Lambda@Edge` to the `Viewer Request`

With `Lambda@Edge` setup, just remove the `Custom error response` from `Error pages`. The resulting flow is illustrated below.

![SPA Content Delivery 2.0-01](/assets/img/2024-02-08-img-07.png)

### Caching fix
The next target was to minimize the `cache miss` illustrated in the diagrams above. To cache content, the CDN looks for the `Cache-Control` headers. A quick search yielded that our entire bucket of static content had `Cache-Control` set to `no-cache, no-store, must-revalidate`. Let's break this down:
- `no-cache`: This does not mean "don't cache". Cache will always revalidate content from the source before serving.
- `no-store`: No content would be store in **any** cache.
- `must-revalidate`: Response will be reused while fresh. Response will be validated once stale.

The `no-store` not allowing anything to be stored made the remaining 2 directives practically useless. Worth mentioning that `must-revalidate` always should be used with `max-age` to help the cache identify fresh content. In short, the configuration was completely broken. 

What we wanted was the cache to reuse the responses until a reasonable amount of time has passed or there is new content available. We concluded a sweet spot would be `7 days`, and simply set our `Cache-Control` header to `must-revalidate max-age=604800`. On each new release, we programmatically invalidated all CloudFront caches to ensure the new release always results in fresh content for our users. Here it what it looked like:

![SPA Content Delivery 2.0](/assets/img/2024-02-08-img-08.png)

### Enabling compression
Simply enabling GZip compression reduced the size and transmitted data from 70% to 90% of the overall size. Enabling GZip compression is simple and requires few clicks.

Steps:
- On CloudFront, Edit `Behaviors`. 
- In the `Cache key and origin requests` section, select `Cache policy and origin request policy (recommended)` and set `Cache Policy` to `CachingOptimized`.

## Interesting discovery - Lambda@Edge vs CloudFront Functions
While monitoring the logs I noticed that all content is being served from Frankfurt. Almost all of our customer base is in Saudi Arabia and not serving content from Mumbai (our deployment region), was definitely an improvement. I wondered if there were still a few more millisecond I could reduce. Reading online I discovered how AWS lays out its caches and the stark differences between an `Edge Cache` and a `Regional Cache`.

**tl;dr** `Edge Location` serves content with reduced latency because it is nearer to the users as compared to the `Regional Edge Cache`.

The reason why all our content was being served from Frankfurt is because `Lambda@Edge` executes only on the Regional Edge Cache. Since our routing logic was simple, moving it to `CloudFront Functions` (Event source: Viewer Request) was the better choice. A clear reduction of approx. 100ms was noticed by moving from `Lambda@Edge` to `CloudFront Functions` You can find all the difference between Lambda@Edge and `CloudFront Functions` at [this link](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-choosing.html).

![AWS CloudFront CDN](/assets/img/2024-02-08-img-09.png)

## Results
I recorded the TTFB numbers from 4PM to 9AM (the next day) for 3 consecutive weekdays to test the hypothesis. Here are the results:

|**Function**|**Avg. TTFB (75pc)**|
|No function| 987.45ms|
|Lambda@Edge|490.34ms|
|CloudFront Functions|401.46ms|

And here are the numbers for the amount of data transferred for 2 consecutive months:

|**Dates**|**Data Transferred**|
|25th Oct - 24th Nov| 9.49 GB|
|25th Nov - 25th Dec|2.02 GB|

Overall, the results came out to be just fantastic. The 75pc of **TTFB was reduced by 50%** and the **Total Data Transfer was reduced by 75%** on average. Target of reducing TTFB to less than 500ms was successfully achieved.