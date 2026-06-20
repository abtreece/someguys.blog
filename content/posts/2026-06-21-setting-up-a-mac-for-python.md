---
layout: post
title: Setting up a Mac to learn Python, the way you'd actually want to
date: 2026-06-21
tags: ["python", "macos", "pcc-companion", "homebrew"]
---

I bought my dad a MacBook Air for Father's Day. He joined Sun Microsystems in 1990 and spent two decades there writing Java, at the company that invented it, then moved to Oracle when it bought Sun in 2010. He retired in the summer of 2022. Thirty-two years, and he hasn't really opened a computer since. The plan is to get him back into it with Python, using Eric Matthes' [*Python Crash Course*](https://nostarch.com/python-crash-course-3rd-edition), which is the book I'd hand anyone starting today.

There's a small joke buried in that. *Python Crash Course* opens with a dedication: "For my father, who always made time to answer my questions about programming." I'm running it backwards.

<!--more-->

Chapter 1 of that book takes you from a blank machine to `Hello world!` in about five pages. For a lot of readers that's exactly the right amount. But it skips almost everything that makes a computer pleasant to work on for the next year, and it makes a couple of choices I wouldn't make. So this is the first post in a series that fills that gap. It's written to sit alongside the book, but if you found it because you searched "set up a Mac for Python," it stands on its own and you don't need the book to follow it.

A note on who this is for. I'm writing it for someone capable but rusty: maybe you wrote code professionally a while ago, maybe you've just never done it on a Mac. **If you're brand new**, I'll flag the spots where you'd otherwise get lost. **If you've done this before**, I'll flag the one or two things that have actually changed, so you can skim the rest.

## Where the book starts, and where it stops

Here's what *Python Crash Course* tells a Mac reader to do, and it's all reasonable. Check whether you have a recent Python by running `python3`. If it's older than 3.9, download an installer from python.org and run it. Double-click `Install Certificates.command`. Install VS Code and the Python extension. Make a `python_work` folder on your desktop. Run your first program from inside the editor.

That gets you to `Hello world!`, and the book is honest that it's the minimum. It says so itself, in passing: after you run your first program, "you'll likely see some additional output showing the Python interpreter that was used," and if you want to clean that up, well, see Appendix B. That's the whole shape of the problem. The real tooling is in the appendices, where a beginner never looks, and the main path optimizes for getting one program to run rather than for the machine you'll be living on for the next year.

What Chapter 1 never mentions: how to manage more than one version of Python without making a mess, how to keep each project's libraries from stepping on each other, how to install command-line tools at all, or how to save your work so you can't lose it. Some of that is in the appendices. Most of it we're going to do up front, because it's the part that actually makes the rest easy.

We start one layer below Python itself, with two things: a terminal you can stand to look at, and a way to install software from one line. Python comes in the next post.

## Open the terminal

The terminal is the text window where you type commands instead of clicking. Apple ships one. Open Spotlight with Command-Space, type `terminal`, press Return, and you've got it.

**If you're brand new**, this window is going to do a lot of the work in this series, and it looks more intimidating than it is. It's a prompt waiting for you to type something and press Return. That's the whole interface. You can't break the machine by typing the commands in these posts.

## A word about the shell, so it doesn't trip you up

When you open the terminal, the thing reading what you type is called a shell. On a new Mac, that's **zsh**, pronounced "zee shell." You'll see references everywhere to **bash**, an older shell, and it's worth thirty seconds to explain why they're different and why you can ignore the difference.

**If you've done this before**, you almost certainly knew bash. The good news is that for everyday interactive use, zsh is bash with the rough edges sanded off. The commands you remember (`cd`, `ls`, `export`, pipes, redirection) all work the same. Your shell configuration lives in `~/.zshrc` now instead of `~/.bashrc`, and tab completion and globbing are smarter, but you are not learning a new language. You're using the one you know, with better defaults.

So why did it change? Apple froze the copy of bash it ships at version 3.2, from 2007, because newer bash uses a license Apple won't distribute. Rather than ship a permanently ancient shell, Apple made zsh the default back in macOS Catalina in 2019. "Use bash instead" on a modern Mac means either living with a shell from 2007 or installing a newer one and fighting every default that assumes zsh. It isn't worth it.

The practical advice is the easiest advice: use the default, don't treat it as something to study. And one thing not to do yet: do not install oh-my-zsh or any plugin framework on day one. That's how people lose an afternoon and end up with a slow shell full of features they don't understand. The shell as it ships is good. Come back to that when you have an itch to scratch, not before.

## Install the Command Line Tools

Before you can install much of anything, macOS needs Apple's Command Line Tools. This is a bundle that includes a compiler, `git`, and the basic plumbing that almost every developer tool expects to find. Install it with:

```bash
xcode-select --install
```

A dialog box pops up asking if you want to install. Say yes and let it run. It pulls down a few gigabytes, so give it a few minutes.

If it tells you the tools are already installed, that's fine, you're done with this step. Note that you do not need the full Xcode application from the App Store, which is enormous and built for making iPhone apps. The Command Line Tools are the small piece you actually want.

## Install Homebrew

This is the part the book leaves out entirely, and it's the one that pays off the most.

[Homebrew](https://brew.sh) is a package manager. If you've used `apt` on Linux, it's that, for the Mac. Instead of hunting down a website, downloading an installer, and clicking through it every time you need a tool, you type one line and Homebrew fetches it, installs it, and keeps track of how to update it later. Once it's set up, installing almost anything is `brew install <name>`.

**If you've done this before**, this is the same Homebrew you remember. The only wrinkle worth knowing is the PATH step on Apple Silicon, which is below.

Install it by pasting this single line into the terminal and pressing Return. It's the official installer, straight from brew.sh:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

It will explain what it's about to do and ask for your Mac password. When you type the password you won't see anything appear, no dots, no asterisks. That's normal, it's just hidden. Type it and press Return.

When it finishes, it prints a short "Next steps" section, and on an Apple Silicon Mac (any M-series machine, which is all of them now) it tells you to run two more lines. This step matters: it's how your shell learns where Homebrew put things. Skip it and the `brew` command appears to vanish every time you open a new terminal. Run these:

```bash
echo >> ~/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

What that does, briefly: Apple Silicon Macs keep Homebrew in `/opt/homebrew`, and those three lines add it to your PATH so the shell can find `brew` and anything you install with it, in this terminal and every one you open from now on.

Confirm it worked:

```bash
brew --version
```

You should see a version number. If you'd like Homebrew to check itself over, run `brew doctor`, which looks for common problems and, on a fresh install, should tell you "Your system is ready to brew."

## Where your code will live

The book has you make a folder called `python_work` on your desktop. I'd skip that. The desktop is for things you're actively looking at, not a place to pile up a year of projects. And on a Mac there's a concrete reason to avoid it: if you have iCloud's "Desktop & Documents" syncing turned on, and plenty of people do, everything you put there gets uploaded and synced. That's fine for a spreadsheet and a genuine headache once a folder fills up with virtual environments and git repositories.

Make a dedicated directory in your home folder instead. I keep everything under `~/projects`:

```bash
mkdir -p ~/projects
```

The name is a matter of taste. `~/code`, `~/dev`, and `~/src` are all common, and macOS even reserves a special `~/Developer` folder if you'd rather use that. The only things that matter: keep it in your home folder rather than the desktop or Documents, and pick one and stick with it so you always know where your work is. When we make your first real project in the next post, it'll go here.

## Optional: a nicer terminal

You can finish this entire series in Apple's built-in Terminal. Nothing here needs more than that, and if you'd rather not think about it, skip this section with a clear conscience.

That said, the stock Terminal is bare, and now that you have Homebrew, replacing it costs one line. The one I'd reach for is **[Ghostty](https://ghostty.org)**: it's fast, it looks good with zero configuration, and it stays out of your way, which is exactly what you want when you have plenty of other new things to learn. It's made by Mitchell Hashimoto, who built Terraform and Vagrant, and it has come together quickly.

```bash
brew install --cask ghostty
```

The `--cask` part is how Homebrew installs full Mac applications, as opposed to command-line tools. After it finishes, Ghostty is in your Applications folder like any other app.

The other well-known option is **[iTerm2](https://iterm2.com)**, which is excellent and has been the standard upgrade for years. I'd steer a returning beginner to Ghostty anyway, because iTerm2's whole strength is deep configuration, and a preferences window with a dozen tabs is a way to spend an afternoon tuning a terminal instead of learning Python.

## How to keep following the book from here

If you're reading along with *Python Crash Course*, here's where you are. Chapter 1's next move is to install Python from python.org. Don't do that yet. We're going to install Python in the next post using a version manager, which is a cleaner approach that the book saves for later and never fully commits to, and I'll show you how to still run every exercise in the book exactly as written. Everything else in Chapter 1, the editor and your first program, we'll get to with better tools in the posts that follow.

If you're not reading the book, you've still done the meaningful part. You have a terminal, you understand the shell it's running, and you can install anything you need from one line.

## What you've actually done

You haven't written a line of Python yet, and that's the right place to stop. You've done the part Chapter 1 takes for granted: the Command Line Tools are in place, Homebrew is installed and on your PATH, your code has a home that isn't the desktop, and you have a terminal you don't mind looking at.

This is the unglamorous part where you make a machine pleasant to work on. It's worth doing once, properly, and then not thinking about again. Next we install Python.
