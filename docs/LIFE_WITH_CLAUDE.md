# How to Work Effectively with Claude Code

## Introduction

This text summarizes my personal experiences to develop effectively
with Claude Code and similar AI coding assistants. I have found that
efficient application of agentic coding can multiply your productivity,
especially when working with content or programming languages at the periphery
of your expertise.

Like any tool, agentic coding requires practice and effective techniques to
get good results. This becomes increasingly important the more complex the
task is.

The following sections outline my current workflow and best practices for
working with Claude Code. My experience with these tools and workflows is
still limited and evolving, so take everything with a grain of salt!

## Overview and General Guidelines

Language models are still essentially automata that provide output that sounds
plausible, given any input prompt. They have layers built on top of this foundation
to help them reason, plan, and execute tasks more effectively. However, they
still have no life goals and suffer from a short attention span. As a human developer,
it is your job to give them those life goals and keep them on track.

Language models excel at, well, working with language and verbal instructions.
They can infer a lot from even vague prompts, so working with them is fairly
effortless. You just need to keep in mind that they are still just tools
that need to be guided effectively. Luckily, regarding software engineering,
the same principles and methodologies that work for human developers also apply
to AI assistants.

For effective outcomes, provide them clear objectives. Define specific,
achievable goals for each coding session. Break down tasks into manageable
chunks to maintain focus. Regularly assess progress and adjust plans as needed.

Importantly, no matter what you do, Claude won't follow your rules. That's unavoidable and a law of nature when working with LLMs. No matter how clearly you set rules and processes, how much you make **MANDATORY** remarks, after a short while, Claude will happily start doing their own thing. You just need to constantly remind them of the rules and processes.

When Claude does stupid things, stop them and tell them what to do instead. Getting irate or scolding them does not help -- they're just a tool, and it won't make you feel better, either. Instead, provide them constructive instructions on what to do differently. Think of it as a patience-building exercise! Based on what I have read, instructing in negative terms (what *not* to do) is a bit like asking someone to *not* think of a pink elephant. It's not very effective and you should focus on the alternatives instead.

## Maintaining Context: Workspaces and Context Files

When working with Claude Code, you need to ensure they have access to
the full context of the project. Set up a workspace repository that contains
all relevant documentation and individual code repositories. Workspaces let
Claude Code easily navigate between different parts of the project.
I personally find submodules to be more hassle than they are worth, so I typically have a
`run` script that consolidates all common operations within the workspace,
including cloning the individual sub-repositories.

Agentic coding tools typically support special files that they automatically
read into their context (short-term memory) when starting a conversation.
For Claude Code, this file is `CLAUDE.md`. Most other tools are gravitating
towards using `AGENTS.md` for this purpose. To support either, I typically
have all relevant documentation in `AGENTS.md` and then have `CLAUDE.md`
simply point to it using its special include syntax: `@AGENTS.md`.

My preference is to use `AGENTS.md` mostly as a table of contents that points
to other documentation files. I do include certain critical policies and guidelines
directly in `AGENTS.md`, but mostly explain where to find the relevant information.
This is based on my personal preference, though -- it might well be that providing
much more direct information in `AGENTS.md` works better. In any case, use
Claude Code itself to edit and maintain the `AGENTS.md` file!

## Getting Started with an Existing Project

To get started with an existing project, create a new directory (your workspace)
and clone every related code repository under that directory. You might want to
create a `README.md` file that explains the overall project purpose if that is
not obvious from the individual repositories. Then, run `claude` or open
the workspace directory in Visual Studio Code and start Claude Code from there,
using an appropriate plugin. Give the command `/init` to have Claude Code browse
through the entire workspace and create a `CLAUDE.md` file that summarizes the
entire context for you. That's it! (If you plan to use other tools, rename
`CLAUDE.md` to `AGENTS.md` and add the link to it in `CLAUDE.md`.)

This is already sufficient to get started. However, for anything but simplest
tasks, I recommend adding additional documentation and guidance that helps
Claude Code understand your expectations and preferred workflows. These
are described in the following sections.

## Starting a New Project

When starting a new project, I like to follow a structured workflow. For a
small to mid-size one-person project of my own, I probably wouldn't care much
about such formalities. However, when working with Agentic tools, this approach
becomes vital for ensuring you get intended results.

The project structure should follow the same workspace approach as described
above. (Of course, if you want to use an actual monorepo, no-one is stopping you!)

Create a `docs/` directory in the workspace root. This is where all
documentation files go. Copy the policy and workflow documents from another repo such as [halos-distro](https://github.com/hatlabs/halos-distro) to get started quickly.

You should at least have the following files:
- [PROJECT_PLANNING_GUIDE.md](PROJECT_PLANNING_GUIDE.md)
- [DEVELOPMENT_WORKFLOW.md](DEVELOPMENT_WORKFLOW.md)
- [IMPLEMENTATION_CHECKLIST.md](IMPLEMENTATION_CHECKLIST.md)
- [LIFE_WITH_CLAUDE.md](LIFE_WITH_CLAUDE.md) (this document)

It is also a good idea to write a README.md that explains the overall project purpose and what you intend to do. Try to give as much information on the project goals as possible. Don't worry about the format or structure, though. Claude will understand and help you improve it later. Spend some time at this point, though - the more context you provide, the better the results will be later, and since Claude isn't
a mind-reader, you need to provide the kickstart context yourself.

## Project Planning Workflow

At this point, you are ready to start! Launch Claude Code in the workspace
and tell it something like this:

> I want to create a new project called X. Read the README.md and other documentation files in the docs directory to understand the overall goals, context and workflow expectations. Then, start following the planning workflow as described in PROJECT_PLANNING_GUIDE.md. Explore and ultrathink. Ask the user clarifying questions as needed. Stop after each step to get user feedback and approval before proceeding.

If you want, you can also provide existing projects as inspiration or reference:

> Here are some existing projects that are similar to what I want to do: [link1], [link2]. Study their structure/implementation to get ideas.

That "ultrathink" directive is important! It tells Claude Code to take its time
and really consider the problem deeply before rushing into action.

Once started, the project planning will start rolling on its own. Keep feeding
Claude Code with feedback and clarifications as needed. The first stages of the
process are crucial as they define the contents of all subsequent steps. Don't
fall into analysis paralysis, though! You can always request changes later and
Claude will be happy to plan any necessary refactorings.

During the process, you will create SPEC.md (specification) that defines what
you want to build, ARCHITECTURE.md that describes how the system will be
structured, and TASKS.md that breaks down the implementation into manageable
tasks. One important thing when reviewing TASKS.md is to ensure that CI testing is planned early on.

Already when creating TASKS.md, you might want to remind Claude to adhere
to DEVELOPMENT_WORKFLOW.md.

As an end result of the planning process, you should have a set of GitHub issues
that you can start working on as well as an initial repository structure, including
your core design documents.

## Development Workflow

Once you have your project planned out and issues created, you can start working on the implementation. The process is similar to the planning workflow, but on a smaller scale. You give Claude a specific issue to work on. Use phrases like "Start working on my-project issue #4. Remember to follow the DEVELOPMENT_WORKFLOW.md carefully. Ask questions if needed."

If you're not sure what issue should be picked next, you can always ask. You can even ask complex questions such as "Have a look at the open issues in all sub-repos in this workspace. Based on their mutual dependencies and priorities, create a recommended implementation order list. Explain your reasoning."

## Working on Individual Issues

Now, Claude will follow the DEVELOPMENT_WORKFLOW.md to implement the given task. Or that's the idea, at least! In practice, here's where the hand-holding starts. You will be herding the proverbial pack of kittens that is Claude getting excited about the latest shiny object it sees. Pay close attention to the progress. If you think something is going wrong, don't hesitate to smash the Esc key and give corrective instructions or even reminders. "Did you create the tests yet?" is a common one.

If you let Claude work on their own, they might end up with sloppy solutions, but when working interactively, Claude is usually smarter than you think. You can ask them to evaluate alternative approaches, take inspiration from existing code, and so on. Just remember that they are the sociopathic sycophant as all LLMs are: if you want to present them an idea, ask them to compare and to find problems with it. Otherwise they'll happily say "You're absolutely right!" to anything you say.

When the implementation is done, ask Claude to verify that everything is working as intended. You can even have them create a checklist of verification steps based on the original SPEC.md and ARCHITECTURE.md documents. Once verified, have them create a PR.

There is one more crucial step: code review! Always have Claude review their own PRs before merging. (Even if you are not using GitHub, perform a code review!) My own practice is to run a second Claude in a separate terminal window and asking them to review the PR created by the first Claude and to post the comments directly to the PR. You can also instruct the reviewer to compare the PR changes against the original SPEC.md and ARCHITECTURE.md documents to ensure everything is in line. Invariably, there are issues to be fixed, so have the original implementer Claude address those comments. For really critical code, you could, shock and horror, even perform a manual code review yourself. Usually this takes a couple of iterations until everything is OK.

## Plan and Models to Use

The increased productivity, unfortunately, comes at a cost. The regular USD 20 /month Claude Pro plan is good for maybe one or two hours of daily development work, and it will also consume the weekly quota in no time. It is still usable for occasional use, though. The USD 100 /month Claude Max plan is sufficient for full-time development work, as long as you don't try to run multiple sessions in parallel. It is also possible to pay for extra usage if needed, so if you only need a little more than what Claude Pro offers, you don't need to jump to Max immediately.

The output quality also depends on the LLM model used. Anthropic's Claude Sonnet 4.5 is a capable model. I have not found much improvement when using their best model, Claude Opus 4.1. Opus is also five times as expensive.

Anthropic also has a faster and cheaper model called Claude Haiku 4.5. It can be used for straightforward and well-defined implementation tasks, and the speed increase is phenomenal, but it will require a lot more herding and do a lot of annoying mistakes.

## Holding on to the Leash: Modes

Shift-tab or the VSCode window drop-down menu can be used to change the mode Claude code is operating in. By default, user permission is asked for each action. This gets tedious quickly, so there is also "accept edits" mode that lets things fly by much faster. It still asks for permission for actions deemed risky by some unknown criteria. If you want to live dangerously, and I often do, you can start Claude on the command line with `claude --allow-dangerously-skip-permissions` to have them operate in full auto-pilot mode. Use with caution.

One important mode to remember is the planning mode. In planning mode, Claude won't execute any commands or make any changes. This is extremely useful when you want to discuss a plan until you're satisfied with the outcome.

## Managing Available Context Window

All LLM tools have a limited context window ("short-term memory"). For efficient use, you may want to manage the context window actively. For example, at the beginning of each new issue, you can `/clear` the context window to essentially give Claude a full amnesia.

## Access to Local Tools and Hardware

If something doesn't work, you can ask Claude to debug the problem remotely. If you have SSH keys set up, Claude can happily SSH into your development server and test and debug the code there or deploy updates bypassing the CI/CD. In the same manner, it should be possible to provide them access to embedded systems debugging interfaces and let them flash and test code on their own. Don't let them near your production systems, though! They will do stupid things there. Or at the very least, verify each and every command they suggest before executing it.

To provide required information for Claude, I usually create a gitignored `CLAUDE.local.md` file (think of `.env` files) that contains any information specific to my local setup. To ensure other agents can also use it, I include a pointer to it in the AGENTS.md file, usually in Claude's special syntax: `For local environment information, see @CLAUDE.local.md`.

If you are already paying for GitHub Copilot, it can be somewhat effectively used alongside Claude Code. It even supports Claude Sonnet 4.5. In my experience, it needs more guidance in using the documentation and is not quite as capable in independent tasks, though.

## Access to Additional Resources: MCP

Model Context Protocol (MCP) is a way to provide additional context and resources to language models. Different MCP providers can be used to give Claude access to AWS, databases or other external resources. Often, though, command line interfaces are equally effective and easier to control.

I have no strong opinion on MCPs at this point. I do have [ck-search](https://github.com/BeaconBay/ck) set up and Claude seems to be using it quite frequently. YMMV.

## Providing Guardrails: Hooks

Hooks are user-defined shell commands that can be set to execute at specific points during Claude's operation. They can, for example, be used to validate commands before execution. I have not used them much, except for one thing: `git add -A` is something Claude likes to do, and it pulls in unwanted cruft. I asked Claude to create a hook to forbid that command, and voila! Problem solved.

## Git Pre-commit Hooks with Lefthook

To catch formatting and linting issues before they reach CI, HaLOS repositories use [lefthook](https://github.com/evilmartians/lefthook) for git pre-commit hooks. This ensures CI checks don't fail due to formatting issues that should have been caught locally.

**One-time Setup:**

```bash
# Install lefthook
brew install lefthook
```

**Per-repository Setup:**

After cloning any HaLOS repository that has a `lefthook.yml`, install the hooks:

```bash
./run hooks-install
```

This enables pre-commit hooks that run format and lint checks matching what CI runs. To skip hooks when needed (e.g., WIP commits):

```bash
git commit --no-verify -m "WIP: message"
```

## References

- https://www.anthropic.com/engineering/claude-code-best-practices
- https://docs.claude.com/en/docs/about-claude/models/overview
- https://www.claude.com/pricing
