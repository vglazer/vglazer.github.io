---
layout: post
title:  "Gemini CLI Notes"
date:   2025-07-22 16:47:00
permalink: gemini-cli-notes
categories: ai agents llms productiviy devtools osx gemini gemini-cli google  
---

## Background and Documentation
- [Launch post](https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/)
- [GitHub repo](https://github.com/google-gemini/gemini-cli/)
- [Full Docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/index.md)
- [settings.json settings](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#available-settings-in-settingsjson)
- [Environment varialbes](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#environment-variables--env-files)
- [REPL commands](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/commands.md)
- [Non-interactive mode](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/index.md#non-interactive-mode)
- [MCP servers](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/tutorials.md#setting-up-a-model-context-protocol-mcp-server)

## Configuration Layers
- User [settings](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#available-settings-in-settingsjson) are in `~/.gemini/settings.json`.
- Project-specific `myproject/.gemini/settings.json` files _override_ `~/.gemini/settings.json`. You only need to specify settings whose user-level values you want to change.
- API keys and some other settings are controlled via [environment variables](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#environment-variables--env-files). It is recommended to store them in `~/.gemini/.env` instead of adding them directly to `~/.zshrc` or `~/.bashrc`.
- Project-specific `myproject/.gemini/.env` files _shadow_ `~/.gemini/.env`. You must specify all environment variables, even if you don't want to change their user-level values.
- .env takes precedence over `settings.json` and command-line arguments trump both. See the Appendix below for an example of how Gemini CLI [processes configuration layers](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#configuration-layers).
- In particular, the directory where you launch `gemini` will affect which `settings.json` and `.env` settings are picked up.

## Authentication, Request Limits and Privacy
- The `"selectedAuthType"` setting determines how you [authenticate with Google's AI services](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/authentication.md): `"oauth-personal"` (personal Google account), `"gemini-api-key`" ([AI Studio](https://aistudio.google.com/prompts/new_chat)) or `"vertex-ai"` ([Vertex AI](https://cloud.google.com/vertex-ai)).
- You will be prompted to pick an auth type the first time you launch Gemini CLI, or more generally any time `"selectedAuthType"` isn't found in `settings.json`. You can also use the `/auth` REPL command to switch to a different auth type later. 
- The auth type determines the model request limits as well as what data Google collects. See the [Terms of Service and Privacy Notice](https://github.com/google-gemini/gemini-cli/blob/main/docs/tos-privacy.md) for details. **Keep in mind that if you authenticate using your personal Google account or use the free Google AI tier, your code, prompts and responses will be used to train Google models**. 
- If `"selectedAuthType"` is set to `"oauth-personal"`, you will need to log into your Google account via the browser. This will create `~/.gemini/google_accounts.json` and `~/.gemini/oauth_creds.json`. 
  - Currently (August 2025), you get the most free Gemini 2.5 Pro requests this way ([60 per minute and 1,000 per day](https://github.com/google-gemini/gemini-cli?tab=readme-ov-file#common-configuration-steps)). 
  - However, **[token caching](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/token-caching.md) is not available for this auth type**. You can view your token usage for a given `gemini` session using the `/stats` REPL command.
- If `"selectedAuthType"` is set to `"gemini-api-key`", you will need to `export GEMINI_API_KEY="<Your Gemini API Key>"` in e.g. `~/.gemini/.env`. You can obtain a Gemini API key [here](https://aistudio.google.com/apikey). 
  - Currently (August 2025), the [Free Tier rate limits](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier) for Gemini 2.5 Pro are 5 requests per minute and 100 requests per day. While this is an order of magnitude lower than what you get by authenticating using your personal Google account, **token caching is available for this auth type**. 
  - To get 1,000 requests per day using the Free Tier, you would need to switch to Gemini 2.5 Flash Lite, a less capable model. The lowest paid tier, Tier 1, gives you 150 requests per minute and 10,000 requests per day using Gemini 2.5 Pro.
- Anonymized usage stats, such what tools and models are used, are collected by default. Add `"usageStatisticsEnabled": false` to `~/.gemini/settings.json`" to [opt out](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#usage-statistics).

## Model Selection
- You can use the `--model` command-line argument to override the default moded use by Gemini CLI (currently Gemini 2.5 Pro) like so: `gemini --model <model>`. Supported `<model>` values include `gemini-{2.5, 2.0}-{pro, flash, flash-lite}`.
- You can also `export GEMINI_MODEL="<model>"` in `~/.gemini/.env` or another `.env` file.

## Built-in Tools
- Gemini CLI comes with a number of [built-in tools](https://github.com/google-gemini/gemini-cli/blob/main/docs/core/tools-api.md#built-in-tools). 
- You can list the tools available during a given `gemini` session using the `/tools` REPL coommand. For a longer description of each tool, use `/tools desc`.
- You can use the `coreTools` setting to explicitly list the tools that should be made available to Gemini CLI. For example, `"coreTools" : []` means that no tools will be made available whereas `"coreTools": ["LSTool"]` means that only the `LSTool` will be made available. 
- You can use the `excludeTools` setting to prevent tools from being made available to Gemini CLI.   

## Sandboxing
- When sandboxing is enabled, Gemini CLI will only be able to use tools supported by the sandbox environment, regardless of what tools are available to it.
- Sandboxing is off by default. Add `"sandbox": true` to `~/.gemini/settings.json` to turn it on. 
- The default sandboxing methodology is platform-specific: Seatbelt on the Mac and Docker elsewhere. You can explicitly set `"sandbox"` to `"docker"`, `"sandbox-exec"` (Mac only) or `"podman"` to override that.
- On a Mac, `export SEATBELT_PROFILE="<profile>"` in `~/.gemini/.env` to pick your Seatbelt profile. The [built-in profiles](https://github.com/google-gemini/gemini-cli/blob/main/docs/sandbox.md#macos-seatbelt-profiles) are `{permissive, restrictive}-{open, proxied, closed}`. The default is `permissive-open`, which restricts writes outside the project directory but allows most other operations. 
- `export SEATBELT_PROFILE="custom"` in `.gemini/.env` to use custom Seatbelt profiles stored in `.gemini/sandbox-macos-custom.sb`. You can start by copying one of the built-in profiles, which live [here](https://github.com/google-gemini/gemini-cli/tree/main/packages/cli/src/utils) and follow the naming schema `sandbox-macos-<profile-name>.sb` (e.g. [`sandbox-macos-permissive-open.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-permissive-open.sb) for `permissive-open` and [`sandbox-macos-permissive-closed.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-restrictive-closed.sb) for `permissive-closed`).
- Note that "custom" is just a tag. If you set `SEATBELT_PROFILE` to `"custom"` in `~/.gemini/.env` every project where `~/.gemini/.env` isn't shadowed will need to have a `.gemini/sandbox-macoas-custom.sb`, but they can be completely different from each other (i.e. a copy of `sandbox-macos-permissive-closed.sb` rather than `sandbox-macos-permissive-open.sb`).
- See [this page](https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles?tab=readme-ov-file) for more info about creating custom Seatbelt profiles. They are written in a language called SBPL (SandBox Profile Language), which is a Scheme-based embedded DSL. The [Apple Sandbox Guide](https://reverse.put.as/wp-content/uploads/2011/09/Apple-Sandbox-Guide-v1.0.pdf) has the details.

## Instructional Context / Memory (System Prompt)
- Store the user-level system prompt in `~/.gemini/GEMINI.md`. Gemini CLI also calls this "instructional context" or "memory".
- Any project-specific context in `myproject/.gemini/GEMINI.md` will be merged with `~/.gemini/GEMINI.md`. Use the `/memory show` REPL command to see the overall, merged system prompt during a `gemini` session.
- More generally, this also works hierarchically as described here. so that you can store related projects in the same parent directory whose `GEMINI.md` will apply to all of them, in addition to any project specific `GEMINI.md` you specify. Use the `/memory show` REPL command to see the overall, merged system prompt during a `gemini` session.

## Example setup
Say the top-level directory where you keep your Gemini CLI project is `GEMINI_PROJ_ROOT`. Then your configuration and context files might look something like this:
`~/.gemini/settings.json`:
```
{
  "theme": "Dracula",
  "usageStatisticsEnabled": false,
  "sandbox": true,
  "selectedAuthType": "oauth-personal"
}
```
`~/.gemini/.env`:
```
export GEMINI_API_KEY="<Your Gemini API Key>" # https://aistudio.google.com/apikey
export SEATBELT_PROFILE="custom" # {permissive, restrictive}-{open, proxied, closed} or a custom
export GEMINI_MODEL="gemini-2.5-pro" # gemini-{2.5, 2.0}-{pro, flash, flash-lite}
```
`~/.gemini/GEMINI.md`:
```
## General Instructions:
- You are an expert software developer responsible for a critical production system
- Write simple, easy to understand code
- Add comments to explain unusual constructs
- Ensure code is modular and suitable for unit testing
- Optimize the performance of your code in terms of both run time and memory usage

```
`$GEMINI_PROJ_ROOT/py/GEMINI.md`:
```
## Python-specific instructions
- Use uv for dependency management
- Use ruff for linting and formatting 
- Use pytest for unit testing
- Use pydantic for data validation
```

## Appendix: Configuration Layers in Action
Let's use sandboxing to illustrate the order in which Gemini CLI processes configuration layers, since it's off by default and can alternatively be turned on via `settings.json` files, `.env` files and command-line arguments.

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