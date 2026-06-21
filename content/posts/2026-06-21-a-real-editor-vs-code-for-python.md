---
layout: post
title: "A real editor: VS Code for Python"
date: 2026-06-21
tags: ["python", "macos", "pcc-companion", "vscode"]
---

So far in this series we set up [the terminal and Homebrew](/posts/2026-06-19-setting-up-a-mac-for-python/), then [installed Python with asdf and gave each project its own virtual environment](/posts/2026-06-20-installing-python-without-touching-the-system-python/). Now we need a place to actually write code. This is the one area where the book and I mostly agree, so this post is less about replacing its advice and more about finishing the job it starts.

<!--more-->

## Where the book starts, and where it stops

Credit where it's due: [*Python Crash Course*](https://nostarch.com/python-crash-course-3rd-edition) recommends VS Code, and it's the right call. VS Code is free, built on an open-source core, runs everywhere, and is the rare editor that's friendly to a beginner and still used by professionals all day. Chapter 1 has you install it and add the Python extension, and that's genuinely most of the way there.

Where it stops is the connecting tissue. The book downloads VS Code from a website; we'll install it with Homebrew, like everything else, so it updates with one command alongside the rest of your tools. The book says VS Code "finds the versions of Python you have installed" and "typically does not require any configuration to run your first programs," which is true for the single global Python it expects you to have. But we did something better in the last post: we gave each project its own virtual environment. VS Code won't use that automatically, and pointing it at the right one is the step that makes everything click. Finally, the book leaves formatting, line-length guides, and the rest of the useful configuration in Appendix B, which a beginner reading front to back won't reach for a long time. We'll set up the parts worth having now.

## Install VS Code with Homebrew

You installed Homebrew in the first post, so this is one line:

```bash
brew install --cask visual-studio-code
```

The `--cask` flag, the same one we used for Ghostty, tells Homebrew to install a full Mac application. When it's done, VS Code is in your Applications folder, and you can launch it like any other app.

That install also gives you a `code` command in the terminal, which is more useful than it sounds. From any folder you can type `code .` to open that folder in VS Code. Confirm it's there:

```bash
code --version
```

If you see a version number, you're set.

## Add the Python extension

VS Code on its own is a good general text editor. The Python smarts, code completion, error highlighting, running, and debugging, come from an extension.

Open VS Code, click the Extensions icon in the left sidebar (it looks like four squares), search for `Python`, and install the one published by Microsoft. It pulls in a couple of companions automatically, Pylance for fast code intelligence and a debugger, so installing the one extension is all you need to do.

**If you've done this before**, this is the mental adjustment from a heavyweight IDE. If you spent years in Eclipse or IntelliJ, VS Code feels inside out at first: the editor is small and the intelligence is bolted on through extensions you choose. A "project" is just a folder you open, not a ceremony you create. And the Python interpreter is selected per folder, which, once you see it, is the same idea as picking a JDK per project. That last point is the next section.

## Point VS Code at your virtual environment

This is the step the book doesn't cover, and it's the important one.

Open one project folder. In the last post we made `~/projects/pcc/python_work` with its own `.venv`, so open that:

```bash
code ~/projects/pcc/python_work
```

Open one project folder at a time, not the `~/projects/pcc` container, so VS Code has a single environment to reason about. VS Code opens that folder as your workspace, and because it contains a `.venv`, the Python extension will usually detect it and offer to use it. To be sure, or to set it yourself, open the Command Palette with Command-Shift-P, type `Python: Select Interpreter`, and choose the one that shows `('.venv')` in its name. That tells VS Code to use the isolated Python you built for this project, not some other one on the system.

You'll know it worked in two places. The bottom status bar shows the selected interpreter and its version. And when you open VS Code's integrated terminal (Terminal menu, then New Terminal), the prompt shows `(.venv)` because VS Code activated the environment for you. That second part is the quiet payoff: any `pip install` you run in that terminal lands in the project's environment automatically, with nothing to remember.

Make a quick file to prove the whole chain works. Create a new file (Command-N), save it as `hello_world.py` in the project, and type the same line the book starts with:

```python
print("Hello Python world!")
```

Run it with **Run ▶ Run Without Debugging**, or press Control-F5, exactly as the book describes. The output appears in the integrated terminal at the bottom, run by your project's Python. That's the book's first program, running on the foundation we built.

## Let the editor format your code for you

The book doesn't mention code formatting, which is understandable for a first program but a gap worth closing early, because letting a tool handle formatting means you never argue with yourself about spacing again. The modern tool for this is Ruff. It's a single, very fast tool that both formats your code and catches a wide class of mistakes, replacing the three or four separate tools people used to wire together.

Install its extension the same way you installed the Python one: open Extensions, search for `Ruff`, and install the official one, whose id is `charliermarsh.ruff` (Ruff and this extension are made by Astral).

Then tell VS Code to use it for Python files and to format every time you save. Because you'll open a different project folder for each part of the book, put this in your User settings so it applies everywhere instead of repeating it in each project. Open the Command Palette with Command-Shift-P and run `Preferences: Open User Settings (JSON)`. If that file is empty or just `{}`, replace its contents with the block below. If it already has settings, paste only the `"[python]"` entry (everything between the outer braces) inside the existing braces, with a comma after the previous entry:

```json
{
  "[python]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    },
    "editor.defaultFormatter": "charliermarsh.ruff"
  }
}
```

Because this lives in your User settings, it applies to every project folder you open. From now on, every time you save a Python file, Ruff tidies the formatting, applies safe fixes, and sorts your imports. You write roughly, save, and it comes out clean. For someone relearning the language, that's one less thing to hold in your head while you focus on the actual code.

## A couple of things worth grabbing from Appendix B

The book's Appendix B has genuinely useful VS Code tips that are easy to miss. Two are worth doing now.

Back in the first post I mentioned the book's own admission that running a program shows "some additional output showing the Python interpreter that was used." Appendix B is where it tells you how to quiet that down, by creating a `launch.json` and switching the console from `integratedTerminal` to `internalConsole` so you see only your program's output. It's a reasonable tweak, with one catch the book notes: the internal console can't accept keyboard input, so once you reach the chapters that use `input()`, you'll want the integrated terminal back. My advice is to leave the default alone while you're learning. The extra line of output is harmless, and the integrated terminal is the one that always works.

The other is the keyboard shortcuts, which are worth skimming in Appendix B: commenting a block with Command-/, indenting with Command-], and moving lines with Option and the arrow keys will cover most of what you do early on. Learn those three and add more as you notice yourself repeating an action.

## How to keep following the book from here

You're now fully set up to work through *Python Crash Course* in VS Code, with a few things quietly better than the book assumes.

When the book says to run a program with Control-F5, do exactly that. The only difference is that your program runs inside the project's virtual environment instead of a single global Python, which is what you want. When you reach the chapters that install libraries, matplotlib, Django, pygame, open that project's folder in VS Code, then open its integrated terminal and run the `pip install` there. Because VS Code already activated that project's environment, the libraries land in the right place and the editor immediately knows about them.

## What you've actually done

You installed a real editor through Homebrew, connected it to the per-project Python from the last post, and set it up to format your code for you every time you save. The book gets you VS Code and the Python extension; you've got those plus the wiring that makes them work with the environment you actually built.

That's the foundation finished: a clean machine, a Python you control, isolated projects, and an editor that understands all of it. Next we start saving your work properly, with git.
