---
layout: post
title:  "Gemini CLI Notes"
date:   2025-07-22 16:47:00
permalink: gemini-cli-notes
categories: ai agents llms productiviy devtools osx gemini gemini-cli google  
---

## Background
- [Launch post](https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/)
- [GitHub repo](https://github.com/google-gemini/gemini-cli/)
- [Docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/index.md)

## Overview
- User [settings](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#available-settings-in-settingsjson) are in `~/.gemini/settings.json`.
- Project-specific `myproject/.gemini/settings.json` files _override_ `~/.gemini/settings.json`. You only need to specify settings whose user-level values you want to change.
- API keys and some other settings are controlled via [environment variables](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#environment-variables--env-files). Store them in `~/.gemini/.env` instead of adding them directly to `~/.zshrc` or `~/.bashrc`.
- Project-specific `myproject/.gemini/.env` files _shadow_ `~/.gemini/.env`. You must specify all environment variables, even if you don't want to change their user-level values.
- The details of how all this fits together are [here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md).
- The `"selectedAuthType"` setting determines how you [authenticate with Google's AI services](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/authentication.md): `"oauth-personal"` (personal Google account), `"gemini-api-key`" (AI Studio) or `"vertex-ai"` (Vertex AI).
- You will be prompted to pick an auth type the first time you launch Gemini CLI, or any time `"selectedAuthType"` isn't found. You can also use the `/auth` REPL command to switch to a different auth type. The auth type you select will determine privacy settings and request limits. See the [Terms of Service and Privacy Notice](https://github.com/google-gemini/gemini-cli/blob/main/docs/tos-privacy.md).
- If `"selectedAuthType"` is set to `"oauth-personal"`, you will need to log into your Google account via the browser. Currently, you get the most free Gemini 2.5 Pro requests per day this way (1000). This will create `~/.gemini/google_accounts.json` and `~/.gemini/oauth_creds.json`. Unless you use an enterprise account, your code, prompts and responses will be used to train Google models.
- If `"selectedAuthType"` is set to `"gemini-api-key`", you will need to `export GEMINI_API_KEY="<Your Gemini API Key>"` in `~/.gemini/.env`. Get your Gemini API key [here](https://aistudio.google.com/apikey). Unless you are using a paid tier, your code, prompts and responses will be used to train Google models. The free tier gives you 100 Gemini 2.5 Pro requests per day.
- Usage stats are collected by default. Add `"usageStatisticsEnabled": false` to `~/.gemini/settings.json`" to [opt out](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#usage-statistics).
- Gemini CLI comes with a number of [built-in tools](https://github.com/google-gemini/gemini-cli/blob/main/docs/core/tools-api.md#built-in-tools). You can list the tools available during a given session using the `/tools` REPL coommand. For a longer description of each tool, use `/tools desc`. When sandboxing is enabled, Gemini CLI will only be able to use tools supported by the specified sandboxing methodology.
- You can use the the `coreTools` and `excludeTools` settings to [restrict](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/shell.md#command-restrictions) what shell commands can be executed, but sandboxing is [more secure](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/shell.md#security-note-for-excludetools).
- [Sandboxing](https://github.com/google-gemini/gemini-cli/blob/main/docs/sandbox.md) is off by default. Add `"sandbox": true` to `~/.gemini/settings.json` to turn it on. 
- The default sandboxing methodology is platform-specific: Seatbelt on the Mac and Docker elsewhere. You can explicitly set `"sandbox"` to `"docker"`, `"sandbox-exec"` (Mac only) or `"podman"` to override that.
- On a Mac, `export SEATBELT_PROFILE="<profile>"` in `~/.gemini/.env` to pick your Seatbelt profile. The [built-in profiles](https://github.com/google-gemini/gemini-cli/blob/main/docs/sandbox.md#macos-seatbelt-profiles) are `{permissive, restrictive}-{open, proxied, closed}`. The default is `permissive-open`, which restricts writes outside the project directory but allows most other operations. 
- `export SEATBELT_PROFILE="custom"` in `.gemini/.env` to use custom Seatbelt profiles stored in `.gemini/sandbox-macos-custom.sb`. You can start by copying one of the built-in profiles, which live [here](https://github.com/google-gemini/gemini-cli/tree/main/packages/cli/src/utils) and follow the naming schema `sandbox-macos-<profile-name>.sb` (e.g. [`sandbox-macos-permissive-open.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-permissive-open.sb) for `permissive-open` and [`sandbox-macos-permissive-closed.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-restrictive-closed.sb) for `permissive-closed`).
- Note that "custom" is just a tag, not a policy. If you set `SEATBELT_PROFILE` to `"custom"` in `~/.gemini/.env` every project where `~/.gemini/.env` isn't shadowed will need to have a `.gemini/sandbox-macoas-custom.sb`, but they can be completely different from each other (i.e. a copy of `sandbox-macos-permissive-closed.sb` rather than `sandbox-macos-permissive-open.sb`).
- See [this page](https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles?tab=readme-ov-file) for more info about creating custom Seatbelt profiles. They are written in a language called SBPL (SandBox Profile Language), which is a Scheme-based embedded DSL. The [Apple Sandbox Guide](https://reverse.put.as/wp-content/uploads/2011/09/Apple-Sandbox-Guide-v1.0.pdf) has the details.
- Store the user-level system prompt in `~/.gemini/GEMINI.md` as described [here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#context-files-hierarchical-instructional-context). The project-specific system prompt in `myproject/GEMINI.md` will be merged with it. This also works hierarchically, you can store related projects in the same directory whose `GEMINI.md` will apply to all of them, in addition to any project specific `GEMINI.md` you specify. Use the `/memory show` REPL command to see the overall, merged system prompt during a `gemini` session.
- `export GEMINI_MODEL="<model>"` in `~/.gemini/.env` to override the default model, which is `gemini-2.5-pro`. Supported `<model>` values include `gemini-{2.5, 2.0}-{pro, flash, flash-lite}`. Different models have different [rate limits](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier).

## Sample setup
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
export SEATBELT_PROFILE="permissive-open" # {permissive, restrictive}-{open, proxied, closed} or custom
export GEMINI_MODEL="gemini-2.5-pro" # gemini-{2.5, 2.0}-{pro, flash, flash-lite}
```
`~/.gemini/GEMINI.md`:
```
## General Instructions:
- You are an experienced software developer writing code for a production system
- Prefer simple, easy to understand code
- Prioritize maintainability over performance
- Avoid hadcoding anything. Use named constants instead.
- Add comments to explain unusual constructs

## C++-specific instructions:
- Favor modern C++ constructs whenever possible
- Use CMake as the build system
- Use Ninja as the CMake generator
- Use GTest for unit testing
- Use clang-format for formatting

## Python-specific instructions
- Use uv for dependency management
- Use pytest for testing
- Use ruff for linting and formatting
- Use ruff for formatting
- Use ty for type checking
- Use pydantic for data validation
- Always format files using ruff
- Always check files using ruff
- Never run pip directly, use uv
- Never run python directly, use uv
- Never remove tests after creating them
- Never remove dependencies from project.toml after adding them

## Bash-specific instructions
- Use shellcheck for linting
```

### Gemini CLI configuration layers in action
Let's use sandboxing to illustrate the order in which Gemini CLI [processes configuration layers](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#configuration-layers), since it's off by default and can alternatively be turned on via `settings.json` files, `.env` files and command-line switches.

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