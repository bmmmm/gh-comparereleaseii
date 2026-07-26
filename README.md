# gh-comparereleaseii

[comparereleaseii](https://github.com/bmmmm/comparereleaseii) as a GitHub CLI
extension — fact-check release notes against the actual code diff, from the
tool you already trust with your GitHub auth.

## Install

```console
$ gh extension install bmmmm/gh-comparereleaseii
$ gh comparereleaseii restic/restic --tag v0.19.1 --html report.html
```

Requirements: Node.js ≥ 24 (the tool runs its TypeScript directly — no
build, no packages), and a judge for LLM verdicts — the
[`claude`](https://code.claude.com) CLI, an `ANTHROPIC_API_KEY`, or any
OpenAI-compatible local server. Without one, the deterministic stages still
run. The first invocation needs network access and fetches ~1 MB into the
extension's directory.

## How updates work — and what runs on your machine

The extension is a thin wrapper. On first use it clones
[bmmmm/comparereleaseii](https://github.com/bmmmm/comparereleaseii) into the
extension's own directory and checks out the **exact commit** recorded in
[`tool.pin`](tool.pin) — never a branch, never a movable tag; the wrapper
rejects anything that is not a 40-hex commit hash.

The pin follows upstream releases automatically: a
[daily workflow](.github/workflows/bump-pin.yml) resolves the latest
comparereleaseii release to its commit SHA and commits the new pin. Your
side picks that up with the normal

```console
$ gh extension upgrade comparereleaseii
```

Security properties — and their honest limits:

- **SHA-pinned**: the code that runs is identified by commit hash, auditable
  in this repo's history (`git log -- tool.pin`). A moved or re-pointed tag
  upstream cannot change what runs on your machine, and nothing updates
  behind your back mid-run — only `gh extension upgrade` moves the pin.
- **No third-party actions**: the bump workflow calls the GitHub API with
  the repo-scoped `GITHUB_TOKEN` only — no external action, no secret.
- **No install hooks, no packages**: nothing executes at install time; the
  tool itself has zero runtime dependencies.
- **The limit, spelled out**: the pin follows upstream's latest release
  unattended — it is a tamper-evident record, not a human review gate. If
  the upstream account itself were compromised, a malicious release would
  propagate within a day plus your next `gh extension upgrade`. Both repos
  have the same owner, so the trust anchor is that one account.

If the tool checkout ever gets corrupted, delete it and re-run — the
wrapper re-fetches the pinned commit:

```console
$ rm -rf ~/.local/share/gh/extensions/gh-comparereleaseii/tool
```

## Support

If this tool is useful to you, you can support development at
[ko-fi.com/bmabma](https://ko-fi.com/bmabma).

## License

GPL-3.0-or-later — see [LICENSE](LICENSE).
