---
layout: post
title: Letting AI help you learn without learning for you
date: 2026-06-22
tags: ["ai", "learning", "pcc-companion"]
---

The first four posts in this series set up a machine you can work on: [the terminal and Homebrew](/posts/2026-06-19-setting-up-a-mac-for-python/), [Python and virtual environments](/posts/2026-06-20-installing-python-without-touching-the-system-python/), [an editor](/posts/2026-06-21-a-real-editor-vs-code-for-python/), and [version control](/posts/2026-06-21-version-control-with-git-and-github/). This last one is about a tool that didn't exist when *Python Crash Course* was written, and that has changed what it's like to learn programming more than anything else on that list: an AI assistant sitting one keystroke away while you work.

<!--more-->

## Where the book stops

[*Python Crash Course*](https://nostarch.com/python-crash-course-3rd-edition) has a good chapter on getting unstuck, Appendix C, and the advice in it is still correct. Before you ask anyone for help, answer three questions: what are you trying to do, what have you tried, and what happened? Explain the problem out loud to a rubber duck. Take a break. Search the exact error message. Lean on Stack Overflow, the r/learnpython community, and the official docs. None of that has aged a day.

What's missing is the thing most people now reach for first. When you hit an error today, the odds are you'll paste it into [Claude](https://claude.ai) or [ChatGPT](https://chatgpt.com) before you open a forum, and the book has nothing to say about that, because it was written before these tools could hold a conversation about your code. That silence matters, because using an AI assistant while you're learning is genuinely different from using forums and docs, and it can help you enormously or quietly rob you of the thing you came for.

## The tutor and the crutch

An AI assistant is the most patient tutor you will ever have. It will explain the same concept five different ways at one in the morning without sighing. It will translate a wall of red traceback into a plain sentence. It never makes you feel stupid for asking. For learning, that is close to miraculous.

It is also the most tempting shortcut anyone has ever put in front of a student. Ask it to solve the exercise and it will, correctly, instantly, and the code will run. And there is the trap, sharper for a learner than for anyone else: you can produce working code you do not understand, and it will feel exactly like progress while teaching you almost nothing. A professional can lean on that and ship. A beginner who leans on it never becomes the professional.

So the whole game is using the tutor without taking the shortcut. Here's how I'd draw that line.

## Ask it to explain, not to produce

The single rule that keeps AI on the tutor side of the line: ask it to explain things, not to write things. "Why does this loop raise an `IndexError`?" teaches you something. "Write the solution to exercise 6-5" teaches you nothing. Both get you a working program; only one gets you a working programmer.

A few habits make this concrete:

**Type the code yourself.** If you ask for an explanation and it shows you code, retype it rather than pasting it, and don't move on until you can say what each line does. Pasting code you can't read is the purest form of the shortcut. The friction of typing is where a surprising amount of the learning actually happens.

**Do the book's first steps before you ask.** Appendix C's three questions, what am I trying to do, what did I try, what happened, are not just for forum posts. Answer them before you turn to the AI, and two good things happen. Often you'll spot the bug yourself, the way you would with the rubber duck. And when you don't, you'll ask a far sharper question, which gets a far better answer. The clarity that helps a human helps the model.

**Sit with the error for a minute first.** Reading a traceback and reasoning about it is a core skill, and it only develops if you practice it. Take a real swing at understanding the error on your own before you hand it over. Then, if you're still stuck, ask the AI to explain the traceback rather than fix it, and check its explanation against what you were already thinking.

## What it's genuinely great at

Used as a tutor, an AI assistant is extraordinary at exactly the things beginners struggle with:

- **Translating error messages.** Paste a traceback and ask what it means in plain language. This alone removes a huge amount of early frustration.
- **Explaining at your level.** "Explain list comprehensions to someone who understands `for` loops but not these yet." You can ask for the explanation pitched exactly where you are.
- **Reviewing your code.** Show it something you wrote and ask what's weak and why, not for a rewrite. This is the closest thing to having a senior developer read over your shoulder.
- **Checking your understanding.** Ask it to quiz you on what you just read, or to ask you three questions about a concept. Recall is how things stick.
- **Being there at midnight.** The forum might take a day to answer. The tutor answers now, which keeps you in flow.

## Where it goes wrong

Three failure modes are worth knowing before they bite you:

**It is confidently wrong sometimes.** An assistant will occasionally invent a function or a library feature that doesn't exist, and say so with total assurance. As a beginner you can't always tell. This is exactly why Appendix C's advice to check the official docs still matters: when the AI tells you something, running the code and confirming it against the real documentation is how you catch the confident mistakes.

**It will hand you code above your level.** Ask for a solution and you may get something elegant, idiomatic, and completely beyond your ability to debug when it breaks later. Code you can't maintain is a liability even when it works.

**It can erase the productive struggle.** The frustrating part, where you stare at a problem and slowly work it out, is not an obstacle to the learning. It is the learning. An assistant that removes every moment of difficulty also removes the part that builds the skill.

## A setup for learning, not just shipping

If I were setting this up for someone working through the book, I'd reach for a chat assistant, Claude or ChatGPT, as the tutor: a place to ask, discuss, and have things explained. That conversational mode is where the learning lives.

I'd be more careful with inline autocomplete tools like GitHub Copilot, which suggest whole lines as you type. For an experienced developer that's a real accelerator. For someone learning the fundamentals it's corrosive, because it writes the code before you've had the thought, which is the one thing you're there to practice. I'd leave that kind of autocomplete off until the basics feel solid, then turn it on once finishing the line yourself has become easy.

I'll be honest that I [lean on these tools heavily in my own work](/posts/2026-05-12-twenty-years-in-infrastructure/). The point isn't that AI is something to keep at arm's length. It's that the way you use it to *learn* is deliberately different from the way you use it to *produce*, and the difference is the whole post.

**If you've done this before**, coming back after years away, AI is a fantastic on-ramp. Ask it what's changed in Python since the version you last used, or for the modern idiom that replaced a pattern you remember. Just notice that the crutch risk applies to you too, in any corner of the ecosystem that's genuinely new.

## How to keep following the book from here

Use Appendix C's resources and an AI tutor together. When an exercise stumps you, ask the assistant to explain the concept or your error, not to hand you the finished answer. The test is simple: after the AI helps, close the chat and write the next exercise without it. If you can, you learned something. If you can't, you borrowed something, and it's worth going back.

## What you've actually done

Over five posts you've gone from a sealed laptop to a real development environment: a clean Mac, a Python you control, isolated projects, an editor that understands them, version control backing it up, and a sense of how to use the most powerful learning tool any beginner has ever had without letting it do the learning for you.

The setup is finished. What's left is the part the book was always really about: opening it to Chapter 1 and starting to write code. The tools are better than they've ever been. The work of understanding what you write is still yours, and it's still the whole point.
