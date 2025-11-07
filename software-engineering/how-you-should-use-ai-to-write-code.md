# How You Should Use AI To Write Code

<!-- tl;dr starts -->

This is a summary of [Simon Willison Weblog, 2025-03-11, "How I use LLMs to help me write code"](https://simonwillison.net/2025/Mar/11/using-llms-for-code/), that's hand-typed by me and not by AI.

<!-- tl;dr ends -->

## Prompting workflows

1. Solve [blank page problem](https://thoughtbot.com/blog/the-blank-page-problem) in [initial research phase](how-you-should-build-a-feature.md#step-2-create-an-issue-and-research-the-problem)

Ask it to give you simple prototype that proves that the key requirements of that project can be met:

- Is my goal achievable with current technology or tools?
- If so, what are the possible implementation approaches? Among the available options, which approach is most effective or recommended? Write a simple implementation based on that approach and iterate on more sophisticated implementations.
- If you already have determined your approach and possessed related code examples, use them as inspiration to generate implementation.

Avoid introducing immatured tech stack by practicing [the principle of boring technology](https://boringtechnology.club/). Feed the LLM with a lot of official documentations (via `Context7` MCP server) and examples that you can find or read the documentation and write it yourself.

> Read [Simon Willison's "Running OCR against PDFs and images directly in your browser"](https://simonwillison.net/2024/Mar/30/ocr-pdfs-images/#ocr-how-i-built-this) to know his specific prompts and workflow.

**Remember:** your first prompt will often yield a decent—but not production-ready—result. The more you practice, the better your initial outcomes will be, but expect to iterate with follow-up prompts. Treat imperfect outputs as a starting point to guide the model toward your goals. The AI can revise its answers as many times as needed, so take advantage of this.

2. Tell them exactly what to do

_Example:_ Write implementation and test for a Python function, given its signature and detail instructions.

```
Write a Python function that uses asyncio `httpx` with this signature:

`async def download_db(url, max_size_bytes=5 * 1024 * 1024) -> pathlib.Path:`

Given a URL, this downloads the database to a temp directory and returns a path to it. BUT it checks the content length header at the start of streaming back that data and, if it's more than the limit, raises an error. When the download finishes it uses sqlite3.connect(...) and then runs a PRAGMA quick_check to confirm the SQLite data is valid - raising an error if not. Finally, if the content length header lies to us - if it says 2MB but we download 3MB - we get an error raised as soon as we notice that problem.
```

Then a follow-up prompt:

```
Now write me the tests using `pytest`
```

> **NOTE:** Remember to check the test manually.

3. Test what it writes

Invest some time to **run the test** manually, strengthen your QA habits.

## Tools that can run the code for you

- **SaaS, Sandbox-based tool:** ChatGPT Code Interpreter, Claude Artifacts and ChatGPT Canvas.

- **IDE/Code Editors agentic coding tool:** [Cursor](https://www.cursor.com/), [Windsurf](https://codeium.com/windsurf), [GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat)

- **CLI agentic coding tool:** [Aider](https://aider.chat/), [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)

I'm using GitHub Copilot Chat at the moment due to its generous Pro tier for students. If Claude Code has something similar I will consider it.

## Vibe coding is great way to learn

The best way to learn LLMs is to play with them, and the best way to play with them is to create throwaway, extremely low stakes projects.

Simon Willison has a list of tools that only consists of HTML + CSS + JS code that's written with the assistance of LLMs at his [simonw/tools](https://github.com/simonw/tools) GitHub repository and [tools.simonwillison.net/](https://tools.simonwillison.net/) GitHub Pages published version of the repository.

Visit [tools.simonwillison.net/colophon](https://tools.simonwillison.net/colophon), a tool showing the commit history of each of his tool. Each tool's commit history embedded links to the transcript of the chat that most of the code came from. Its UI/UX is better than GitHub Web UI.

> He also built this tool [mostly with LLM](https://gist.github.com/simonw/323e1b00ee4f8453c7834a7560eeafc1).

## Answering questions about codebases

When working with an unfamiliar tech stack, onboarding can be challenging—especially if the tool lacks clear documentation or the codebase is poorly commented and hard to follow. Chatting directly to your codebase can save a lot of time.

GitHub Copilot Chat's Chat participant `@workspace` can use existing codebase as context so you can ask.

However, the context is blackboxed and wasn't visible to user, personally I would like a more transparent solution like so:

- Create a super large file which is the result of the concatenation of all related files in the repository via [simonw/files-to-prompt](https://github.com/simonw/files-to-prompt).
- Calculate the number of tokens via [simonw/ttok](https://github.com/simonw/ttok).
- Feed the file into LLM with long context window token (about 1M+), such as Google Gemini Pro 2.5.

```sh
files-to-prompt . --cxml --output path/to/output.md
```

> **IMPORTANT:** remember NOT to include `@workspace` chat participant since you already have everything you need.

_Use cases:_

- Generate unofficial documentation derived from your pet project's source code (e.g. `Write extensive user documentation for this project in markdown`)
  > **CAUTION:** ALWAYS REVIEW IT!
- Review a core feature. (e.g. `Output in markdown a detailed analysis of how this code handles the challenge of running SQLite queries from a Python asyncio application. Explain how it works in the first section, then explore the pros and cons of this design. In a final section propose alternative mechanisms that might work better.`)

## LLMs amplify prior experience

Blank-page problem for AI prompts must be solved by human. Ask the right initial question, and ask the right follow-ups questions. To ask the right questions the engineer will need years of experiences. This has been already been mentioned by [Simon Willison, 2024-03-22, "Claude and ChatGPT for ad-hoc sidequests"](https://simonwillison.net/2024/Mar/22/claude-and-chatgpt-case-study/#not-simple)

> **It’s not the ship, it’s the captain.**

## Conclusion

You will get to see a lot of AI-related posts if you lived in 2024-2025 and followed some AI subreddits, build-in-public practitioners sharing on Twitter, Facebook about how they vibe-code an MVP SaaS using AI, or building 7749 AI Wrappers, ...

My advice is to not give a f\*\*k (do not be FOMO) and choose 1 to 2 learning sources with practical examples. I only have one and that is [Simon Willison's Newsletter](https://simonw.substack.com/) that I received every week and specific contents on [Simon Willison's Weblog](https://simonwillison.net/).
