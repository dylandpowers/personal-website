---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Striking the Balance: Development in a Fast-Moving Startup"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-02-09T18:55:28-05:00
lastmod: 2020-02-09T18:55:28-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
Perhaps made most famous by Facebook's "Move Fast, Break Things" mantra, the philosophy of iterating extremely quickly has become proliferate in the tech world. Tools like JIRA, GitHub, and suites of CI/CD tools give teams no excuse to not push out products and updates as quickly as they can. However, as an organization grows, developers need to take into account scalability and flexibility of codebases. As such (at least from what I have observed), larger organizations tend to slow down over time to accommodate more scrutinous code reviews and technological research.

![ourspace](featured.jpg)

Over the past five months, I have been a Software Engineer for [OurSpace](https://ourspaceapp.com/), a real-time Q&A platform for locations and spaces. Currently, we have released a full working product at Duke University, and we plan to expand as we continue to perfect our core product. Working on a startup means that we _do_ move extremely quickly; a lot quicker than anything I am used to. This might mean pushing out hundreds of lines of code every day to meet our strict deadlines and release goals. As such, we often do not have time to waste with overly scrutinous code reviews and setting up rigorous CI/CD pipelines.

At least, that's what I thought at first. Since then, I have realized that while we do need to hit deadlines, investing a little bit more time into flexibility really pays off in the long run. In this article, I aim to explain why it's important to strike that balance between moving quickly and developing flexible, robust code. 

### Creating Reusable Interfaces

Part of the excitement of working on a startup is the freedom in choosing which technologies to use - this also means switching technologies often. This means that, especially on the server-side, creating reusable interfaces is invaluable, as it allows future devs to easily swap out implementations.

When developing quickly, it's really easy to fall into the mindset of "I'm just going to make a class that works right now, so there's no need to create an interface." Yes, the development process will be easier _now_, but what happens in the future when another developer has to swap out the implementation? They will spend at least twice as long as they would if you had just made an interface.

To give you an example: when we first started on this journey, we used Google's [Cloud Firestore](https://firebase.google.com/docs/firestore) for all of our data storage. This meant clients on the backend for communicating with our Firestore. Luckily enough, our early developers created a `DbClient` interface that  declared methods for communicating with _whatever_ db we chose, so when we finally made the choice to migrate to [MongoDB](https://www.mongodb.com/), the transition was effortless. Even though we lost some early development time creating these interfaces, we more than made up for it by saving a plethora of time swapping out our implementation.

![mongodb](mongodb.jpg)

### Documenting Patterns

Writing good documentation is yet another action that most reserve for medium-to-large companies only. It's also something that takes time which could be "better spent" working on features and moving us toward our project goals. However, despite the initial overhead, writing down our most-used design patterns in plain English has saved us countless time.

I recommend creating a wiki for all different areas of development; for us, that includes server-side, our iOS client, and DevOps, among others. In that wiki, write down all of the common practices used throughout the organization so that developers can turn to it when initially planning out a feature. This saves developers a lot of time that would otherwise be spent either brainstorming a new design (convoluting the existing codebase) or mapping out existing code to understand how a design works. Writing it down in plain English will take 5-10 minutes, but will save developers hours of time in the future.

### Automating Processes

This should go without saying, but always reserve time at the beginning to automate **everything**. Run tests on every push. Connect GitHub to your messaging platform to get notifications whenever a PR is created. Deploy whenever a PR is closed. While this may cost some time at the beginning doing research on automation tools, the amount of time saved is astronomical. Please, set some time aside right now, and ask how you can best automate your current workflow.

![cicd](cicd.jpg)

Protip: [GitHub Actions](https://github.com/features/actions) are a really cool and easy way to automate workflows if you use GitHub for hosting.

--- 

Working on a startup has been incredibly challenging given the required speed of development. However, as I continue to develop mass amounts of code daily, I thank my past self for designing things well enough to make development for present-day me much easier. When embarking on this long, rewarding journey, don't fall into the "move fast" mentality unless you append "in an well-thought out manner" to the end of it. 
