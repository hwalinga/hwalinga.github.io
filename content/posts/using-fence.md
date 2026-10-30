---
title: "Another AI incident: Protect yourself with a sandbox"
description: "Using Fence for a sandbox around your AI"
summary: "Using Fence for a sandbox around your AI"
tags: ["test"]
date: 2026-10-30
draft: false
showToc: true
TocOpen: true
lightCode: true
comments: true
---

Today the official Claude Code extension for VS Code from the Cursor Marketplace get trojan malware. 
The same is true for the Claude Code extension for Cursor itself.

This is just an ordinary supply chain attack: Malware in developer dependencies.
However, I believe AI tooling is much more at risk of infecting your computer.
You give AI full access to your computer and it can run any software installed just following instructions from anywhere.
A prompt injection from a random skill file can also just as easily backdoor your computer!

And even that there is some weak mechanism in Claude Code for protecting you against this,
I believe you should just sandbox your computer. 
Just make any damage and leaked data impossible!
Put it in a box.

A sandbox limits running software by what it can read and write on your computer.
In addition you can limit what it can access on the internet.
This is a hard enforced boundary. 
The protection that is by default in Claude Code relies on weak detection which can be circumvented.
If you disable using curl for example, Claude Code could just write something with Python.
Also if you have docker installed, it can just do everything via docker.

Claude Code has an experimental sandbox available to use for you as well!
I have decided to use fence instead as if Anthropic gets compromised this sandbox might as well.
So getting this from two different providers feels a bit more safe.
And with the obscurity comes also some security, although it might not be as hardened.

I just set an alias:

```sh
alias claude="fence -- claude --permission-mode auto"
```

This is my fence config in `~/.config/fence/fence.json`:

```json
{
  "allowPty": true,
  "network": {
    "allowLocalBinding": true,
    "allowLocalOutbound": true,
    "allowLocalOutboundPorts": [8000, 3000, 5432],
    "allowedDomains": [
      // LLM API providers
      "api.openai.com",
      "*.anthropic.com",
      "*.claude.ai",
      "claude.ai",
      "claude.com",
      "*.claude.com",
      "claud.de",
      "*.claud.de",
      "api.githubcopilot.com",
      "generativelanguage.googleapis.com",
      "api.mistral.ai",
      "api.cohere.ai",
      "api.together.xyz",
      "openrouter.ai",
      "api.morphllm.com",
      "*.amazonaws.com",
      "*.readthedocs.io",

      // OpenCode
      "opencode.ai",
      "api.opencode.ai",

      // Amp
      "ampcode.com",
      "*.ampcode.com",

      // Factory CLI (droid)
      "*.factory.ai",
      "api.workos.com",

      // Cursor API
      "*.cursor.sh",

      // Charm
      "data.charm.land",
      "catwalk.charm.sh",

      // Copilot
      "*.githubcopilot.com",

      // Git hosting
      "github.com",
      "api.github.com",
      "raw.githubusercontent.com",
      "codeload.github.com",
      "objects.githubusercontent.com",
      "release-assets.githubusercontent.com",
      "gitlab.com",
      "docs.gitlab.com",

      // Package registries
      "registry.npmjs.org",
      "*.npmjs.org",
      "*.context7.com",
      "context7.com",
      "registry.yarnpkg.com",
      "pypi.org",
      "*.djangoproject.com",
      "djangoproject.com",
      "files.pythonhosted.org",
      "crates.io",
      "static.crates.io",
      "index.crates.io",
      "proxy.golang.org",
      "sum.golang.org",
      "formulae.brew.sh",
      "gohugo.io",

      // Model registry
      "models.dev"
    ],

    "deniedDomains": [
      // Cloud metadata APIs (prevent credential theft)
      "169.254.169.254",
      "metadata.google.internal",
      "instance-data.ec2.internal",

      // Telemetry (optional, can be removed if needed)
      "statsig.anthropic.com",
      "*.sentry.io"
    ]
  },

  "filesystem": {
    "allowWrite": [
      ".",
      // Local cache, needed by tools like `uv`
      "~/.cache/**",

      // Claude Code
      "~/.claude*",
      "~/.claude/**",

      // Amp
      "~/.amp",

      // Codex
      "~/.codex/**",

      // Cursor
      "~/.cursor/**",

      // OpenCode
      "~/.opencode/**",
      "~/.local/state/**",

      // Gemini CLI
      "~/.gemini/**",

      // Factory CLI (droid)
      "~/.factory/**",

      // Pi
      "~/.pi/**",

      // Copilot
      "~/.copilot/**",

      // Kiro CLI
      "~/.kiro/**",
      "~/Library/Application Support/kiro-cli/**",

      // Package manager caches
      "~/.npm/_cacache",
      "~/.npm/_npx",
      "~/.cache",
      "~/.bun/**",

      // Cargo cache (Rust, used by Codex)
      "~/.cargo/registry/**",
      "~/.cargo/git/**",
      "~/.cargo/.package-cache",

      // Shell completion cache
      "~/.zcompdump*",
      "~/.zsh_history",
      "~/.bash_history",
      "~/.z",
      "~/.z.*",
      "~/.conda/",
      "~/.mamba/",
      "~/.pixi/",
      "~/.sqlite_history",

      // XDG directories for app configs/data
      "~/.local/share/**",
      // "~/.config/**",

      "~/.vim/undodir",
      "~/.vim/swapfiles",
      "~/.vim/backupfiles",
      "~/.local/share/claude",
      "~/tmp",
      "/tmp/.X11-unix/",

      "~/.config/surf",
      "~/.firefox-sandbox",
      "~/.plannotator"
    ],

    "denyWrite": [
      // Protect environment files with secrets
      "**/.env",
      "**/.env.*",

      // Protect key/certificate files
      "**/*.key",
      "**/*.pem",
      "**/*.p12",
      "**/*.pfx"
    ],


    "allowRead": [
      "/var/lib/texmf",
      "/var/cache/fontconfig",
      "/var/lib/dpkg",
      "~/.zshrc",
      "~/.bashrc",
      "~/.secrets",
      "~/.oh-my-zsh",
      "~/.tmux-panes",
      "~/.bash_aliases",
      "~/.bashrc-user",
      "~/.gitconfig",
      "~/.vimrc",
      "~/dotfiles/",
      "~/.linuxbrew",
      "~/miniconda3",
      "~/.local",
      "~/bin",
      "~/repos",
      "~/tmp/",
      "/tmp/.X11-unix/",
      "~/.gitconfig/",
      "~/.config/git/",
      "~/.config/leaf",
      "~/.config/vim",
      "~/.config/**",
      "~/.vim/",
      "~/.ssh/known_hosts",
      "~/n",
      "~/.claude",
      "~/.claude.json",
      "~/.pyenv",
      "~/.cargo",
      "~/programs",
      "~/.fzf.zsh",
      "~/.fzf.bash",
      "~/.npmrc",
      "~/Important",
      "~/.config/surf",

      "~/n",
      "~/.npm",
      "~/npm"
    ],

    // default deny everything instead,
    // use allowRead for fine grained read permissions
    "defaultDenyRead": true,
    "denyRead": [
      // SSH private keys and config
      "~/.ssh/id_*",
      "~/.ssh/config",
      "~/.ssh/*.pem",

      // GPG keys
      "~/.gnupg/**",

      // Cloud provider credentials
      "~/.aws/**",
      "~/.config/gcloud/**",
      "~/.kube/**",

      // Docker config (may contain registry auth)
      "~/.docker/**",

      // Package manager auth tokens
      "~/.pypirc",
      "~/.netrc",
      "~/.git-credentials",
      "~/.cargo/credentials",
      "~/.cargo/credentials.toml"
    ]
  },

  "command": {
    "useDefaults": true,
    "deny": [
      // Git commands that modify remote state
      "git push",
      "git reset",
      "git clean",
      "git checkout --",
      "git rebase",
      "git merge",

      // Package publishing commands
      "npm publish",
      "pnpm publish",
      "yarn publish",
      "cargo publish",
      "twine upload",
      "gem push",

      // Privilege escalation
      "sudo",

      // GitHub CLI commands that modify remote state
      "gh pr create",
      "gh pr merge",
      "gh pr close",
      "gh pr reopen",
      "gh pr review",
      "gh pr comment",
      "gh release create",
      "gh release delete",
      "gh repo create",
      "gh repo fork",
      "gh repo delete",
      "gh issue create",
      "gh issue close",
      "gh issue comment",
      "gh gist create",
      "gh workflow run",
      "gh api",
      "gh auth login",
      "gh secret set",
      "gh secret delete",
      "gh variable set",
      "gh variable delete"
    ]
  }
}
```

This is modified from: https://github.com/fencesandbox/fence/blob/main/internal/templates/code.json

It might be that over time whenever something is not working you have to add some more permissions are add work arounds. 

I run this for a few months now, so it already got some additions over that time. 

I also have for example a script to start firefox sandboxed for using [plannotator](https://github.com/backnotprop/plannotator).

```sh
#!/bin/sh

set -u

profile="$HOME/.firefox-sandbox"

export MOZ_ASSUME_USER_NS="${MOZ_ASSUME_USER_NS:-0}"

if [ "${FIREFOX_NO_SANDBOX:-0}" = "1" ]; then
    export MOZ_DISABLE_CONTENT_SANDBOX=1
    export MOZ_DISABLE_GMP_SANDBOX=1
    export MOZ_DISABLE_RDD_SANDBOX=1
    export MOZ_DISABLE_SOCKET_PROCESS_SANDBOX=1
fi

exec firefox -no-remote -profile "$profile" "$@"
```

In your `~/.bashrc` or `~/.zshrc`

```sh
export PLANNOTATOR_BROWSER=firefox-sandbox.sh
```
