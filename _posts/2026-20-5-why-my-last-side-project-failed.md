---
layout: post
title: A reflection on why my last side project failed
description: Me ranting about why my last side project failed, some disappointments and my plan for the next one
date: 2026-05-20
permalink: /why-my-last-side-project-failed/
categories: rant
---

TLDR:

- The side project failed due to the lack of planning, unforeseen technical
  difficulties and the lack of motivation
- Not all is in vain since I had some experiences and new skills learned
- The next one will be carefully executed, no rush

Since the beginning of 2026, for two months, at least 1 hour per day, I was working
on a side project called [Task Manager](https://codeberg.org/thuyencode/task-manager).
The idea was simple, it was based on my actual need for a pomodoro-based task manager
app to help me effectively manage task across clients and projects.
And it could automatically calculate my pay as well.

To be fair, almost all the core functionalities worked as expected like pomodoro,
task creating, editing and deleting, etc. But it did not meet my expectation and
seemed underwhelming compared to other alternatives such as [Super Productivity](https://super-productivity.com/).
The progress didn't go as fast as I wanted, due to the fact that I chose to use
Next.js in order to learn about it as well. The mental model shifting required
some time.

The implementation of my ideas was a lot trickier than I thought, especially
the pomodoro one. How to hydrate a pomodoro session from the server,
auto-switch to the next session, pause and resume one, what to do when a session
is completed... It not a pure client-side app so there are a lot of things to
consider because you have to think about the state on the server as well and
make sure both client and server are in sync.

Honestly I don't want to look at my old code again.

Not to mention the lack of planning (I think I'm repeating myself here). Back then
I just simply jumped straight into coding and tried to put my scrambled ideas on
the way. Didn't end well as you know. No plan, no concrete design, just whatever
was in my head patched together with the best design patterns I know
(I made it sound fancy but it's just solutions).

On the upside, I learned a ton about NextJS, again. And the 16 version is the most
pleasant one to work with. With Cache Components, no longer I need to think about
which page should be SSG, SSR, ISR or CSR or whatever buzzword there is.
Simply focusing on which component needs which data helped me reduce a lot of the
cognitive load I had with older NextJS version. I also learned about Zustand,
the new and happy way to fetch data and handle errors in React 19,
and I made my own context menu from scratch!

So you may ask, what's next? Having no projects to work on makes me feel uneasy,
you know. I need something to keep me busy so I don't need to face the truth my
brain has been trying to tell me. One of the reasons this project failed is because
it's not fun! There's no incentive for anyone but me to use it, and I already have
Super Productivity. So I decided to abandon it. I felt guilty about all the time
I wasted on it tho. And from that shame, this time I'll make sure such a mistake
shall not be repeated. I must have a flagship project I can be proud of and
that people want to use.

Ok, it was time to choose something to cook. I certainly didn't want to make
another multi-tenant SaaS or simply follow through a huge tutorial on YouTube.
No matter how impressive the outcome is, it'll not leave a deep mark on my brain
if someone holds my hand all the time. I want to be its architect and builder
from start to finish.

I was wondering about it for a week. Maybe I should abandon the idea of spending
my precious remaining time left just practice interviewing? Then I came up with
an idea. What if you could draw with your friends without seeing each other or
touching some grass? Well, my idea is an app that lets you and a few friends, if
you have any, join a room and share the same canvas. You can see what the others
draw as well. You can also chat and send emotes which is funny and is nice to have.

Yay I had the idea. It's time for the planning part. For now on, at least one
per week, I am going to publish a blog post about the progress of planning
this app. My future self will thank me for this.
I am my only audience here anyway 😆

Happy coding. Stay tuned.
