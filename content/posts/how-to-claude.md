---
title: "How do I Claude"
summary: "Example of my Claude workflow"
tags: ["AI"]
date: 2026-09-06
draft: false
showToc: true
TocOpen: true
math: true
lightCode: true
---

This shows how I currently use Claude Code, or simply CC.

First I explain my setup, then I go over I usually use it with a feature implementation.

## Setup

### Sandboxing

```sh
fclaude
```

Instead of just `claude` directly, I sandbox `claude` with [fence](https://github.com/fencesandbox/fence). `fclaude` is simply aliased:

```sh
alias fclaude
fclaude='fence -- claude --permission-mode auto'
```

I might later share my fence configuration on my [dotfiles repo](https://github.com/hwalinga/dotfiles/).

It looks a lot like the [Anthropic one](https://github.com/anthropics/sandbox-runtime) by the way, but there are some reasons I liked `fence` better, not in the first place to prevent the agent and the sandbox be from the same source.

The sandbox limits my CC instance in what files it can see and what network access it has.
As either AI as Anthropic cannot be trusted I think that is a good idea.

### Fearless auto mode

This sandboxing also makes me lose no sleep in always using auto mode. (As shown by `--permission-mode auto` in the alias.)
I think auto mode is superior in that AI can now much better gather the right context in running commands to test its hypothesis 
and I do not want to manually approve everything. 
Also I think the CC guardrails are horrible and make CC do weird stuff (hacking around it with docker for example).
Even though it can be useful, I think OS level limitations is always required.

### plan mode

On the topic of modes, I am a CLI kind of guy and like control, 
so I actually do not use the plan mode. I am always on auto mode. However I have custom commands for custom tasks.

This is my `~/.claude/commands/planner.md` command file: 

```
Plan $ARGUMENTS

Put the plan in the file PLAN.md and number different steps.

If PLAN.md already exists:
- Read it first.
- If it is for a different task (even a related one), move it to `.claude/plans/` with a nice name (keep the file, do not delete it) before writing the new plan.

Do not edit or write any file other than PLAN.md during this planning phase. This is read-only exploration and planning.

In this plan mode, you should:
1. Thoroughly explore the codebase to understand existing patterns
2. Identify similar features and architectural approaches. Actively search for existing functions, utilities, and patterns that can be reused — avoid proposing new code when suitable implementations already exist
3. Consider multiple approaches and their trade-offs
4. Use AskUserQuestion if you need to clarify the approach. Do not make large assumptions about user intent — tie up loose ends before the plan is done
5. Design a concrete implementation strategy

The finished plan should contain:
- A clear summary of the approach
- An ordered list of files to create/modify, with specific changes
- Step-by-step implementation order (numbered)
- Testing and verification steps
- Potential risks and mitigations
```

I prefer these over skills but they are very much the same. 
The difference is that skills can trigger whenever, but commands have to be mindfully triggered. 
Skills can just as easy be used as commands, and commands are even considered deprecated!
But as I like control, I obviously prefer commands over skills as long as that is supported. (But I also use skills when I think that is a good use.)

## System prompts

This kind of working has a downside. 
CC is filled with system prompt that trigger based on what it is doing. 
This is what makes agents tick and what makes them good. 
It is a whole collection of specific prompts that trigger on the right time. 
What is more, the LLM behing CC is trained on CC system prompts. 

(This coupling between agent and LLM is also why mix and match LLM with agent does not always give you reliable results.
And yes you can with [openrouter](openrouter.ai) use any LLM behind CC you desire. [^openrouter])

[^openrouter]: Openrouter has some pretty interesting features and you can also bring your own token(BYOK). Unfortunately, Claude plans do not include any way to get an API key. API keys are only available on pay-per-token basis.

And that is the rub with using custom instruction too much, 
you are fighting its intended usage and that only goes badly.

Luckily the CC system prompts are [available on the internet](https://github.com/Piebald-AI/claude-code-system-prompts) 
and that is what I have used to build my `planner` command.

### The model

As for the model, I usually stick to Sonnet for most tasks, 
but occasionally if I need some more power I take Opus. 
As you might know Opus 5 speaks horrible,[^opustic] so I usually use Opus 4.6 for this.
Put this in your `~/.bashrc` / `~/.zshrc` to get Opus 4.6 in your CC:

[^opustic]: See some opustic behavior on [here](https://opusfived.dev).

```sh
export ANTHROPIC_CUSTOM_MODEL_OPTION="claude-opus-4-6"
export ANTHROPIC_CUSTOM_MODEL_OPTION_NAME="Opus 4.6"
export ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION="Opus 4.6"
```

As CC is pretty vibecoded, you cannot have more custom (old) models available beyond this.

You can use `ALT+P` to change model in your CC. The latest used model is the default.

There is also effort you can edit on this menu, but I always have it on `high`.
I have read that if you want to reduce the AI speak, setting this lower can help.

### CLAUDE.md

The `CLAUDE.md` gives additional context to a prompt beyond the system prompts. 

There is the global one `~/.claude/CLAUDE.md` and the project specific one.

#### Global CLAUDE.md

As this one ends up everywhere, I try to keep it short: 

```md
Try to stick to ASD-STE100 Simplified Technical English, keep answers short and to the point.

Always run `pytest -n auto`, never bare `pytest`.

Please in every final response, address me with my name, Hielke.
```

That final line is there as a good way to see if there is too much context and CC is forgetting instructions.

#### Project CLAUDE.md

This should set the basics for working in that project. You can get a basic one with `claude init`.

It is a good idea to put this on version control and share with everybody in the project.

If you have other people not using Claude but something else you can have `@AGENTS.md` in `CLAUDE.md` instead.[^agents]

[^agents]: Now it seems that in the latest release you can configure this.

Personally, this would be more like a project index. As for most things, other developer operations should be automated as much as possible!

For example, do not instruct CC how to activate the environment, but just use something as [direnv](https://github.com/direnv/direnv) to always do that.

Use `CLAUDE.local.md` if you really have something environment specific you do not want to bother other people with.

### The status line

I stole the initial status from the openrouter example to show openrouter usage.
Now that unfortunately did not work very well. (I think it just updated too slow and CC give it a timeout.)

But I did edit to show Claude usage and context size which I think is just as useful!

You can take a look from my [dotfiles](github.com/hwalinga/dotfiles) for `statusline.*`.

# Example

I will now show a (simple) real world feature implementation

## The plan

With the `/planner` command discussed in the setup I make a plan. 
Usually the Gitlab issue text is the starting point.

Then when the plan is made, you have to read it. 
I am a terminal junkie so I have a terminal markdown viewer for this. 
I am using [leaf](https://github.com/RivoLink/leaf) for this.

### After planning

After reading the plan you can edit it in places where you think it was wrong.
Claude really seems to like doing a bit more then instructed.
This is usually not a bad thing, but you have to keep this in mind.

The plan can sometimes be a lot of slop.
Usually the decisions and risks parts is the most interesting stuff.

Also the test strategy has to adjusted sometimes.
Testing strategy is quite a tailored one for the project needs.
And with AI testing can sometimes be too ridiculous and you do not want those kinds of testing in the codebase.

I am not always editing the plan,
but when I do, it can be helpful to ask the AI to make that edit, instead of doing it manually.
That can make the edit land better, or you get some ideas from the AI as well if your edit is correct anyways.

## Execution

Now we can execute the plan. I have a command again for that.

```sh
/execute
```

It is pretty simple one, haven't really found the need to extent.

```md
Perform the plan in @PLAN.md

$ARGUMENTS
```

The `@` symbol before a filename will immediately insert the file contents in the context.

### Keeping a small context

To save tokens, this can be a good point to new session with `/clear` after planning and before executing. 
You have all the important stuff in `PLAN.md` nonetheless.
But it might be useful to have the planning context still available.
You can also use `/compact` to compact the context instead. (It will summarize the context.)

Keeping your context small and clean helps in efficiency and accuracy. Big context hallucinate and take longer to process.

Nowadays I usually run `/compact`. I do not have so much problems with not having enough tokens in even the cheapest plan.

I can also be helpful to execute each section in a separate context, 
but without too much guidance the AI sometimes has trouble implementing half features.

## The waiting game

Now, you have to wait in-between the prompts. You can do something else, 
but that what is flashing on the screen can be helpful to read. 
To already get a feel for what it is doing, or to stop it `ESC` when it goes off the rails.
So I usually split the terminal (`tmux`) and read hackernews next to it.[^5]

[^5]: From [ggerganov/hnterm](https://github.com/ggerganov/hnterm) with my fork [hwalinga/hnterm](https://github.com/hwalinga/hnterm) with light theme support

### Notification

Of course you can also do something else, but to not get too distracted I want to be notified when it is done. 
So I have a notification script for that you can edit in the settings.

Configuring notification as follows in `~/.claude/settings.json`:

```js
  "hooks": {
    "PermissionRequest": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "claude-noti.sh Claude needs permission",
            "async": true
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "claude-noti.sh Claude has finish working",
            "async": true
          }
        ]
      }
    ],
    "PostCompact": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "claude-noti.sh Claude has compacted the conversation",
            "async": true
          }
        ]
      }
    ]
  },
```

And it then runs this notification script: 

```sh
#!/usr/bin/env bash
# claude_notify.sh
# Reads a Claude tool-use JSON event from stdin and pops a yad notification.
# Usage: echo '<json>' | ./claude_notify.sh "Claude has finished working"

set -euo pipefail

prefix="$*"

msg=$(jq -r --arg prefix "$prefix" '
  (.tool_input.command // .tool_input.file_path // "") as $arg
  | if .tool_name then
      "\($prefix): \(.tool_name)" + (if $arg == "" then "" else "(\($arg))" end)
    else
      $prefix
    end
') || msg="$prefix"

[[ -n $msg ]] || msg="$prefix"

yad \
  --text="$msg" \
  --title="Claude" \
  --text-align=center \
```

This uses yad, but you can of course configure it send you a notification via any possible means you want.

### The human verification

Now it is time to read the code. I consider this much more important (and also interesting) then reading the plan.
In the terminal I have [diff-so-fancy](https://github.com/so-fancy/diff-so-fancy) to have a nicer diff. 
I got used to that many years ago, and I believe better (terminal) diff viewers exists nowadays.
It can also be helpful to read the code in the Gitlab diff viewer, just to have a different context than a terminal.

#### The comments

AI can be very annoying with the comments. It can have very verbose comments. It can also remove existing helpful comments. It can also comment on a previous situation you did not choose. Why comment about a worse solution not chosen? This is usually where there is still a lot of AI sanitization is required. Especially because the comments are there for your team, and you know your team mates much better then the AI.

In this case that was also the case:

```
Versions are linked through `direct_source` (each points to its
immediate predecessor) rather than a shared identification code —
every version now has its own code.
```

### Committing

I also have a `/commit` command, which in turn sets Claude as a co-author. This is basically Anthropic advertising. (That is also why they do not support `AGENTS.md`, the `CLAUDE.md` in the repo is just a free add!) But I think the commit is a good thing to communicate this code had AI as an assistant in writing it.

```md
Create a conventional commit for the staged changes.

1. Analyze what changed
2. Determine the commit type (feat/fix/docs/refactor/test)
3. Keep in mind that feat and fix end up to the CHANGELOG, if it just a small fix/change that does not need to be communicated, just use oops or chore.
4. Write a clear, concise message
5. Include scope if applicable
6. Keep in mind that with pre-commit, sometimes the code is automatically fixed. Just run the git command again, and then it passes.

$ARGUMENTS
```

# Conclusion

I do not know if this was much faster than doing it manually. I am also still pretty close to all the code (not really vibing it). 
My current stance on AI assistance is that it can help me learn more about the codebase which I might miss when doing it manually.
(Allthough I do learn different things when going manually through the code, like actually remembering path names and so, and assessing whether changing features is still easy enough.)

What mostly helps a lot is the AI assistant coding increases the quality and consistency of the code.
Often the AI points out some other things I might have not considered when doing it myself.
I am not yet fully convinced on the immediate speed increase of it.
The one thing I think that it really speeds up is making a lot of bad code.
Making quality code is still a laborious process.

# There is more

After committing all the stuff happening on Gitlab I still do manually, but that is something you can also give CC access to.

Some other things worth talking about, but I left out of this blog post:

 - More plugins (I do not use that much, I have installed official code-simplifier, context7, and playwright)
 - Use CC to review other code. (Still requires heavy filtering, AI can be really nitpicky.)
 - Memory management (Let your agents `/dream` occasionally)
 - Git worktrees (I had my pros and cons)
 - There are more agent harnesses, and I do believe CC is at the moment the best including the LLM harness parity, but this can of course change over time.
