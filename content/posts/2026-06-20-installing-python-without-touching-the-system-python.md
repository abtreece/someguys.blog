---
layout: post
title: Installing Python without touching the system Python
date: 2026-06-20
tags: ["python", "macos", "pcc-companion", "asdf"]
---

In the [last post](/posts/2026-06-19-setting-up-a-mac-for-python/) we set up the foundation: the terminal, the Command Line Tools, Homebrew, and a `~/projects` folder for your code. No Python yet. This post installs it, and it's where this series makes its first real departure from the book.

[*Python Crash Course*](https://nostarch.com/python-crash-course-3rd-edition) has you download Python from python.org and double-click an installer. That works, and if you've already done it, nothing here is going to break it. But it leaves you with a single Python wired into your whole machine, and no clean way to keep one project's libraries from leaking into another. We're going to install Python so that you control which version you're running and so that every project gets its own isolated set of packages. It's two tools doing two jobs, and once it's in place you stop thinking about it.

<!--more-->

## Where the book starts, and where it stops

The book's macOS path is: check `python3`, install the latest from python.org if it's missing, and from then on type `python3` for everything because on a Mac the plain `python` command points at "an outdated version of Python that should only be used by internal system tools," in the book's own words.

That last part is the tell. macOS ships its own Python for the operating system's use, and it is not yours to mess with. The book steers you around it by always typing `python3` and installing a second Python next to it. That's a reasonable way to avoid a landmine, but it leaves two things unsolved.

First, you get exactly one version of Python. The day you want to try something on a newer version, or a project needs an older one, you're back to downloading installers and hoping they coexist. Second, every package you install with `pip` goes into that one Python, shared by everything. Install two projects with conflicting library versions and they fight. The book doesn't address this until deep in the Django project near the end, where it finally introduces virtual environments because by then it has to.

We'll solve both now, up front, because they're the things that make everything after this easy.

**If you've done this before**, here's the short version: we're using asdf to manage Python versions and a per-project venv for isolation. If you've used pyenv, nvm, or rbenv, asdf is the one-tool version of all of them. If you used asdf years ago, note that the 0.16 rewrite changed things: it's a Go binary now, and `asdf set` replaced `asdf global` and `asdf local`. You can skim to the venv section.

## Install asdf

[asdf](https://asdf-vm.com) is a version manager. It installs and switches between versions of languages, Python today, and Node or Ruby or Go later if you ever want them, all through one tool. That last part is why we're using it instead of a Python-only tool: it sets you up for any language without learning a new manager each time.

Install it with Homebrew, which you set up in the last post:

```bash
brew install asdf
```

Now you need to tell your shell where asdf keeps its "shims," the small stand-in programs that make `python` point at the version asdf is managing. Add this line to `~/.zprofile`, the same file Homebrew wrote to:

```bash
echo 'export PATH="${ASDF_DATA_DIR:-$HOME/.asdf}/shims:$PATH"' >> ~/.zprofile
```

Then open a new terminal window, or reload the current one with `source ~/.zprofile`, so the change takes effect. Confirm asdf is working:

```bash
asdf --version
```

You should see a version number, something like `asdf version 0.19.0`.

## Give asdf what it needs to build Python

Here's the part that trips people up, and that no quick tutorial mentions. asdf doesn't download a prebuilt Python the way the python.org installer does. It compiles Python from source on your machine. That's good, it means the build matches your exact system, but it means the build needs a handful of libraries present, or it either fails outright or silently produces a Python missing features you'll want later.

Install those libraries with Homebrew before you build anything:

```bash
brew install openssl readline sqlite3 xz zlib
```

You only do this once. With these in place, the Python build will pick them up automatically.

## Install Python with asdf

Add the Python plugin, which teaches asdf how to build Python:

```bash
asdf plugin add python
```

Then install the latest stable version. This is the step that compiles from source, so give it a few minutes and don't worry about the wall of output:

```bash
asdf install python latest
```

When it finishes, see exactly which version landed:

```bash
asdf list python
```

You'll see something like `3.13.2`. Make that version your default everywhere by setting it in your home directory, using the number you just saw:

```bash
asdf set --home python 3.13.2   # use the version you installed, not literally this
```

The `--home` flag writes the choice to a file in your home folder, so it applies everywhere unless a specific project says otherwise. Now check that it worked:

```bash
python --version
which python
```

The version should match what you installed, and `which python` should print a path inside `~/.asdf/shims`. Two things worth noticing here. You now have a real, current Python that has nothing to do with the system one, so it's safe to use and safe to throw away. And `python` works, not just `python3`. asdf provides both, along with `pip` and `pip3`, so the book's "always type `python3`" caution no longer applies to you. Type whichever you like.

## A virtual environment for each project

asdf decides *which Python* you're running. A virtual environment decides *which packages* a given project can see. Those are two different jobs, and you want both.

**If you're brand new**, a virtual environment, or "venv," is just a private copy of Python's package area that belongs to one project. When it's active, `pip install` puts libraries there instead of in your global Python, so two projects can depend on different versions of the same library without ever colliding. It's a sandbox, and it's the single most important habit in Python.

Let's set this up the way you'll actually use it through the book. Make one container folder to hold all your Python Crash Course work, and pin a Python version for everything inside it:

```bash
mkdir -p ~/projects/pcc
cd ~/projects/pcc
asdf set python 3.13.2   # the same version you installed above, not literally this
```

That `asdf set` writes a small `.tool-versions` file here, using the version you installed earlier. Because asdf looks up the directory tree, every project folder you create inside `~/projects/pcc` inherits this Python automatically, so you pin it once and never think about it again.

Now make your first project folder inside that container. The book's early chapters keep their files in a folder called `python_work`, so we'll use the same name:

```bash
mkdir python_work
cd python_work
```

Create the virtual environment. The convention is to call it `.venv`:

```bash
python -m venv .venv
```

That makes a `.venv` folder holding this project's private Python and packages. Activate it:

```bash
source .venv/bin/activate
```

Your prompt changes to show `(.venv)` at the front, which is how you know it's active. From here, `python` and `pip` refer to the project's private copies. Install something to see it work:

```bash
pip install --upgrade pip
pip install pytest
```

When you want to record exactly what a project depends on, so you or anyone else can recreate it later, freeze the list:

```bash
pip freeze > requirements.txt
```

And to rebuild that same environment from the list on another machine, or after deleting `.venv`:

```bash
pip install -r requirements.txt
```

When you're done working, leave the environment with:

```bash
deactivate
```

One habit to start now: the `.venv` folder is large and machine-specific, so it should never go into version control. When we get to git a couple of posts from now, we'll add it to a `.gitignore`. The `requirements.txt` file, which is small and portable, is the thing you actually keep.

That `python_work` folder is just the first of several. The book has you create a new directory for each of its larger projects, and you'll make each one inside `~/projects/pcc`, each with its own `.venv`:

- `~/projects/pcc/alien_invasion` for the arcade game (Chapters 12 to 14)
- `~/projects/pcc/data_visualization` for the charts and API work (Chapters 15 to 17)
- `~/projects/pcc/learning_log` for the Django web app (Chapters 18 to 20)

The steps are the same every time: make the folder, create a venv, activate it.

```bash
cd ~/projects/pcc
mkdir alien_invasion
cd alien_invasion
python -m venv .venv
source .venv/bin/activate
```

Each project's packages now belong to that project alone. Your pygame game and your Django site never share a library, and you can delete one project's `.venv` and rebuild it from its `requirements.txt` without touching the others. You don't pin Python again in each folder; they all inherit the version you set on `~/projects/pcc`.

## Never use sudo pip

One rule that will save you a bad afternoon: never run `pip install` with `sudo`. If a command ever fails with a permissions error and the internet suggests adding `sudo`, that's the sign you're trying to install into a Python you don't own, usually the system one. With asdf and a venv, every Python you touch is yours, so you never need elevated permissions to install a package. If you reach for `sudo pip`, stop and check which Python is active instead.

## How to keep following the book from here

You're set up to do everything *Python Crash Course* asks, with better tools underneath.

When the book tells you to type `python3 hello_world.py`, you can type `python hello_world.py` instead, or `python3` if you'd rather match the book exactly. Both work now.

The early chapters install nothing, so you'll work in `~/projects/pcc/python_work` with its venv active and nothing will feel different from the book. The projects are where our approach and the book's diverge in a way worth knowing. To install a package, the book mostly uses a command like `python -m pip install --user pygame`. Drop the `--user` flag. Activate that project's venv and install without it:

```bash
pip install pygame
```

The `--user` flag installs into your global Python, the exact clutter a venv exists to prevent, and inside an active venv you never need it. The book only switches to a virtual environment for its Django project in Chapter 18 (it even names one `ll_env`); you'll have been using one for every project since the start, so that step will be something you already do.

## What you've actually done

You installed a current Python that's entirely your own, made it switchable for any future version or language, and learned the one habit, a virtual environment per project, that keeps a Python setup from turning into a mess a year from now.

The book gets you one Python and asks you to tiptoe around the system one. You've got a version manager and clean, disposable environments instead. Next we'll set up VS Code and point it at exactly the Python you just built.
