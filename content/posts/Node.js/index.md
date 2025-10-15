---
title: Why I Use Meteor.js in 2022
date: "2022-11-19T15:40:32.169Z"
template: "post"
draft: false
slug: "why-i-use-meteor.js-in-2022"
category: "Node.js"
tags:
  - "Meteor.js"
  - "Node.js"
  - "Software Architecture"
  - "SaaS"
  - "MicroServices"
description: "Meteor.js is a great Node.js framework for hyper-development of SaaS solutions."
---

If you're a veteran in the Node.js/SaaS community, you probably have not seen the name Meteor.js in a while.

[Meteor.js](https://www.meteor.com) is a Node.js framework that first appeared back in December of 2011 with the project name [Skybreak](https://blog.meteor.com/first-preview-8d4675d7fe35).

Meteor.js pioneered many concepts junior full stack developers may take for granted today, such as:

- Real-time client-side updates out-of-the-box
- One language for front-end and back-end
- Latency compensation
- The concept, "Less is more".

## Software Development from a Business Perspective

When designing the architecture for a new piece of software I always think of it from a perspective that I am creating a business.

After all, when I build applications I am building it to solve a problem. Which at it's core is what every business is. A solution for a problem.

The success of that business depends on two factors:

1. The value of the solution to the problem.
2. The number of people in the world that experience the same problem.

## Opportunity Cost

A quote that I frequently read from successful SaaS founders is "Build fast, and ship often."

This advice is valid when you consider the opportunity cost associated with long release times of software.

User feedback can optimize your development scope by focusing on engineering features and functionality that provide more value for the user.

Two things you can take away from this advice:

1. Don't develop software in a vacuum. Get user feedback ASAP.
2. Don't reinvent the wheel.

If you are developing an application, there are some features that are universal. Things such as user accounts, email communications, data fetching, access control, etc..

Meteor.js includes these universal features out of the box utilizing industry best-practices.

As a developer, having access to these features as soon as I start a Meteor.js project I am able to prioritize development of the application's business logic; or **value creation**.

What I mean by **value creation** is the unique features and functionality of the application.

This enables me to launch products at lightning-speed to learn if the product is going to be successful... or not.

## Criticism: Meteor Doesn't Scale

One common misconception is that Meteor does not scale. This is both true and false.

Meteor.js is a **Node.js** framework. Meaning it is just Node.js under the hood.

The community believes that Meteor does not scale because a lot of developers leverage Meteor as a monolithic application.

Meaning that in order to scale their applications, they often have to vertically scale their cloud instances.

One arguement you can make is that you do not always have to vertically scale your application. Another option is to horizontally scale your application by launching multiple instances of your application.

Both options are valid options, however there becomes a point in the scalability of your app where you meet an equillibrium.

What I mean by this is that if you have an application that has a minimum resource constraint, for example 1ECU and 2GB of memory. You can spin up as many instances of that vertically-sized stack... horiztonally.

But from a business perspective this might not make the most sense because the vertical-stack constraints may be caused by a particular function, or feature, within your application.

## Introducing MicroServices

This is what makes Meteor a great option for start-ups.

Let's say you build your application extremely fast because Meteor allowed you to not "re-invent the wheel" with fundamental application functions.

You now have your application live and your userbase is growing quickly.

You have a new problem on your hands, trying to scale your application without breaking the bank.

In the previous section I mentioned reaching the vertical and horizontal equillibrium of a monolithic application. At this point it is time to introduce MicroServices to your DevOps.

For my example application I have 3 pages:

- Page 1 - Less Resource Intensive (Minimum Vertical-stack: 0.5ECU-1GB)
- Page 2 - More Resource Intensive (Minimum Vertical-stack: 1ECU-2GB)
- Page 3 - Less Resource Intensive (Minimum Vertical-stack: 0.5ECU-1GB)

The obvious outlier of the 3 pages is Page 2 which is causing the instance size to be twice as large as the other two pages.

The answer is to extract the business logic from your application and create a MicroService to only provide that function within your app.

You can utilize serverless solutions to provide access to this MicroService in an even more cost-effective manner.

## How I Organize My Meteor Projects for Business Growth

- /server
  - index.js (Start-up Logic, Method Imports)
- /client
  - /{Generic React App}
- /imports
  - /ui
  - /api
    - /{Collection Name}
      - index.js (Schema & Mongo Collection Initialization)
      - methods.js (Controller / Meteor Method Logic) - Handle method registration, authorization, routes, call service
      - publications.js (Controller / Meteor Publication Logic) - Handle subscription registration, authorization
    - /services
      - {CollectionService}.js - Handle business logic of application and return result to method

This is a very common project structure for Node.js applications. By isolating the server-side business logic of the application from the Meteor Method, you are able to easily extract the service from the monolithic architecture of Meteor into it's own service; resulting in a plug-and-play implementation of MicroServices to enable horizontal scaling.

I often rewrite services from Node.js to Rust/Golang when required to optimize resource utilization.

## Conclusion

I choose Meteor as the starting point for all my applications because of three reasons:

1. Shortest time to get "up and running" and start working on _value creation_
2. Battle-tested and long support history
3. Ability to easily scale applications through MicroServices when (or if) an application becomes successful
