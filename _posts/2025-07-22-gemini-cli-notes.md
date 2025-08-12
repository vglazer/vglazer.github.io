---
layout: post
title:  "Gemini CLI Notes"
date:   2025-08-11 22:00:00
permalink: gemini-cli-notes
categories: ai agents llms productiviy devtools osx gemini gemini-cli google  
---

## Background and Documentation
- [Launch post](https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/)
- [gemini-cli GitHub repo](https://github.com/google-gemini/gemini-cli/)
- [Documentation](https://github.com/google-gemini/gemini-cli/blob/main/docs/index.md)
  - [Architecture Overview](https://github.com/google-gemini/gemini-cli/blob/main/docs/architecture.md)
  - [settings.json settings](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#available-settings-in-settingsjson)
  - [Environment variables](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#environment-variables--env-files)
  - [REPL commands](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/commands.md)
  - [GEMINI.md files](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#context-files-hierarchical-instructional-context)
  - [Built-in Tools](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/index.md)

## Configuration Layers
- User [settings](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#available-settings-in-settingsjson) are in `~/.gemini/settings.json`.
- Project-specific `myproject/.gemini/settings.json` files _override_ `~/.gemini/settings.json`. You only need to specify settings whose user-level values you want to change.
- API keys and some other settings are controlled via [environment variables](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#environment-variables--env-files). It is recommended to store them in `~/.gemini/.env` instead of adding them directly to `~/.zshrc` or `~/.bashrc`.
- Project-specific `myproject/.gemini/.env` files _shadow_ `~/.gemini/.env`. You must specify all environment variables, even if you don't want to change their user-level values.
- `.env` takes precedence over `settings.json`, whereas command-line arguments trump both. See the Appendix at the end of the post for a deeper dive into how Gemini CLI [processes configuration layers](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#configuration-layers).
- In particular, the current working directory will affect which `settings.json`, `.env` and `GEMINI.md` files `gemini` picks up.

## Authentication, Request Limits and Privacy
- The `"selectedAuthType"` setting determines how you [authenticate with Google's AI services](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/authentication.md): `"oauth-personal"` (personal Google account), `"gemini-api-key`" ([AI Studio](https://aistudio.google.com/prompts/new_chat)) or `"vertex-ai"` ([Vertex AI](https://cloud.google.com/vertex-ai)).
- You will be prompted to pick an auth type the first time you launch Gemini CLI, or more generally any time `"selectedAuthType"` isn't found in `settings.json`. You can also use the `/auth` REPL command to switch to a different auth type later. 
- The auth type determines the model request limits as well as what data Google collects. See the [Terms of Service and Privacy Notice](https://github.com/google-gemini/gemini-cli/blob/main/docs/tos-privacy.md) for details. **Keep in mind that if you authenticate using your personal Google account or use the free Google AI tier, your code, prompts and responses will be used to train Google models**. 
- If `"selectedAuthType"` is set to `"oauth-personal"`, you will need to log into your Google account via the browser. This will create `~/.gemini/google_accounts.json` and `~/.gemini/oauth_creds.json`. 
  - Currently (as of August 2025), you get the most free Gemini 2.5 Pro requests this way: 60 per minute and 1,000 per day. 
  - However, **[token caching](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/token-caching.md) is not available for this auth type**. You can view your token usage for a given `gemini` session using the `/stats` and `/stats model` REPL commands.
- If `"selectedAuthType"` is set to `"gemini-api-key`", you will need to `export GEMINI_API_KEY="<Your Gemini API Key>"` in e.g. `~/.gemini/.env`. You can obtain a Gemini API key [here](https://aistudio.google.com/apikey). 
  - Currently (as of August 2025), the [Free Tier rate limits](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier) for Gemini 2.5 Pro are 5 requests per minute and 100 requests per day. While this is considerably lower than what you get by authenticating using your personal Google account, **token caching is available for this auth type**. 
  - To get 1,000 requests per day using the Free Tier, you would need to switch to Gemini 2.5 Flash Lite (Flash Lite is a version of Lite optimized for low-latency use cases; Lite is not a "reasoning model", unlike Pro). The lowest paid tier, Tier 1, gives you 150 requests per minute and 10,000 requests per day using Gemini 2.5 Pro.
- Anonymized usage stats, such what tools and models are used, are collected by default. Add `"usageStatisticsEnabled": false` to `~/.gemini/settings.json`" to [opt out](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#usage-statistics).

## Model Selection
- You can use the `--model` command-line argument to override the default model (currently Gemini 2.5 Pro) like so: `gemini --model <model>`. 
- Supported `<model>` values include `gemini-{2.5, 2.0}-{pro, flash, flash-lite}`.
- You can also `export GEMINI_MODEL="<model>"` in `~/.gemini/.env` or another `.env` file.

## Built-in Tools
- Gemini CLI comes with a number of [built-in tools](https://github.com/google-gemini/gemini-cli/blob/main/docs/core/tools-api.md#built-in-tools) with the following [execution flow](https://github.com/google-gemini/gemini-cli/blob/main/docs/core/tools-api.md#tool-execution-flow). Their definitions live [here](https://github.com/google-gemini/gemini-cli/tree/main/packages/core/src/tools).
- You can extend the list of built-in tools using MCP Servers. See [this page](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/mcp-server.md) for details.
- Use the `/tools` REPL command to list the tools available in a given `gemini` session. For a longer description of each tool, run `/tools desc`.
- Use the `coreTools` setting in `settings.json` to explicitly list what tools should be made available. For example, `"coreTools" : []` means that no tools will be available whereas `"coreTools": ["LSTool"]` means that only `LSTool` will be available.
- The name a tool appears under when you run `/tools` may not match the one you need to use in `coreTools`. For example, `LSTool` shows up as "ReadFolder". This is because `LSTool.Name` is set to `ReadFolder` in [`ls.ts`](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/ls.ts), where its definition lives. 
- Most built-in tools follow the pattern of `FooBarTool` for the `settings.json name`, “FooBar” for the `/tools` name and `foo-bar.ts` for the definition, but there are a handful of exceptions of which `LSTool` is one.
- Below are the `settings.json` names for the built-in tools, along with what `/tools` calls them and where they are defined:

  | settings.json name   | /tools name     | definition         |
  |----------------------|-----------------|--------------------|
  | `EditTool`           | "Edit"          | [edit.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/edit.ts)         |
  | `GlobTool`           | "FindFiles"     | [glob.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/glob.ts)         |
  | `WebSearchTool`      | "GoogleSearch"  | [web-search.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/web-search.ts)      |
  | `ReadFileTool`       | "ReadFile"      | [read-file.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/read-file.ts)       |
  | `LSTool`             | "ReadFolder"    | [ls.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/ls.ts)           |
  | `ReadManyFilesTool`  | "ReadManyFiles" | [read-many-files.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/read-many-files.ts) |
  | `MemoryTool`         | "Save Memory"   | [memoryTool.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/memoryTool.ts)      |
  | `GrepTool`           | "SearchText"    | [grep.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/grep.ts)         |
  | `ShellTool`          | "Shell"         | [shell.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/shell.ts)           |
  | `WebFetchTool`       | "WebFetch"      | [web-fetch.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/web-fetch.ts)       |
  | `WriteFileTool`      | "WriteFile"     | [write-file.ts](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/write-file.ts)      |  

- The default tools set correponds to having the following `coreTools` setting in `settings.json`: 
```
"coreTools": ["EditTool", 
                "GlobTool", 
                "WebSearchTool", 
                "ReadFileTool", 
                "LSTool", 
                "ReadManyFilesTool", 
                "MemoryTool", 
                "GrepTool", 
                "ShellTool", 
                "WebFetchTool", 
                "WriteFileTool"]
```
- You can also use the `excludeTools` setting in `settings.json` to remove specific tools from the default list. For example, adding `"execludeTools": ["ShellTool"]` to `settings.json` will remove `Shell` from the list of available `/tools`. 
- A tool listed in both `coreTools` and `excludeTools` is _excluded_. This means that you could list the default tool set in `coreTools` and specify just the ones you want to disallow in `excludeTools`. 
- For example, if `settings.json` contains the following
```
"coreTools": ["EditTool", 
                "GlobTool", 
                "WebSearchTool", 
                "ReadFileTool", 
                "LSTool", 
                "ReadManyFilesTool", 
                "MemoryTool", 
                "GrepTool", 
                "ShellTool", 
                "WebFetchTool", 
                "WriteFileTool"]
"excludeTools": ["WebSearchTool", "ShellTool", "WebFetchTool"]
```
then `/tools` won't show `GoogleSearch`, `Shell` and `WebFetch`, whereas the rest of the default tools will remain available.
- Keep in mind that `ShellTool` can execute arbitrary shell commands and thus poses a security risk. To help mitigate this, `ShellTool` supports [granular command restrictions](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/shell.md#command-restrictions). 
- For example, to enable only `git` to be executed, replace `"coreTools": ["ShellTool"]` with `"coreTools": ["run_shell_command(git)"]` in `settings.json`. 
- Although you can also use `run_shell_command` in `excludeTools`, this is [not recommended](https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/shell.md#security-note-for-excludetools) for security reasons.
- Although restricting the commands `ShellTool` can execute via `run_shell_command` does provide a measure of security, it is nevertheless a good idea to enable sandboxing if you are going to make `ShellTool` available (which it is by default, with no command restrictions).

## Sandboxing
- Sandboxing is _off_ by default. Add `"sandbox": true` to `~/.gemini/settings.json` to turn it on.
- When sandboxing is enabled, Gemini CLI will only be able to call tools supported by the sandbox environment, regardless of what tools are included via `coreTools` and `excludeTools` in `settings.json`.
- The default sandboxing methodology is platform-specific: Seatbelt on a Mac and Docker elsewhere.
- On a Mac, `export SEATBELT_PROFILE="<profile>"` in `~/.gemini/.env` to select your Seatbelt profile. The [built-in profiles](https://github.com/google-gemini/gemini-cli/blob/main/docs/sandbox.md#macos-seatbelt-profiles) are `{permissive, restrictive}-{open, proxied, closed}`. The default is `permissive-open`, which restricts writes outside of the project directory but allows most other operations. 
- `export SEATBELT_PROFILE="custom"` in `.gemini/.env` to use a _custom Seatbelt profile_ stored in `.gemini/sandbox-macos-custom.sb`. You can start by copying one of the built-in profiles, which live [here](https://github.com/google-gemini/gemini-cli/tree/main/packages/cli/src/utils) and follow the naming schema `sandbox-macos-<profile-name>.sb` (e.g. [`sandbox-macos-permissive-open.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-permissive-open.sb) for `permissive-open` and [`sandbox-macos-permissive-closed.sb`](https://github.com/google-gemini/gemini-cli/blob/main/packages/cli/src/utils/sandbox-macos-restrictive-closed.sb) for `permissive-closed`).
- See [this page](https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles?tab=readme-ov-file) for some general pointers on creating custom Seatbelt profiles. They are written in a language called SBPL ("SandBox Profile Language"), which is a Scheme-based embedded DSL. The [Apple Sandbox Guide](https://reverse.put.as/wp-content/uploads/2011/09/Apple-Sandbox-Guide-v1.0.pdf) has the details.
- Adding `"sandbox": true"` to `settings.json` is the equivalent of running `gemini --sandbox` (the `--sandbox` command-line argument is a boolean flag). Alternatively, you can explicitly set `"sandbox"` to one of the supported methodologies: `"docker"`, `"sandbox-exec"` (Mac only) or `"podman"`.
- In particular, _you can use Docker-based sandboxing on the Mac_ by specifying `"sandbox: "docker"` in `settings.json`. You will need Docker Desktop for the Mac to be [installed and running](https://docs.docker.com/desktop/setup/install/mac-install/) for this to work properly.
- Similar to having a custom Seatbelt profile, you can also create a custom Dockerfile in `myproject/.gemini/sandbox.Dockerfile`, [as described here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#sandboxing). 
- Per the docs, you should be able to add `export BUILD_SANDBOX=1` to `myproject/.gemini/.env` and have `gemini` automatically build the custom Docker image. 
- When I tried this on a Mac, though (with Gemini CLI installed via `brew install gemini-cli`), I got the following error:
```
ERROR: cannot build sandbox using installed gemini binary; run `npm link ./packages/cli` under gemini-cli repo to switch to linked binary.
```
While I do have the `gemini-cli` repo checked out locally, I haven't tried giving this a go. This is apparently a [known issue](https://github.com/google-gemini/gemini-cli/issues/3404).

## Instructional Context ("Memory")
- Gemini CLI calls the system prompt "instructional context" or "memory". It's loaded from files named `GEMINI.md` (the name can be changed via the `contextFileName` setting in `settings.json`, if desired).
- Context loaded from `GEMINI.md` files will reduce the number of tokens available in the context window, but this should not be an issue for models like Gemini 2.5 Pro where the size of the window is 1 million tokens.
- Store user-level context in `~/.gemini/GEMINI.md`. You can add to it dynamically using the `/memory add` REPL command.
- Store `myproject`-specific context in `myproject/GEMINI.md` (_not_ `myproject/.gemini/GEMINI.md`). See, e.g. Gemini CLI's [own GEMINI.md](https://github.com/google-gemini/gemini-cli/blob/main/GEMINI.md). 
- Project-specific context will be merged with user-level context. Use the `/memory show` REPL command to see the overall, merged context.
- Context can be controlled in a more granular way using multiple `GEMINI.md` files, with the directory you launch `gemini` from determining exactly which ones are included. The details are [here](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/configuration.md#context-files-hierarchical-instructional-context).

## An Illustrative Example
Say that you are on a Mac and didn't set any Gemini CLI-related environment variables in `~/.zshrc` or `~/.bashrc`. Assume that your user-level Gemini CLI configuration looks like this:

`~/.gemini/settings.json`:
```
{
  "theme": "Dracula",
  "usageStatisticsEnabled": false,
  "sandbox": true,
  "selectedAuthType": "gemini-api-key"
}
```
`~/.gemini/.env`:
```
export GEMINI_API_KEY="<Your Gemini API Key>" https://aistudio.google.com/apikey
export GEMINI_MODEL="gemini-2.5-flash-lite" # gemini-{2.5, 2.0}-{pro, flash, flash-lite}
```
`~/.gemini/GEMINI.md`:
```
## User-level context
```

Now suppose that you have a project called `myproject`, configured like this:

`myproject/.gemini/settings.json`:
```
{
  "selectedAuthType": "oauth-personal",
  "excludeTools": ["ShellTool"]
}
```
`myproject/.gemini/.env`:
```
export SEATBELT_PROFILE="custom" # {permissive, restrictive}-{open, proxied, closed}
```
`myproject/.gemini/sandbox-macos-custom.sb`:
```
;; a suitably modified copy of sandbox-macos-permissive-open.sb, say
```
`myproject/GEMINI.md`:
```
## Project-specific context
```
`myproject/some_module/GEMINI.md`:
```
## Module-specific context
```
- If you run  `gemini` _outside_ `myproject`:
  - You will authenticate using a Gemini API key, as per `~/.gemini/settings.json`
  - The API key will be read from `~/.gemini/.env`. If you comment `GEMINI_API_KEY` out you will get an error and `gemini` will fail to launch.
  - Sandboxing will be enabled, as per `~/.gemini/settings.json`.
  - Since no sandboxing methodology was specified explicitly, `sandbox-exec` will be used (the default, on a Mac).
  - The Seatbelt profile will be `permissive-open` (the default).
  - The model will be Gemini 2.5 Flash Lite, as per `~/.gemini/.env`.
  - The default set of tools will be availalbe. Run the `/tools` REPL command to list them.
  - The instructional context will generally consist of the user-level context in `~/.gemini/GEMINI.md`, unless other GEMINI.md files are picked up based on your current working directory. Run the `/memory show` REPL command to confirm.
- On the other hand, if you `cd myproject; gemini`:
  - You will authenticate using your Google account, as per `myproject/.gemini/settings.json`.
  - Since there is no need for an API key, commenting out `GEMINI_API_KEY` in `~/.gemini/.env` has no effect.
  - The Seatbelt profile used will be `myproject/.gemini/sandbox-macos-custom.sb`, as per `myproject/.gemini/.env`.
  - Irrespective of what `~/.gemini/.env` says, the model will be Gemini 2.5 Flash Pro (the default), even though you didn't specify `GEMINI_MODEL` in `myproject/.gemini/.env`.
  - `ShellTool` won't be available, per `myproject/.gemini/settings.json`. Run the `/tools` REPL command to confirm that `"Shell"` does not appear.
  - The instructional context will be a combination of user-level context from `~/.gemini/GEMINI.md`, project-specific context from `myproject/GEMINI.md` and  module-specific context from `myproject/some_module/GEMINI.md`. Run the `/memory show` REPL command to confirm. 
  - Since command-line argument take precedence over `settings.json` and `.env` files, even after you `cd myproject` you can still turn off sandboxing via `gemini --sandbox false` or explicitly select a different model via `gemini --model gemini-2.0-pro`.

## Appendix: Configuration Layer Processing
We can use sandboxing to illustrate the order in which Gemini CLI processes configuration layers, since it's off by default and can alternatively be turned on via `settings.json` files, `.env` files and command-line arguments:

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