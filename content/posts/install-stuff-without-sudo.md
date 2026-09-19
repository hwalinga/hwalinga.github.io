---
title: "Homebrew version 7 release"
description: "And more ways to install stuff without sudo"
summary: "Examples of how to get stuff on your PC without using sudo"
tags: ["devops"]
date: 2026-09-17
draft: true
showToc: true
TocOpen: true
math: true
lightCode: true
---

This shows some ways I like to get stuff on my PC, or the HPC environment (where I do not have sudo).

## TLDR

Install stuff easy without sudo:

```sh {style=github}
bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" \
  -- --path ~/.hbrew
brew install bcftools
```

## First

Of course a lot of language package manager will support installing stuff as well 
However most of these are tailored towards you developing a project in that language. 
It can be useful to have some isolation in runtime/buildtime.
Some of the language tooling already have some options:
(Node: `npm install -g`, Python: `pipx`/`uvx`).
And of course this is less of a problem of things that just provide binaries (Go (`go build`) / Rust (`cargo install`)).
Now the problems start with tools in C/C++, or other complex setups beyond the language itself.
Those can just be weird.

## `brew install`

A bit of history: Homebrew used to be my go to for whenever I wanted to install something not in my system package manager,
and also especially when to install something on a HPC where I do not have sudo. 
However, homebrew is designed for usage on a personal Mac. 
That is works on Linux without sudo is not its prime target.

I even went ahead to [fix](https://github.com/Homebrew/install/pulls?q=hwalinga) this behavior when it broke.

Now at some point, the installer got rid of this feature and installing in a custom prefix got (even more) unsupported.
You could still manually tar it in a place you had access to, which was fine, but not great.

But with the latest release of homebrew (7.0.0), we got this [back again!](https://brew.sh/2026/09/13/homebrew-7.0.0/#-non-default-prefix-users)
And to support this the installer added a (somewhat undocumented) `--path` to do this:

```sh
bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" \
  -- --path ~/.hbrew
```

Now what I liked most about brew was that it was hackable in the sense that I could `brew edit <FORMULA>` to change any installation script.
It also included a way to install from the `--head` source to get the latest github release of some software.
This is of course also something you can hack in with the `brew edit`.[^5]
And finally for me personally the `brew tap brewsci/bio` had very broad selection of software useful for my bioinformatics needs.

[^5]: NB. Keep in mind that for safetey there is checksum checking which you do not have if you install from the "head".

## `pixi global install`

Some time ago a new package manager appeared and it was pixi.
Pixi is a great replacement of conda for installing conda packages.
(Conda itself is not so great (anymore)[^6] but the conda ecosystem is still definitely worth it.)
Pixi does some very interesting other stuff which I leave to you to figure out, 
but one thing to mention is that it can also install packages globally,
exposing only the specific commands, 
and by using shims keeping the environment of that package isolated to only those commands.
(They call this [trampolines](https://pixi.prefix.dev/latest/global_tools/trampolines/) internally.)

[^6]: Not just its speed and cumbersome usage, but it also not so free anymore

You can still manage that environment even after the fact!

```sh
pixi global install --environment jupyter notebook \
    --with polars --expose jupyter --expose jupyter-notebook
pixi global add --environment jupyter pyarrow altair
jupyter notebook  # no need to activate environments, 
# by running the exposed `jupyter` it activates it automatically internally
```

One unfortunate thing is that this cannot [circumvent any channel priority](https://github.com/prefix-dev/pixi/issues/3688) 
issues you sometimes have with conda.

For more pixi see my slides [here](/slides/pixi-presentation/)

## `mise use -g`

Final shootout to `mise` which is more then just install software, 
but that is for you to figure out.[^7]
I used this for some things that were not in brew and it worked great:

```sh
mise use -g opencode
```

[^7]: Mise can for example automatically activate your environment, but I just `direnv` for that.

## (Another) system package manager

There is also of course your system package manager.
This is I would say often a preferred way to install.
It is the intended way after all.
However, it is often less complete or old versions.
And of course, you need sudo for that.

Now there are some system package managers,
which you can use even if you are not on that system.
These might be worth exploring.

Arch provides a very comprehensive one where the AUR fills every need you ever would have.
It would be great to have that on every system and some projects promise this.
Unfortunately, in my experience they often are limited as they create a certain isolation from your own system with the separate system.

The same can be said for the nix package manager.
I have no experience with this one.
