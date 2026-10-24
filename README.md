# @databuddy/scan

Find the places in your codebase where a product analytics event is missing.

```sh
npx @databuddy/scan
```

Run it inside a Git repository, or pass a folder to scan only that folder. No account or key. Node.js 22+, or `bunx @databuddy/scan`.

```
  databuddy. / event scan

  Scan complete · 13 findings in 9 files · 14s

  app/api/stripe/checkout/route.ts:9 · GET · missing · Payments
  app/(dashboard)/pricing/page.tsx:88 · form.action checkoutAction · missing · Payments
  app/(dashboard)/dashboard/page.tsx:204 · form.action inviteAction · missing · Setup & onboarding
```

It recognises React and JSX handlers, form and server actions, Vue, Svelte and Astro components, inline HTML handlers, `addEventListener`, Next.js, Express-style and Hono route handlers, and FastAPI, Flask and Django write routes. If it finds actions in only a small share of your files, it says so rather than reporting a clean result.

## Where your code goes

The scan sends code to Databuddy's scan API (`api.databuddy.cc`), which classifies it with the Jev model on Vercel AI Gateway under **zero data retention**. It sends only the user actions it finds and the functions they call, not whole files. Swift files, and files it cannot parse, are the exception and are sent whole. **Your source is never stored or logged.** The API only counts scans, using a random run ID, the CLI version and request sizes; nothing identifies you or your repository. The CLI prints where the source is going, with the number of files, before it sends anything.

- `--dry-run` lists every file and line range that would be sent, and sends nothing. `--dry-run --json` prints the exact payload.
- `databuddy-scan <folder>` sends code only from that folder. It still reads the rest of the repository to find existing tracking, and sends that as a list of event names and file:line references, so actions tracked elsewhere are not reported as missing.
- To keep source off Databuddy entirely, set `AI_GATEWAY_API_KEY` to your own [Vercel AI Gateway](https://vercel.com/docs/ai-gateway) key. Requests then go straight to your account and Databuddy receives nothing.

Only Git-tracked files are read. See [how the scanner handles your code](https://www.databuddy.cc/docs/privacy/event-scanner). Tests, examples, symlinks and files that look like they contain secrets are skipped.

## For agents

```sh
npx @databuddy/scan --json
```

Prints every finding with its file, line, coverage (`missing`, `partial`, `covered`), product area and priority, most important first. `summary.destination` and `summary.zeroDataRetention` state where the source went. The privacy notice goes to stderr, so stdout is always one JSON document, including `{"error": ...}` on failure.

## Options

| | |
| --- | --- |
| `[path]` | Folder or file to scan. Defaults to the current folder. |
| `--dry-run` | List the lines that would be sent. Sends nothing. |
| `--json` | Print results as JSON. |

Running again is fast: results are cached in `~/.cache/databuddy/scan`, unchanged code is not sent again, and an interrupted scan resumes where it stopped. Findings are suggestions to review; the scanner never edits your code.

## Source and releases

This repository is the scanner's full source; the npm package is built from it.
Releases from 0.1.4 on are published by this repository's Release workflow with
npm provenance, so `npm view @databuddy/scan@<version> --json` and the npm page
link each version to the commit and workflow run that built it. The bundle in
`dist/cli.js` is not minified, so the shipped code can be read as well. Versions
0.1.3 and earlier were built in Databuddy's private monorepo before this
repository existed and cannot be reproduced from it.

The scan API's server is not in this repository. What it receives is fully
determined by the client code here: `src/evaluate.ts` builds every request, and
`--dry-run --json` prints the exact payload without sending it.

## Contributing

```sh
bun install
bun src/cli.ts --dry-run .   # Run from source
bun run test                 # Typecheck, build and test
```

## License

MIT
