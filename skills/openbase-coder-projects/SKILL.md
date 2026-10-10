---
name: openbase-coder-projects
description: Create a new project or register, find and understand automatically discovered Openbase Coder projects. Use when the user asks to create or start a new project, app or workspace.
---

# Openbase Coder Projects

Run the Openbase Coder CLI on the computer where the project should live. Local commands affect that computer; a cloud workspace and the user's personal computer have separate files.

1. Check `openbase-coder projects list` and the user's intended location. Reuse the existing project for a follow-up task. For a new project, choose a dedicated folder using the user's requested location or existing project organization; if neither exists, use a folder under `~/Projects`. Never use the home directory itself or a system/temporary directory as the project root.
2. Run `openbase-coder projects create PATH` with a quoted path, for example `openbase-coder projects create "$HOME/Projects/my-app"`. The command creates missing parent folders, registers the project and prints its absolute root. An existing folder is preserved and registered. If the command fails, inspect the error and fix the cause before starting a worker or claiming success.
3. Start the worker with the returned absolute path as `cwd` and the user's task as `prompt`. Check that its turn actually started. If the folder belongs to a Multi workspace, the returned path is the workspace root.
4. Have the worker inspect existing files, check for an applicable scaffold/template, and build the requested project. Creating a folder is only the first step; verify the requested result before reporting completion.

`openbase-coder projects add PATH` registers an existing folder. `openbase-coder projects list` prints JSON for manual registrations and automatically discovered projects. Registration is local and does not initialize Git, create a remote repository or publish anything.

Openbase automatically discovers projects from agent working directories when thread state is synchronized, folding nested Multi repositories into their workspace root. This registers existing folders; it does not create a directory or scaffold an app. Explicit creation registers the folder immediately, without waiting for thread synchronization. A project previously hidden from the list is restored by `create` or `add`.
