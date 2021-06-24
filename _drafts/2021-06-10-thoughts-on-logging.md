---
layout: stories
title: Thoughts on Logging
description: Simple thoughts and guidelines on logging
author: Eldon
hero:
   href: 'https://source.unsplash.com/EaB4Ml7C7fE/600x250'
   alt: Developer working
---
## DRAFT

![Developer Working](https://source.unsplash.com/EaB4Ml7C7fE/800x450)

These days I don't get as much dedicated developer time as I did earlier in my career. However, when I do start a new application - one of the earliest decisions I make is what I'm going to do about logging. It's such a critical component for both how I'm going to develop the application and how I'm going to deal with it running in production that the decisions around this process need to be made within the first iteration.


I joined the LexisNexis global engineering team as a Technical Fellow in September last year. Providing guidance and coaching is a big part of what we do. So even when I am not coding per se, I can't begin to count the number of times in this job or previous roles when another developer has asked for my help in troubleshooting some error, only for us to immediately hit a brick wall because they had anaemic logging.


To help kick off our new engineering blog, I thought I would share a few thoughts about an area that I still see many developers struggle with: the importance and purpose of logs.


Every application is going to have different requirements and constraints for how and what they log. An API running on a server will have vastly different logging requirements than an application running on an end-user's machine, and different industries or problem domains will have man????. This blog is not a list of rules or commandments. Instead, I hope to add value by sharing some war stories of painful experiences that I've faced myself or witnessed over the years, and by extracting some of the guiding principles that I use.


### Logging is not about metrics, tracing, etc.

As an industry, we have a serious problem with conflating ideas together, and logging is no exception. 


Developers are given requirements to capture metrics about the application, or to build in support for (of?) tracing across a distributed architecture... and next thing you know, suddenly trying to find information about any specific error or request is like looking for a black cat in a coal cellar. The signal-to-noise ratio is just too high to be able to use.


Instead, there are a wealth of tools such as open-telemetry, prometheus and Jaegar which can be used to capture this data separately from your logs in a cleaner way.


### Logging is your best view into what an application is doing

The first principle that I use is that logs should be written for the people who need to support the application (both developers and operations).


Logs are the view into what's happening in my application or service. Which is incredibly valuable when running the application locally but absolutely essential once that code is running in any other environment.


Tests will tell me what the application is supposed to do, but logs tell me what the application is actually doing. Debuggers help me understand how some piece of code works, but in my experience they are rarely useful for troubleshooting a production issue.


### Implementing logging later will end up costing more

At a previous company, I got pulled into an existing project where the sales person had promised that a new product would be completed by a certain deadline. The company had been throwing bodies at the problem and made the decision to push for abandoning anything and everything that wasn't in direct support to meeting that deadline. It was a painful but also kind of a fun bonding experience with everyone else on the team. The decision had been made (prior to the time when I joined one of the teams) that stories would be written to add logging into the app at "some point in the future". To be fair, on the positive side of things TDD wasn't prevented, but it also wasn't encouraged, and since this was a group of developers with intermediate to senior levels of experience at least the code was high quality and had some level of tests to help. The teams also did regular showcases to the stakeholders showing progress and getting regular feedback.


Through a herculean effort the teams met the deadline and the product was released into the wild. However, as the famous Mike Tyson quote goes, "Everyone has a plan until they get punched in the mouth." Every application looks great until it has to deal with real data and real users. Suddenly, bug reports started coming and the teams had no good way to investigate them. Everyone scrambled to add logging but retrofitting logging onto something that wasn't designed for it is about as frustrating as trying to write tests for legacy code. Rather than being the huge success with a smooth hand-off to the customer, for almost a year later, that company was still investing in (out of their own pocket) a (smaller) team of developers to support, troubleshoot and fix issues that were being discovered.


The principle is that logging can't be put off "til later". Code will be written in a way that makes it painful to add logging later without refactoring, context about what information would be valuable to log will be lost, and the end-result will be a spaghetti mess of logging statements scattered across the entire application.


### Structured or Unstructured Logging

One question that comes up often is in what format we should publish logs. Should we use plain text files or put them into something more structured? There are pros and cons to both approaches. My only guiding principles here are that either way, we need to have consistency across all applications logging into the same format and they they still need to be human readable / parseable even in a raw state.

You don't always have to go to a fully machine-parseable standard such as JSON. You can decorate your logs with some smaller tags and let a tool like Splunk provide useful indexes on those tags.

### Logging levels

This might be somewhat controversial, but I believe that logging levels are rarely valuable and end up introducing more complexity than any benefit they provide. Information about something happening in your application is either valuable for logging or it's not.

In my experience, supporting logging levels creates needless debates within teams over whether something should be logged as informational, a warning, or an error. And far too often, you'll end up having to go read the code to rediscover at what level something was logged. 

It just takes too much time after discovering that there was an error to reconfigure the system to change the logging levels, and then attempt to reproduce the scenario that caused the error.  

Dave Cheney wrote a great blog article about this subject https://dave.cheney.net/2015/11/05/lets-talk-about-logging. 


### Logging is expensive but don't be afraid to log

To counter-balance my earlier point about trying to keep the signal-to-noise ratio low, I wanted to share two more stories. 

The first is about a team of junior developers that was working on putting together a set of continuous delivery tooling. While this team had put in some logging, it was inconsistent in terms of log formatting, what got logged, and even what areas of the code were responsible for generating those logs. While this was bad, what really caused this team the most pain is that their logging was really only focused on errors.

This is fine if your application code is actually crashing, but completely useless when it's just not working as intended.

As such, every time an issue was reported, this team would spend days trying to identify the root cause, especially for issues that couldn't be reproduced locally.  This led to their git history being full of commits like:
```
S-135180 Adding temporary logging in to test xxx
S-135180 Adding temporary logging in to test xxx
S-135180 Adding temporary logging in to test xxx
S-135180 Adding temporary logging in to test xxx
E-18058 temporary debug lines for build failure
S-105438 Adding a temporary debug handler
```
Whenever you see multiple commits with the exact same message like this, you know that someone is having a bad day.

Contrast that to the second story about one of my last startups where we designed one of our APIs to log detailed information about every request and response that the API generated - capturing not just the technical details but the business value for each request. We were then able to have simple dashboards based on those logs that displayed business success metrics in the development area. On more than one occasion, just having that information visible allowed the team to notice an odd trend such as a retail location having lower than average redemptions, and to proactively discover and fix an issue before the clients were even aware.

### Wrap up

At the end of the day, the lesson here is that logging is more complex and more valuable than many developers recognize. A good set of logs can often make the difference between an outage that lasts for a few minutes to one that lasts for hours or days. While there is no "one true way" to do logging - there are multitudes of ways to do it poorly.



