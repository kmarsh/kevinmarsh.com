---
layout: post
title: "2026 Development Setup (AI Edition)"
date: 2026-02-19
draft: true
---

It's been awhile since I've posted but I thought I'd take some time to write about my set up for development in our new AI era.

First of all, I have mixed feelings about the whole thing, from the environmental to economical implications but setting that all aside I'm currently enjoying a ton of productivity with the latest crop of frontier models.

I'm using:

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code), the $100 Max plan
- [Zed](https://zed.dev)

## Claude Code

Claude Code is an agentic coding tool that runs in your terminal. You give it a task, it reads your codebase, makes edits, runs commands, and iterates until things work. It's not an autocomplete or a chatbot bolted onto an editor — it's more like pair programming with someone who can actually read and modify your files.

I went with the $100/month Max plan after burning through the Pro plan's limits in about two days. The Max plan gives you significantly more usage of the Opus model which is noticeably better for complex, multi-file tasks. It's not cheap, but I've already gotten my money's worth several times over.

Some things I've found it particularly good at:

- Scaffolding new features across multiple files (routes, models, tests, the whole stack)
- Writing tests for existing code — the kind of thing you always mean to do but never get around to
- Refactoring. Describe what you want the code to look like and it just... does it
- Explaining unfamiliar codebases. Point it at a repo and ask how something works
- The tedious stuff: migrations, config files, boilerplate

It's not perfect. Sometimes it goes off on a tangent or makes a change you didn't ask for. You still need to review everything it does (it's a tool, not a replacement for thinking.) But the feedback loop is fast — you describe what you want, it takes a shot, you course correct, and within a few iterations you're usually where you want to be.

I keep a `CLAUDE.md` file in the root of my projects with notes about the codebase, conventions, and things it should know. It reads this automatically and it makes a real difference in the quality of its output.

## Zed

I switched to [Zed](https://zed.dev) a little while back after years of bouncing between VS Code and various other editors. It's written in Rust, it's _fast_, and it has native support for AI assistants built right in.

Honestly the speed alone was worth the switch. After getting used to VS Code's Electron-based sluggishness, opening Zed feels like going from a Honda Civic to a sports car. Everything is instant: file switching, search, startup. It just doesn't get in the way.

The AI integration is nice (it has its own inline assistant and chat panel) but truthfully I use Claude Code in a terminal alongside Zed more than I use Zed's built-in AI features. Zed is my editor, Claude Code is my AI — and they complement each other well. I'll have Zed open for reading and navigating code while Claude Code handles the heavy lifting in a split terminal.

A few other things I like about Zed:

- Vim mode that actually works well (not an afterthought like in most editors)
- Multi-buffer search and replace
- Collaborative editing built in, though I haven't used it much yet
- Extensions are growing but it's still early days — if you depend on a specific VS Code extension, check first

## Real World Uses

Enough about the tools, here's what I've actually been _doing_ with them.

### Migrating a Test Suite from Minitest to RSpec

I had a Rails project with a full Minitest suite that I wanted to move to RSpec. This is the kind of task that's straightforward but incredibly tedious — every file needs to be rewritten, the assertions need to be translated, the setup/teardown blocks need to become before/after hooks, factories need to be wired up, and on and on. It's the kind of thing you'd put off for months (or forever) because the codebase works fine, you just don't _love_ it.

With Claude Code I pointed it at a test file, told it to convert it to RSpec following the project's conventions, reviewed the output, ran the tests, and moved on to the next one. File by file, it churned through the whole suite. I still had to fix things here and there but what would have been a miserable multi-day slog turned into an afternoon.

### Switching to Tailwind

Similar story. I had a project using a hodgepodge of custom CSS and wanted to move to Tailwind. Again, not _hard_ work, just tedious work — the kind of thing where the effort-to-reward ratio never quite tips in favor of actually doing it. Having an AI that can look at a component, read the existing styles, and rewrite the markup with Tailwind classes made it realistic to actually pull off.

### Polishing Shell Scripts

This one surprised me. I have a bunch of shell scripts that have accumulated over the years — deployment scripts, data processing pipelines, little utilities. They work, but they're held together with duct tape. No error handling, no help text, hardcoded paths, the usual.

I've been having Claude Code go through them and add the kind of polish I'd never bother with on my own: proper argument parsing, usage messages, error handling, colored output, that kind of thing. Each script takes a few minutes to clean up and the result is something I'm actually not embarrassed to share. It's like finally cleaning out the garage because someone offered to help carry the heavy stuff.

### Getting Unstuck as a Solo Developer

This is the big one for me. As a solo developer you don't have anyone to bounce ideas off of. You hit a wall and you're just... stuck. You can Google around, read Stack Overflow, dig through documentation, but there's no one to say "hey, have you tried X?" or "I think the issue is in Y."

Claude Code fills that gap surprisingly well. Not perfectly — it doesn't know your codebase the way a long-time colleague would — but it's available at 2am on a Sunday when you're stuck on a weird bug and just need another set of eyes. I've had multiple occasions where I described a problem, it pointed me in the right direction, and I was unstuck in minutes instead of hours.

### Mosquito Tasks and Shipping Momentum

This might be my favorite trick. Some mornings I sit down and ask Claude Code: "look at this codebase and find a small improvement we can make." Maybe it's a missing index on a database column, a method that could be simplified, a test that's been skipped and forgotten about, or a deprecation warning that should be addressed.

These tiny tasks — I've been calling them mosquito tasks — take five or ten minutes each but they build real momentum. You start the day with a commit. Then another. Before you know it you're in the zone and tackling the bigger stuff. Shipping momentum is real and having a tool that can surface these little wins on demand has been a game changer for my productivity, especially on days when I'm not sure where to start.

## The Workflow

My typical workflow these days looks something like this:

1. Open the project in Zed
2. Fire up Claude Code in the integrated terminal (or a separate terminal, depending on the task)
3. Describe what I want to build or fix in natural language
4. Review the changes in Zed as they happen
5. Test, iterate, commit

It's surprisingly smooth. The thing that took me a while to internalize is that you get the best results when you treat the AI like a junior developer who's very fast but needs clear direction. Vague prompts get vague results. Specific prompts with context get surprisingly good code.

## What's Next

I'm still figuring this stuff out, honestly. The tools are changing faster than I can keep up with and what works today might be obsolete in a few months. But for now, this setup — Claude Code in the terminal, Zed as my editor — has been the most productive I've been in a long time. And that's saying something, coming from someone who's been doing this for over 20 years and is generally skeptical of hype cycles.
