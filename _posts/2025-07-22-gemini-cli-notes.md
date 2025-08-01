---
layout: post
title:  "Gemini CLI Notes"
date:   2025-07-22 16:47:00
permalink: gemini-cli-notes
categories: ai agents llms productiviy devtools osx gemini gemini-cli google  
---

## Why this post?
The Gemini CLI [launch post](https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/) summarizes the tool's capabiliies and the [GitHub repo README.md](https://github.com/google-gemini/gemini-cli/) should help you get up and running quickly. 

However, the [full documentation](https://github.com/google-gemini/gemini-cli/blob/main/docs/index.md) covers a lot of ground and I thought others might find the notes I took to help me navigate it useful. While there is a bit of a Mac focus, most of this should hopefully apply on other platforms as well.

## TL;DR
- Most settings are controlled via `~/.gemini/settings.json`. Project-specific `.gemini/settings.json` settings will be merged with the ones in `~/.gemini/settings.json`, so that you need only specify the ones you want to override.
- API keys and some other settings are controlled via environment variables in `~/.gemini/.env`, which take precedence over `~/.gemini/settings.json`. You can create project-specific `.gemini/.env` files, but they will _not_ be merged with `~/.gemini/.env`, requiring you to specify every variable and not just the ones you want to override.
- Sandboxing is disabled by default. To enable it, add `"sandbox": true` to `~/.gemini/settings.json` or `export GEMINI_SANDBOX="true"` in `~/.gemini/.env`. On a Mac, `export SEATBELT_PROFILE="<profile>"` in `~/.gemini/.env` to select from one of several bundled `sandbox-exec` profiles, with `permissive-open` being the default.
- Usage stats are collected by default. To opt out, set `"usageStatisticsEnabled": false` in `~/.gemini/settings.json`".
- Specify context such as coding guidelines in `~/.gemini/GEMINI.md`. Hierarchical `.gemini/GEMINI.md` files are supported with context merged together. For additional info, use the `--debug` command-line option or the `/memory show` REPL command. Add context dynamically during a session with the `/memory add` REPL command.
- Use `gemini --model <model>` or `export GEMINI_MODEL="<model>"` in `~/.gemini/.env` to override the default model, which is `gemini-2.5-pro`.

## Authentication
The first time you launch Gemini CLI (or, more generally, if there is no `"selectedAuthType"` specified in `settings.json`), you will be prompted to authenticate yourself with Google's AI services in [one of three ways](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/authentication.md):

- "Login with Google". This uses the [Gemini Code Assist](https://developers.google.com/gemini-code-assist/docs/overview) license, of which there is a [free personal edition](https://developers.google.com/gemini-code-assist/docs/overview#supported-features-gca). The corresponding `"selectedAuthType"` is `"oauth-personal"`.

- "Use Gemini API Key". Create a Gemini API key using [AI Studio](https://aistudio.google.com/apikey) and `export GEMINI_API_KEY="<Your Gemini API Key>"` in `~/.gemini/.env`. The corresponding`"selectedAuthType"` is `"gemini-api-key"`.

- "Vertex AI". In Express Mode, set `GOOGLE_API_KEY` in `~/.gemini/.env`. Otherwise, set both `GOOGLE_CLOUD_PROJECT` and `GOOGLE_CLOUD_LOCATION`. The corresponding`"selectedAuthType"` value in `~/.gemini/settings.json` is `"vertex-ai"`.

Your choice will determine the request limits ([Gemini Code Assist](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier), [Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/quotas), [Vertex AI Express Mode](https://cloud.google.com/vertex-ai/generative-ai/docs/start/express-mode/overview#models)) and [what data is collected](https://github.com/google-gemini/gemini-cli/blob/main/docs/tos-privacy.md).

You can subsequently switch between different authentication methods in a `gemini` session using the `/auth` REPL command.
 
## Configuration
Gemini CLI configuration involves a mix of command-line arguments, `settings.json` files and environment variables. The exact order in which configuration layers are processed is described [here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#configuration-layers). 

### settings.json
Most Gemini CLI settings are controlled via `settings.json`. The full list of supported `settings.json` settings is [here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#available-settings-in-settingsjson). In particular, if you want to [opt out](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#usage-statistics) of usage stats collection, set `"usageStatisticsEnabled": false`.

User-level settings are stored in `~/.gemini/settings.json`. You can create project-specific overrides in `.gemini/settings.json`.

### Environment Variables and .env Files
Certain settings are controlled via environment variables. The full list of supported environment variables is [here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#environment-variables--env-files).

While environment variables can be set directly in `~/.zshrc` or `~/.bashrc`, the recommended approach is to store them in dedicated `.env` files and place these inside `.gemini` directories. You can create project-specific `.gemini/.env` files, but they [work differently](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/authentication.md#persisting-environment-variables-with-env-files) from project-specific `.gemini/settings.json` files.

### An Illustrative Example
Let's use sandboxing to illustrate how it all fits together, since it's off by default (more on that in the next section) and can be alternatively turned on via `settings.json`, environment variables and command-line switches:
- Create an empty project directory: `mkdir -p ~/junk/myproject`.
- `cd ~/junk/myproject` so that `myproject` is your current project.
- Run `gemini`. Sandboxing is _off_ (the status bar at the bottom will say "no sandbox (see /docs)"). Exit `gemini`.
- Add `"sandbox": true` to the user-level settings in `~/.gemini/settings.json`. You should also see additional settings there, such as `"theme"` and `"selectedAuthType"`.
- Run `gemini`. Sandboxing is _on_ (on a Mac, the status bar will say "macOS Seatbelt (permissive-open)" if you are using the default Seatbelt profile). Exit `gemini`.
- Create a project-specific `settings.json` file: `mkdir .gemini; echo '{"sandbox": false}' > .gemini/settings.json`.
- Run `gemini`. Sandboxing is _off_, **because project-level `settings.json` settings override user-level `settings.json` settings**. Note that you only need to specify the value of `sandbox`, the setting you want to override. Exit `gemini`.
- Set the `GEMINI_SANDBOX` environment variable via a user-level `.env` file: `echo 'export GEMINI_SANDBOX="true"' >> ~/.gemini/.env`.
- Run `gemini`. Sandboxing in _on_, **because environment variables take precedence over `settings.json` settings**. Exit `gemini`.
- Create a project-specific `.env` file: `echo 'export GEMINI_SANDBOX="false"' > .gemini/.env`. This is sufficient if `"selectedAuthType"` is set to `"oauth-personal"` in `~/.gemini/settings.json`.
  - However, unlike with project-level `settings.json` overrides (where only `sandbox` needed to be specified), _all_ relevant environment variables must be set in `.gemini/.env` file and not just `GEMINI_SANDBOX`.
  - For example, if `"selectedAuthType"` is set to `"gemini-api-key"` in `~/.gemini/settings.json`, you will need to `export GEMINI_API_KEY="<Your Gemini API Key>"` in `myproject/.gemini/.env` in order to successfully authenticate yourself when running `gemini` from within `myproject`.
- Run `gemini`. Sandboxing in _off_, **because the project-specific `.gemini/.env` file _shadows_ the user-level `~/.gemini/.env` file**. Exit `gemini`.
- Run `gemini --sandbox`. Sandboxing is _on_, **because command-line arguments take precedence over environment variables**.

## Sandboxing
Given Gemini CLI's ability to execute arbitrary commands on your machine, safety is a concern. To mitigate this risk, Gemini CLI supports [sandboxing](https://github.com/google-gemini/gemini-cli/blob/main/docs/sandbox.md), which is disabled by default (except in "YOLO mode"). There are multiple ways of enabling it, covered in the Configuration section above.

The default sandboxing methodology is platform-specific: `"sandbox-exec"` on a Mac and `"docker"` elsewhere. You can override it by explicitly setting `GEMINI_SANDBOX` to `"sandbox-exec"`, `"docker"` or `"podman"`, though `"sandbox-exec"` only works on a Mac.

Gemini CLI ships with several [predefined sandbox-exec profiles](https://github.com/google-gemini/gemini-cli/blob/main/docs/sandbox.md#macos-seatbelt-profiles): "permissive-open", "permissive-proxied", "permissive-closed", "restrictive-open", "restrictive-proxied" and "restrictive-closed". The default profile is "permissive-open". It restricts writes outside the project directory, but allows most other operations. To select a different profile, `export SEATBELT_PROFILE="<profile>"` in `.gemini/.env`.

The actual profile definitions live [here](https://github.com/google-gemini/gemini-cli/tree/main/packages/cli/src/utils) and follow the naming schema `sandbox-macos-<profile-name>.sb`, e.g. [`sandbox-macos-permissive-open.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-permissive-open.sb) for "permissive-open". It's also possible to create custom Seatbelt profiles and store them in your project's `.gemini` directory, as in `my-project/.gemini/sandbox-macos-custom-profile.sb`. The details of the `*.sb` file format are somewhat obscure, but [this page](https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles?tab=readme-ov-file) offers a good starting point. The full guide is [here](https://reverse.put.as/wp-content/uploads/2011/09/Apple-Sandbox-Guide-v1.0.pdf).

## Tools
Gemini CLI comes with a number of [built-in tools](https://github.com/google-gemini/gemini-cli/blob/main/docs/core/tools-api.md#built-in-tools) which enable it to interact with your local environment. You can list the tools available during a given session using the `/tools` REPL coommand. For a longer description of each tool, use `/tools desc`. When sandboxing is enabled, Gemini CLI will only be able to use tools supported by the specified sandboxing methodology.

Gemini CLI will prompt you before executing each tool unless it is in “YOLO mode”, which is off by default. YOLO mode can be enabled at start up via `gemini --yolo` or toggled off and on during a Gemini CLI session using `Ctrl + y`. **It's humorous name nothwistanding, YOLO mode should be considered dangerous given the criticality of having a human in the loop for any tools capable of modifying the local environment.**

You can explicitly limit the tools availale to Gemini CLI via the `coreTools` and `excludeTools` settings in `settings.json`. The former explicitly lists what tools should be made availalbe (all, by default), whereas the latter excludes particular tools (none, by default). Moreover, when sandboxing is enabled, Gemini CLI will only be able access tools which are availalbe within the specified sandbox environment. 

You can also impose tool-specific restrictions. Take the Shell Tool ([run_shell_command](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/shell.md)) for example, which is used to run scripts and perform command-line operations. You can prohibit specific commands such as `rm`, or only allow particular commands such as `git`, as described in the [Command Restrictions](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/shell.md#command-restrictions) section.

For example, if you add `"coreTools": ["run_shell_command(git)"]` to `settings.json` and then ask Gemini CLI to "run ls", you will get an error message along the lines of `"Command 'ls -F' is not in the allowed commands list"`.

## Instructional Context
TODO    
**To get a better sense of how it all fits together, try turning on debug mode with `gemini --debug`**.

## Model Selection
The `GEMINI_MODEL` environment variable lets you override the default model, which is equivalent to `export GEMINI_MODEL="gemini-2.5-pro"`. While there is no equivalent `settings.json` setting or REPL command for it, you can also specify which model to use for a given session via the `--model` [command-line argument](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#command-line-arguments). But why switch to other models when [Gemini 2.5 Pro](https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-pro) is already "the best"? 

[Rate limits](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier) are one reason. When using Gemini API keys, for example, you get more than twice as many free requests per day with [Gemini 2.5 Flash](https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-flash) (`export GEMINI_MODEL="gemini-2.5-flash"`) as with Gemini 2.5 Pro. 

Response time is another. With [Gemini 2.5 Flash-Lite](https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-flash-lite) (`export GEMINI_MODEL="gemini-2.5-flash-lite"`), you get much lower latency in addition to ten times as many free requests per day.

Supported `<model>` values include `gemini-{2.0, 2.5}-{flash-lite, flash, pro}`.

You can use the `/stats model` REPL command to see the number of requests made so far and their average latency. It will also show you the number of tokens used.

## Sample setup
A bare-bones `~/.gemini/settings.json` might look like the following:
```
{
  "theme": "Dracula",
  "usageStatisticsEnabled": false,
  "selectedAuthType": "gemini-api-key"
}
```