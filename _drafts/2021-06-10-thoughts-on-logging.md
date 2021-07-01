---
layout: stories
title: Thoughts on Logging
description: Simple thoughts and guidelines on logging
author: Eldon
hero:
href: 'https://source.unsplash.com/EaB4Ml7C7fE/600x250'
alt: Developer working
---


![Developer Working](https://source.unsplash.com/EaB4Ml7C7fE/800x450)


I joined the LexisNexis global engineering team as a Technical Fellow late last year. So a large portion of my energy is providing guidance and coaching to help improve the state of software development internally.is a big part of what we do. So even when I am not coding per se, I can't begin to count the number of times in this job or previous roles when another developer has asked for my help in troubleshooting some error, only for us to immediately hit a brick wall because they had anaemic logging.

These days I don't get as much dedicated developer time as I did earlier in my career. However, when I do start a new application - one of the earliest decisions I make is what I'm going to do about logging. It's such a critical component for both how I'm going to develop the application and how I'm going to deal with it running in production that the decisions around this process need to be made within the first iteration.

To help kick off our new engineering blog, I thought I would share a few thoughts about an area that I still see many developers struggle with: the importance and purpose of logs. As an industry, Software engineering tends to have a serious problem with conflating ideas together.  , and logging is no exception.

Every application is going to have different requirements and constraints for how and what they log. An API running on a server will have vastly different logging requirements than an application running on an end-user's machine, and some industries and domains will even have specific legal requirements mandated on what they have to log.

Therefore this blog can't be a list of rules or commandments. Instead, I hope to add value by sharing a few stories of painful experiences that I've faced or witnessed over the years, and extract some of the guiding principles that may be useful for others to design their own logging solutions.


### Logging is not about metrics, tracing, etc.
` not happy with this heading - needs rewording`

The first and foremost principle that I use is that logs should be written for the people who need to support the application in production (both developers and operations) and should be separate from things like metrics, observability and tests.

That's because - Logging is the developers best view into what is actually happening within an application.

Tools like New Relic, DataDog, Prometheus, Jaeger, OpenTelemetry, etc are wonderful tools for collecting data about what it happening with an application. I don't want to discount the value of tools like these but for the sake of this article, I will point out that these tend to be more useful for observing an application and identifying when there are changes in patterns, healthiness or other  symptoms that could indicate a problem.

Developers are given requirements to capture metrics about the application, or to build in support for (of?) tracing across a distributed architecture... and next thing you know, suddenly trying to find information about any specific error or request is like looking for a black cat in a coal cellar. The signal-to-noise ratio is just too high to be able to use.

Knowing that there's a problem is not the same as understanding what's causing it, which leads me to my next point.

###  Logs are the best tool for understanding when something is or isn't working

As I already said - logs are the view into what's happening within an application or service. This is incredibly valuable when running the application locally but absolutely essential once that code is running in any other environment.

And this is where I've seen some teams put too much hope and reliance on other tools such as automated testing or debuggers - especially in local development. Which then doesn't translate towards understanding why something has failed once it's no longer running on a developers laptop.

Tests are a powerful tool for both design and protection and against regressions. They will tell me what the application is supposed to do but logs tell me what the application is actually doing.

Although I only use them sparingly, debuggers can be incredibly useful to help  understand how some piece of code works, but in my experience they are rarely useful beyond that and certainly not something that you can connect to a production instance.

It's for these reasons that I strongly advocate that logs are implemented as early as possible in the development cycle and that teams develop a practice of using them daily. If a development team is using a combination of both automated tests and log verification to validate feature correctness in their normal feature development cycles then that team will design and create better than average logs as a natural by-product. Ideally, part of the definition of done for any given story or feature is a demonstration of the logs that the feature can/will generate.

### Implementing logging later will end up costing more

Before joining LexisNexis I would often get pulled into projects where the sales person had promised that a new product would be completed by a certain deadline. The last time this happened to me, the  company had been throwing as many developer bodies  at the problem as they could find for several months. In order to try and speed up development the leadership of the project had made the decision to push teams to abandon anything and everything that wasn't in direct support of meeting the deadline. One of the casualties of this decision was that the teams were instructed that they would add logging to the application at "some point in the future".

The leadership justified this decision because they had staffed the teams with primarily  developers of intermediate to senior levels of experience. Because of their experience the code had some decently levels of quality and even a modest level of tests. These teams also maintained regular showcases to stakeholders showing progress and getting regular feedback to maintain an iterative approach. Through a herculean effort and the loss of many weekends - the teams did meet the deadline and the product was released into the wild.

However, as the famous Mike Tyson quote goes, "Everyone has a plan until they get punched in the mouth." Every application looks great until it has to deal with the real world (where real data and real users will cause many unforseen issues). Bug reports started coming and the teams were scrambling to troubleshoot them as without good logs they were hamstrung to investigate them.

Leadership scrambled to have teams prioritize to add logging through the application, but retrofitting logging onto something that wasn't designed for it nearly as frustrating as trying to write tests for legacy code.

Rather than this project being the huge success with a smooth hand-off to the customer, it was a slow painful and expensive death. Almost a year after the deadline that company was still investing (out of their own pocket) a smaller team of developers to support, troubleshoot and fix issues that were being discovered.

The principle is that logging can't be put off "til later". When teams push that pain to the end, then code will be designed and written in a way that makes it painful to add logging later without extensive refactoring. Equally important the context about what information would be valuable to log will be lost, with the end-result turning into a tangled mess of spaghetti logging statements scattered across the entire application and likely still not providing the necessary value.


### Logging is expensive but don't be afraid to log

One thing that we need to acknowledge is that logging isn't free. Anytime we introduce a  hit to any form of external i/o we're going to incur some performance penalty.  So we don't want to log **everything and the kitchen sink**. Our logs need to be succint so that we can find the information we need and it has the relevant data that we need to understand if something is working or not e.g. our signal-to-noise ratio needs to be low.

However, I've seen more teams fall into the trap of prematurely trying to optimize what goes into logs than I've seen teams suffer from having too much logged.

One example that I witnessed was a team of junior developers that was tasked with creating a set of continuous delivery tooling. While this team had put in some logging, it was inconsistent in terms of log formatting, what got logged, and even what areas of the code were responsible for generating those logs. While this was mildly painful from a maintenance perspective, what really caused this team the most pain is that they had made a decision to primarily only log when an error occurred.

Logging errors are useful when your application code is actually crashing, but completely useless when it's simply not working as intended.  As a consequence, every time an issue was reported, this team would spend days trying to identify the root cause. And when it was for an issuethat couldn't be reproduced locally you would discover that their git history would fill up with commits like:

```  
S-135180 Adding temporary logging in to test xxx  
S-135180 Adding temporary logging in to test xxx  
S-135180 Adding temporary logging in to test xxx  
S-135180 Adding temporary logging in to test xxx  
E-18058 temporary debug lines for build failure  
S-105438 Adding a temporary debug handler  
```  

Whenever you see multiple commits with the exact same message like this, you know that someone is having a bad day.

Contrast that with one of my last startups where we designed our APIs to log detailed information about every request and response that the API generated. Not intent with just capturing a simple response status, we wanted to be able to review what was occuring and be able to identify not just the technical details but the business value for each request.

Using a stream of those logs from production, we were able to have simple dashboards within the developer area that a visualization of live traffic and kept a running total of business success metrics. On more than one occasion, just having that information visible allowed the teams to proactively notice odd trends such as a retail location having lower than average redemptions. Because the teams could immediately look at logs for a given location they were able to immediately identify and remedy many issues even before clients were aware.


### Wrapping up
The core principle that I hope I have shared here is that logging is more complex and more valuable than many developement teams recognize.

A good set of logs can often make the difference between an outage that lasts for a few minutes to one that lasts for hours or days. And while there is no "one true way" to do logging - there are multitudes of ways to do it poorly.