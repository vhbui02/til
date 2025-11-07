# VSCode Chat Features and GitHub Copilot Chat Extension

<!-- tl;dr starts -->

**IMPORTANT:** VSCode monthly releases introduce a lot of LLM major breaking changes. Everything is new, non-standardized and unstable. This is both a chance and a challenge to me.

<!-- tl;dr ends -->

## [Cheatsheet](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_chat-variables)

> [!TIP]
>
> For VSCode/VSCodium >=v1.103, `Chat View in New Window` doesn't allow you to choose LLM models for initial prompt.
>
> - Hit `Ctrl + Shift + P > Chat: Show Chats` to restore a past chat session.
> - Prompt from there.

1. **THREE** chat UI:

- **Quick Chat**: for quick learning, explanation.
- **Inline Chat**: for quick code suggesting.
- **Chat view**: for complex operations.

2. [**FOUR** chat modes](https://code.visualstudio.com/docs/copilot/chat/chat-modes):

- Three built-in: **Ask**, **Edit**, **Agent**.
- One custom: **Custom**.

Hit `Ctrl + .` to switch modes.

3. [**FIVE** main ways to customize chat responses in VSCode/VSCodium](https://code.visualstudio.com/docs/copilot/customization/overview):

- **Instructions files:** project-wide coding practices/standards/requirements, language/framework/tech stack-specific rules, commit message + PR/MR title/description guidelines, code review rules (security, performance, coding practices/standards again), ...

  > Yet another prompt engineering practice.
  >
  > - Don't use coding guidelines to enforce style guidelines that can be covered by your linter or static analysis tool.
  > - Don't use wording that is ambiguous or could be interpreted in different ways.
  > - Don't fit multiple different ideas into a single coding guideline.
  >
  > Consider the size and complexity of the repository to do and don't do the following:
  >
  > - Refer to external resources.
  > - Instructions to answer in a particular style.
  > - Always respond with a certain level of detail.

- **Prompt files:** scaffold new component, API route, unit test generation, code review rules (security, performance, coding practices/standards again), step-by-step/specialized workflow required implementation plans, architectural designs, migration strategies, ...

  > Prompt file can be integrated with a chat mode.

- **Custom Chat modes:** Planning mode with access to read-only tools (`codebase`, `fetch`, `search`, ...) to generate implementation plan, Research mode using networking tools, Front-end Developer mode with read-write access to front-end code ONLY, ...

  ```md
  # Planning mode instructions

  You are in planning mode. Your task is to generate an implementation plan for a new feature or for refactoring existing code.
  Don't make any code edits, just generate a plan.

  The plan consists of a Markdown document that describes the implementation plan, including the following sections:

  - Overview: A brief description of the feature or refactoring task.
  - Requirements: A list of requirements for the feature or refactoring task.
  - Implementation Steps: A detailed list of steps to implement the feature or refactoring task.
  - Testing: A list of tests that need to be implemented to verify the feature or refactoring task.
  ```

- **Language models**: Base model for quick + cheap code suggestions, Premium model for complex operations.

- **MCP + tools**: Integrate with external services to fetch latest docs (Context7), run database query and analyze data (Supabase), ...

4. [Chat Contexts](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context)

- `@chat-participant` or `@chat-participant /command`.
- `#chat-variable`
- `Open Editors`
- `Files & Folders`
- `Instructions`
- `Screenshot Window`
- `Source Control`: add history items of a specific commit in a specific branch.
- `Problems`
- `Symbol`
- `Tools` (**NOTE:** max 128 tools)
- ... more in the future

5. [Chat variables](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_chat-variables) are list of tools/tool sets, either from built-in, custom grouped or Extensions.

Most common:

- `#foo.bar` or `#src/`: attach a file/directory as context. Sometimes a file/directory can't be found on auto-completion list, you can right-click the it on Folder View to add it manually.
- `#<symbol>` + `#usages`: add symbol name + find definition and references of a symbol.
- `#context7`, `#sequentialThinking`, `#insert-built-in-tool`, `#insert-custom-tool-set`, ... : run built-in/MCP tools.
- `#selection`: attach a code snippet as context.
- `#todos`: tools for manage and tracking to-do items (introduce in `v1.103`)
- `#problems`: check errors for a specific file.

Searching:

- `#search`: toolset
  - `#fileSearch`: search for files in the workspace using glob pattern.
  - `#textSearch`: search for texts via exact string or regex.
  - `#listDirectory`: list the content of a directory.
  - `#readFile`
  - `#codebase`: include relevant file chunks, symbols, ... from your codebase. Very good if you don't know which specific part of code need to be included in your prompt
  - `#searchResults`: add results from "Find and Replace" fields in Search .

Editing:

- `#edit`: toolset
  - `#createFile`
  - `#createDirectory`
  - `#editNotebook`
  - `#newJupyterNotebook`
  - `#editFiles`

Version Control System:

- `#changes`: get diff of change files in version control system
- `#githubRepo`: search a GitHub repository for a relevent source code snippet, using `owner/repo` syntax. E.g. `what is variable reference in VSCode #githubRepo microsoft/vscode`

Networking:

- `#fetch`: fetch the main content of a web page whose URL is specified in the prompt (context doesn't count).
- `#openSimpleBrowser`: open built-in browser, preview local web app. **CAUTION: extermely unreliable and should not be used.**

Misc/Less common:

- `#new`: toolset, scaffold new VSCode workspace
- `#runCommands`
- `#runNotebooks`
- `#applyPatch`
- `#testFailure`: add VSCode's test failure info.
- `#extensions`: search for VSCode extensions.
- `#VSCodeAPI`: ask questions related to VSCode extension development.
- Jupiter-related tools...

6. **Chat participants:** "expert" of a field.

> [!TIP]
>
> This feature is often neglected. There is too many things that developers need to include in their prompts.

- `@workspace`
- `@workspace /explain`
- `@workspace /fix`
- `@workspace /tests`: generate unit tests for the selected code
- `@workspace /setupTests`: setup tests in the whole project (CAUTION: highly experimental)
- `@workspace /new`: scaffold a new file/project in a workspace.
- `@github`: web search + code search related to GitHub.
- `@vscode` + `@vscode /search`: workspace search.
- `@terminal` + `@terminal /explain`.

## Notable features

> [!CAUTION]
>
> Auto approval all tools is dangerous. Set `chat.tools.autoApprove` to `false`

- Resolve merge conflict with LLM
- Add file commit to chat context (you will need to traverse the source control graph, right-click then select `Add to Chat`)
- Tool sets: More MCP servers === more tools === harder to maintain reusable prompt file/custom chat modes. Create a general tool set

```jsonc
{
  {
  "general": {
    "tools": [
      "changes",
      "edit",
      "fetch",
      "githubRepo",
      "new",
      "openSimpleBrowser",
      "problems",
      "runCommands",
      "runTasks",
      "runTests",
      "search",
      "testFailure",
      "think",
      "todos",
      "usages",
      "context7",
      "sequentialThinking"
    ],
    "description": "The base tool set for all chat modes",
    "icon": "tools"
  }
}
}
```

### [MCP](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

Allows LLM to discover and interact with external tools/functions call to perform specialized tasks in general.

- Read/write/searching for files and directories.
- List repositories, create PR, manage issues of a version control system.
- Connect to database.
- Invoke API.
- ...

Architecture: Client-Server

- MCP clients, such as IDE/Code Editor, connect to MCP servers and request actions on behalf of the LLMs.
- MCP servers, such as Context7, Sequential Thinking, ... provide one or more tools/function calls.
- MCP itself, defines the message format for communication between clients and servers, these communcations can be tool discovery, invocation, response handling.

MCP servers can be **self-hosted** via `npm run`/Docker and **hosted remotely**

Trusted/Verified list of MCP servers:

- [VSCode's curated list](https://code.visualstudio.com/mcp)
- [DockerHub's mcp/ organization](https://hub.docker.com/u/mcp)
- [Official server repository](https://github.com/modelcontextprotocol/servers)

List of MCP servers that I've tried so far:

- [Context7](https://hub.docker.com/r/mcp/context7)
- [Sequential Thinking](https://hub.docker.com/r/mcp/sequentialthinking).

List of MCP servers that I'll take a look:

- [Figma MCP](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)
- [Playwright MCP](https://hub.docker.com/r/mcp/playwright)
- [Sentry MCP](https://hub.docker.com/r/mcp/sentry)
- [Serena](https://github.com/oraios/serena)

Best way to run MCP server is inside containerized environment, enable `Allow in this Workspace` (DO NOT hit `Always Allow`) and add MCP resources (`Ctrl + / > Add Context > MCP Resources`) for filesystem/database/... MCPs.

[Configuration format](https://code.visualstudio.com/docs/copilot/chat/mcp-servers#_configuration-format)

```json
// .vscode/mcp.json
{
  // 💡 Inputs will be prompted on first server start,
  //    then stored securely by VS Code.
  "inputs": [
    {
      "type": "promptString",
      "id": "perplexity-key",
      "description": "Perplexity API Key",
      "password": true
    }
  ],
  "servers": {
    // https://github.com/ppl-ai/modelcontextprotocol/
    // name convention: `camelCase`, no whitespace, use unique and descritive name.
    "Perplexity": {
      "type": "stdio",
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "PERPLEXITY_API_KEY",
        "mcp/perplexity-ask"
      ],
      "env": {
        "PERPLEXITY_API_KEY": "${input:perplexity-key}"
      }
    },
    // https://github.com/github/github-mcp-server/
    "Github": {
      "url": "https://api.githubcopilot.com/mcp/"
    },
    // https://github.com/modelcontextprotocol/servers/tree/main/src/fetch
    "fetch": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

## GitHub Copilot and GitHub Copilot Chat

### [Code review](https://docs.github.com/en/copilot/using-github-copilot/code-review/using-copilot-code-review?tool=vscode)

> [!CAUTION]
>
> Local code review features is bad. But PR code reviews is decent.

The power of OSS and FOSS is the community. Anyone who contribute their code to a public repository with active community can receive code audit from the author/maintainers.

However, for self-developing projects, by setting up LLM as code auditor, a single developer can become a whole tech division.

There are **TWO** types of review:

- **Review selection** or local review. Review code in active editor.
- **Review changes** or remote review. Review commits in a PR.

Select the part you want to review inside active editor and run `GitHub Copilot: Review and Comment` command to start

Tips: If you're contributing to OSS/FOSS projects, the review instructions can be taken from `CONTRIBUTING.md`.

### [Instructions](https://code.visualstudio.com/docs/copilot/copilot-customization#_custom-instructions)

Locations:

- User-level setting file `~/.config/VSCodium/User/settings.json` (NOT RECOMMENDED, uules are project-specific and shouldn't be the same among projects)
- Workspace-level setting file `settings.json`
- `copilot-instructions.md`
- `.github/instructions/*.md`.

For Markdown files, set following frontmatter:

- `applyTo: **`: applies to every Chat request.
- `applyTo: "**/*.ts, **/*.tsx"`: applies only to Chat requests referencing code in files of the specified types.

### [Prompts](https://code.visualstudio.com/docs/copilot/copilot-customization)

Locations:

- User-level `~/.config/VSCodium/User/prompts`.
- Workspace-level `.github/prompts`.

Tips:

- Add "dependencies" by using Markdown links `[index](./index.md)`.
- Support [VSCode's variables](https://code.visualstudio.com/docs/reference/variables-reference) by using `${variableName}` syntax. Most common variables are:

  - Workspace: `${workspaceFolder}`, `${workspaceFolderBasename}`.
  - Selection: `${selection}`, `${selectedText}`.
  - File context: `${file}` (the content of currently opened file), `${fileBasename}` (currently opened file's name), `${fileDirname}` (the parent directory of currently opened file)
  - Input: `${input:variableName}`, `${input:variableName:placeholder}` (pass values to the Prompt file from the Chat input field, e.g. `/create-react-form: formName=MyForm`)

### [Indexing mechanism](https://code.visualstudio.com/docs/copilot/reference/workspace-context)

There are three types of indexes, ordered from least to most robust:

- **Basic index**
- **Local index**
- **Remote index**.

Logic:

- If your project has **<750** indexable files, local index is built **automatically**.
- If your project has **750-2500** indexable files, local index must be built **manually** (via `Ctrl + Shift + P` -> `GitHub Copilot: Build Local Workspace Index`). Subsequent builds are faster than initial build.
- If your project has **>2500** indexable files, local index can't be built, only remote index can via `Ctrl + Shift + P` > `GitHub Copilot: Build Remote Workspace Index`.
- If your project has **>2500** indexable files and does not have a remote index, basic index is used.

### Web UI features

- Ask anything about a repository, a file, a GitHub Issue, ... I once do a code audit on a file and it found several bugs.
- Specify System Prompt. Solve blank page problem with this website: [https://prompts.chat/](https://prompts.chat/)

  ```
  You are a seasoned Python developer with over ten years of experience.
  When teaching, give practical examples after explaining each concept.
  Be concise and to-the-point.
  ```

## FAQ

### Question 1: Does GitHub Copilot Chat index EVERY file in workspace?

A: According to [What sources are used for context?](https://code.visualstudio.com/docs/copilot/reference/workspace-context#_what-sources-are-used-for-context), here are the list of indexable and non-indexable files:

**Indexable**:

- Relavant text files inside workspace (seems ambiguous)
- Directory structure.
- GitHub Code Search index (if the workspace is a GitHub Repository and indexed by code search)
- Symbols and definitions in the workspace.
- Currently selected text + Visible text in the active editor.
- Conversation history

> [!CAUTION]
>
> The final input is hidden, I would like it to be transparent.

**Non-indexable:**

- `.tmp`, `.out`
- Anything in `files.exclude` VSCode setting JSON.
- Anything in `.gitignore` (except for files opened in active editor, or explicitly added)
- Binary files: images, PDFs, ...

### Question 2: Does GitHub Copilot use indexed repository for model training?

A: [Copilot say "No"](https://docs.github.com/en/copilot/using-github-copilot/copilot-chat/indexing-repositories-for-copilot-chat#benefit-of-indexing-repositories), but I don't trust them. Local LLM with restrictive networking rules is the only solution.

## References

- [VSCode Docs's "Customize chat responses in VS Code"](https://code.visualstudio.com/docs/copilot/copilot-customization)
- [GitHub Docs's "About customizing GitHub Copilot Chat responses"](https://docs.github.com/en/copilot/customizing-copilot/about-customizing-github-copilot-chat-responses)
- [GitHub Docs's "Copilot Customization: Tips for defining custom instructions"](https://code.visualstudio.com/docs/copilot/copilot-customization#_tips-for-defining-custom-instructions)
- [GitHub Docs's "Dos and Don'ts for Coding guidelines"](https://docs.github.com/en/copilot/using-github-copilot/code-review/configuring-coding-guidelines#dos-and-donts-for-coding-guidelines)
