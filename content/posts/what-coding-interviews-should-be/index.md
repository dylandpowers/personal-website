---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "What Coding Interviews Should Be"
subtitle: ""
summary: "My vision of what coding interviews should involve."
authors: []
tags: [Coding, Algorithms, Data Structures, Interview, Software Engineering]
categories: [Interviewing]
date: 2022-05-16T19:42:35-04:00
lastmod: 2022-05-16T19:42:35-04:00
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
Today at work, I needed to solve the knapsack problem for an important feature. I had to figure out how to put *n* items into a knapsack of weight at most *W*, and maximize the total cost.

After that, I was working on a feature with a graph, and I had to figure out the minimum number of nodes that could be removed while maintaining a connected network.

To wrap up the day, product asked me to find the longest common substring between two strings for a client.

---

Actually, none of this happened. Most of my day *actually* involves writing business logic for our system, discussing specifications with product, reviewing code, and monitoring deployments. I’m willing to bet that, if you’re a software engineer, your day looks somewhat like this as well, with some minor differences.

Unless you work at LeetCode, it’s unlikely that you encounter the problems I mentioned above more than once or twice a year, if at all. These are abstract topics that we scarcely see outside of coding interviews. However, we have widely accepted that coding interviews should and will be like this for the foreseeable future. I ask, though: why are tech companies often evaluating software engineers on their knowledge of dynamic programming, graph theory, and niche algorithms if they are scarcely used on the job? 

I argue that knowledge of these topics is *not* a direct indicator of how well somebody will perform on the job. I still believe we need to evaluate coding ability (obviously), but I think and know that we can do better than the current system. I’ll outline my thoughts here. 

# The Argument For

The main argument for these types of interviews is that they demonstrate the candidate’s thinking style. Companies look for candidates who think out loud and who can translate their thoughts into code. I completely understand that. I think the ability to translate abstract concepts in your mind to code on the screen is the crux of software engineering.

Let me walk you through a typical coding interview. The candidate receives the prompt, and remembers the general approach for tackling these kinds of problems - an approach they likely learned on LeetCode. The candidate begins to outline their approach, pulling in learnings from previous LeetCode problems, and combining them to form a plan. Then, the candidate translates those thoughts into code and writes some test cases.

Yes, the candidate demonstrated the ability to translate their thoughts into code. But *what* thoughts did they translate? Likely, they translated thoughts about past LeetCode problems and abstract concepts. Can we really say that those are thoughts that the candidate will have on the job? It’s more likely that thoughts on the job will be related to architecting systems, creating simple APIs for others to consume, reviewing code for style, designing database schemas, etc. 

# The Ideal Interview

As I mentioned above, companies still need to assess software engineers on their ability to translate thoughts into code. Companies also need to get a glimpse into how a candidate reasons about a problem from start to finish. Finally, companies should still require a proficiency with basic data structures (maps, lists, queues, etc.). We can still accomplish this with a different style of interview.

In my opinion, the ideal engineer can write **clean, maintainable code** with an **easy-to-use API.** I propose that we begin to ask engineers more questions about designing entire **classes** with multiple pieces of functionality**,** rather than just implementing singular methods with a clear expected answer.

For example, a company could ask a candidate to design an in-memory message broker. We would expect methods for `enqueue` and `poll`, and we would encourage the candidate to add any other methods they see fit (maybe to get the number of currently enqueued messages),

In this scenario, there is no one correct answer, so a candidate would not fail solely because they did not generate the correct output. Instead, we would evaluate candidates more holistically:

1. How readable is the code? Are variables named well?
2. How maintainable is the code? Are pieces of functionality split out into methods?
3. How easy-to-use is the API? Could another team easily consume this?
4. Are implementation details hidden?

I believe that these assessments more accurately reflect skills that the candidate would actually use on the job. 

---

As an industry that powers so much of the world, we can and should do better with evaluating candidates. Companies often miss out on top talent just because a candidate can’t solve the knapsack problem, find the minimum number of nodes that can be removed to maintain a connected graph, or find the maximum length substring of two strings. Let’s start evaluating candidates on skill used on the job.

---

_Header image courtesy of [iMocha](https://www.imocha.io/platform/live-coding-interview)._