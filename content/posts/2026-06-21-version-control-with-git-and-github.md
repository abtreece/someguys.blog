---
layout: post
title: "Version control from the first exercise: git and GitHub"
date: 2026-06-21T12:00:00
tags: ["git", "github", "macos", "pcc-companion"]
---

We've built a machine that's ready to work: [the terminal and Homebrew](/posts/2026-06-19-setting-up-a-mac-for-python/), [Python with asdf and per-project virtual environments](/posts/2026-06-20-installing-python-without-touching-the-system-python/), and [VS Code wired up to use them](/posts/2026-06-21-a-real-editor-vs-code-for-python/). This post adds the thing that turns "files on a laptop" into "work you can't lose": version control with git, and a backup of it on GitHub.

<!--more-->

## Where the book starts, and where it stops

Credit where it's due again: [*Python Crash Course*](https://nostarch.com/python-crash-course-3rd-edition) covers git, and covers it well, in Appendix D. If you want the full mechanics of the local workflow, taking snapshots, undoing changes with `git restore`, rolling back with `git reset`, reading the log, that appendix is genuinely good and worth reading start to finish. I'm not going to re-teach what it already does properly.

What it leaves out is the part that matters most once you care about your work: **GitHub**. The book's git coverage stops at your laptop. Every commit lives in a hidden `.git` folder on your machine and nowhere else, so a dead drive or a lost laptop takes all of it. Getting your work onto GitHub fixes that, and it's also how you'd ever share code, back up across machines, or show someone what you've built. The book never makes that jump.

Two smaller things are worth correcting for our setup. The book tells you it's fine to "make up a fake email address" when configuring git. That's true for purely local work, but the moment you push to GitHub, your commit email is how your work gets attributed, so we'll use a real one. And the book's `.gitignore` doesn't mention virtual environments, because it barely uses them; ours does, on every project, so we have one more thing to ignore.

This is also where the foundation pays off: you don't need to install anything to start.

## You already have git

Remember the Command Line Tools from the first post? They included git. So unlike the book's "Installing Git" section, you can skip straight to using it. Confirm it's there:

```bash
git --version
```

You'll see a version number. That's all you need.

## Configure git, with a real email

Git stamps every commit with a name and email. Set yours once, globally:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

Use the email you'll use for GitHub here, not a fake one. This is the one place I'd diverge from the book's advice deliberately: a fake email works for local-only commits, but on GitHub it's how your commits are tied to you, and a wrong one means your work shows up as belonging to nobody. (If you'd rather not put your real address in every commit, GitHub can give you a private `noreply` address to use instead; you'll find it in your GitHub email settings once you've made an account.)

The last line makes every new repository start on a branch called `main`, which is the modern default and what GitHub expects.

## Put your work in a repository

We'll track all your *Python Crash Course* work in a single repository at the container folder from the Python post. Go there and initialize it:

```bash
cd ~/projects/pcc
git init
```

Before you commit anything, tell git what to ignore. Some files have no business in version control: the virtual environments (large and machine-specific), Python's compiled bytecode, and macOS's folder-metadata files. Create a file named `.gitignore` in `~/projects/pcc` with these lines:

```text
.venv/
__pycache__/
.DS_Store
```

These patterns have no leading slash, so they match at any depth. That matters for our layout: this one file at the container root keeps the `.venv` out of every project subfolder, `python_work`, `alien_invasion`, and the rest, without needing a `.gitignore` in each one. The first line is the one we promised back in the Python post, so you're versioning your code and not a copy of Python. The second is the book's `__pycache__/` line, and the third is the `.DS_Store` file the book's macOS note also flags.

Now make your first commit:

```bash
git add .
git commit -m "Start Python Crash Course work"
```

That's a snapshot of everything in `~/projects/pcc` except what `.gitignore` excludes. One thing that does get tracked is the `.tool-versions` file asdf wrote at the container root back in the Python post. That's intentional: it records the Python version for the whole project tree, so the version travels with the repository and anyone who clones it knows exactly what to run. From here, the day-to-day loop is the one Appendix D walks through in detail: change some files, `git add`, `git commit`. Read that appendix for `git restore`, `git reset`, and reading the log; those are the local safety nets and they're well explained there. The piece the book never reaches is next.

## Back it up on GitHub

Right now your history exists only on this laptop. Let's get it onto GitHub.

First, make a free account at [github.com](https://github.com) if you don't have one.

Then install GitHub's command line tool, `gh`, which makes the connection between your machine and your account painless:

```bash
brew install gh
```

Authenticate it with your account:

```bash
gh auth login
```

This walks you through a few prompts: choose GitHub.com, pick HTTPS when it asks about the protocol, and authenticate in your browser when it opens. It will also offer to create and upload an SSH key for you; say yes, and you've handled the part that a manual setup would make you do by hand.

**If you've done this before**, this is the mental shift from the centralized version control of an earlier era. If your last serious version control was CVS, Subversion, or Perforce, git is distributed: your commits are complete and local, and pushing to GitHub is a separate, deliberate step rather than the act of committing itself. GitHub is just a place you've agreed to send copies. Nothing forces you to push, which is freeing and occasionally a foot-gun when you forget to.

Now create a repository on GitHub from your local one and push it, all in a single command, run from inside `~/projects/pcc`:

```bash
gh repo create python-crash-course --private --source=. --remote=origin --push
```

That creates a private repository called `python-crash-course` on your account, connects it to your local repo under the name `origin`, and pushes everything you've committed. I'd keep it private while it's your learning work; you can make it public later if you ever want to show it off. Open the repository on GitHub and you'll see your files, your commit, and your `.gitignore` doing its job: no `.venv`, no `__pycache__`.

## The everyday loop

From now on, the rhythm is short. As you work through chapters, commit when something works:

```bash
git add .
git commit -m "Finish Chapter 5 exercises"
```

And every so often, send your commits to GitHub:

```bash
git push
```

That's the whole habit. Commit often, in small working steps, and push when you've done a meaningful chunk. The commits protect you locally; the push protects you against losing the laptop.

## How to keep following the book from here

Read Appendix D for the local mechanics; it's the best part of the book's tooling coverage, and everything in it works exactly as written on your setup. The only change is the one we made up front: use your real (or GitHub `noreply`) email when you configure git, not the fake one the appendix suggests, because you're pushing to GitHub now.

Beyond that, there's nothing to reconcile. Work through the chapters in your project folders, commit as you go, and push when you've got something worth backing up.

## What you've actually done

Your work is now snapshotted locally and mirrored on GitHub. A mistake is a `git restore` away, a dead laptop costs you nothing, and the code you write over the next months has a home that isn't just one machine.

That's the whole working setup: a clean Mac, a Python you control, isolated projects, an editor that understands them, and version control backing it all up. There's one more thing worth talking about, and it's the part nobody had when this book was written. Next we'll talk about learning to program in the age of AI.
