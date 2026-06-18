---
layout: post
title: "I reviewed the code. They wanted it deployed."
date: 2026-06-18
draft: true
tags: ["hiring", "interviews", "judgment", "engineering"]
---

## Is the work sample an audition?

A take-home exercise asked me to review a junior engineer's pull request the way a senior infrastructure engineer would. So that is what I did. I found the problems, explained why each one mattered, fixed them, added local tests across the stack and the CI pipeline the repo never had, and wrote the whole thing up like a real review. I ran out of time before I deployed it.

The deploy was the part that scored. I did not get the job.

I made a tactical error, and given the same exercise again I would deploy first and decorate later. The interesting part is not that I lost a point. It's that the exercise and I disagreed about what it was for, and the exercise had the only vote that counted.

<!--more-->

## What is left when the typing is automated

As I have argued [previously](/posts/2026-05-12-twenty-years-in-infrastructure/#the-delegation-delta-is-where-senior-engineers-live), typing has been automated, and what is left is judgment. Deciding what the problem actually is. Deciding what matters most and what can wait. Reading an ambiguous situation and committing to a call you can defend. That is the work now, and it is the work job descriptions are openly asking for. The staff-plus level role I interviewed for wanted someone who could understand the bigger picture, cut through ambiguity, and figure out what matters most. I have yet to see a job description ask for someone who can reverse a linked list from memory while a stranger watches.

So, the question for anyone designing a screen for a staff-plus engineer is no longer whether the candidate can produce code. Assume they can, or assume their tools can. The question is whether you can see staff-plus level judgment. And judgment turns out to be much harder to score than typing ever was.

I learned that twice in one interview loop, in two different ways.

## The stopwatch

The first interview was a live system design session. Forty-five minutes, a shared whiteboard, a hypothetical application spec with no real connection to the company's business. I read it, read it again, then attempted to break it down top to bottom in real time with an infrastructure engineer's bias. The format was reactive from start to finish.

The prompt itself was fine. Open-ended, no single right answer, rewarded decomposition over recall. On paper it is one of the better formats. The problem was not the prompt. The problem was the clock.

The thing an experienced engineer does that a newer one does not is sit with a design, turn it over, and come back with where it breaks. That move takes wall-clock time, and a forty-five-minute reactive session does not grant any. There is no point at which you get to think. You perform thinking, live, which is a different and shallower thing. I got to the load-bearing parts of that design eventually, with some prompting from the interviewer. Needing the nudge and not needing it are the same competence. They are not the same signal.

A good prompt run against a clock measures reaction speed, not judgment. The reflection budget decides whether you are measuring what someone produces under reaction or what they actually bring when the work is real. The clock does not just fail to measure the second thing. It punishes it, because the engineer who pauses to think reads as the engineer who did not know.

## The audition

The second interview was the take-home, and it failed to see judgment for a different reason. This one had all the time the whiteboard lacked. Two and a half hours, work on your own, reconvene at the end. Time was not the constraint. The constraint was that the prompt could be read two ways, and only one of them scored.

The bolded instruction said: find what is wrong, explain why, fix what you can, and deploy it. Read literally, that is a checklist, and the fastest path to points is to run the checklist and get the thing deployed.

The sentence right before the bold set the scene. A junior engineer drafted this infrastructure, got it working on their machine, and opened a PR, and your job is to review it as the senior infrastructure engineer. Read that way, the exercise is a code review, and the job is to demonstrate the judgment of the person they are hiring.

When I get a take-home, I read it as a chance to show how I work and how I communicate that work, not as a list of tasks to clear. So I took the second reading. I treated the PR as a real review. I caught a health check wired to a port the application was not serving, which would have failed every probe and crash-looped the pods. I caught a readiness probe pointed at an endpoint that returns a method-not-allowed error to the exact request the probe sends, so the orchestrator would have restarted working pods in a loop. I pulled a firewall rule that opened every port to the internet. I replaced an owner-level CI credential with least privilege. I added [test harnesses across the stack](/posts/2026-05-12-twenty-years-in-infrastructure/#whats-actually-working-for-me), watched them fail, then fixed the code, because that is how I would actually do it. I structured the findings so the work was reviewable.

It was a lot of real work, and it was the work the role described. And I ran out of runway before I deployed, which was one of the unambiguous, bolded, point-bearing deliverables.

This is the part worth sitting with. More time would not have fixed it. I had the time. I spent it on the review because the prompt invited it and I was focused on the bigger picture.

The two readings are not equally likely for everyone. A senior engineer, handed that prompt, very often does exactly what it says: find, fix, deploy, turn it in. That is the right instinct for the instruction as written, and it scores well. The reframing into "show me how you think about this system" is a habit you pick up further up the ladder, where the job stops being the task in front of you and starts being the judgment around it. The senior engineer does what the prompt says. The principal does what it implies. Both are looking at the same paragraph.

Which means the exercise is an audition only if the people scoring it know they are watching one. If the rubric is a find-fix-deploy checklist, it is a task, no matter how the candidate read it. The candidate does not get to decide. The scorecard does. And a scorecard built around the literal reading has no column for the things the implied reading is meant to show.

I own my half of this cleanly. A more test-wise candidate reads the bold, ships the deploy, and adds the review on top if time allows. That is the correct move under the actual scoring, and I did not make it. But notice what the scoring rewards: the candidate who optimized for the instruction over the candidate who optimized for the job. For a role whose entire premise is judgment under ambiguity, that is worth a second look.

## Both failures are the same failure

The stopwatch and the audition look like different problems. One is about time and one is about scoring. Underneath they are the same problem. In both cases the screen left a staff-plus engineer to guess which version of the job it was testing and then graded the result.

The whiteboard tested whether I could perform judgment without the time judgment requires. The take-home tested whether I would read the literal checklist or the implied role, and rewarded only the checklist. Neither was built to see the thing the job was buying. Both are easy to score, which is most of why they persist. Judgment is hard to score. That is a reason to work harder at the screen, not a reason to keep measuring the easy thing and call it judgment.

## If you are hiring

Plenty of people have written that the leetcode screen is obsolete. I am making a narrower claim. Even the formats that look like an upgrade, the open-ended design session and the take-home, will measure the floor unless you build them to measure judgment on purpose. Here is what on purpose looks like.

Tell the candidate what the rubric rewards. If the deploy is the thing that scores, say so. An experienced engineer should not have to gamble on which reading of your prompt you meant. Removing that guess does not make the exercise easier. It moves the test from "did you guess our intent" to "can you do the work," which is the thing you wanted to measure.

Score both readings. If your take-home can be read as a checklist or as a review, decide in advance how you weight one against the other, and grade both. The candidate who does a deep, correct review and misses the deploy is showing you something different from the candidate who deploys a shallow fix. A rubric that collapses them into one score is throwing away your best signal.

Give a reflection budget, not a stopwatch. If you want to see how someone thinks about a design, let them think. Send the prompt ahead of time. Let them sit with it and come back. An hour of reflection shows you how they think. Without it, the exercise rewards the candidate who has seen a similar problem before.

Evaluate the review, not just the result. The deploy is binary and easy to check. The judgment is in the diagnosis: what they caught, what they prioritized, what they chose to leave alone and why. That is the expensive part to grade and the only part that tells you whether this person can do the job you are hiring for.

None of this makes screening easier. It makes it match the work. A few companies have figured that out. Most have not. The interview that can't see judgment does not stop filtering for it. It filters it out, and the engineers it loses are the ones whose value never fit in the window it was measuring.
