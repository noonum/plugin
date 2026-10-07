# Noonum plugin

Build and analyze thematic investment strategies with [Noonum](https://noonum.ai). A strategy is a natural-language objective that Noonum turns into a ranked list of companies, each with the reasoning and evidence for its inclusion.

The plugin bundles the Noonum connector (`https://mcp.noonum.ai/mcp`) and two skills:

- **build-strategy** turns a thesis, an article URL, or pasted text into a new strategy, runs it, and presents the ranked companies.
- **strategy** works with existing strategies: holdings, why a company is in or out, outliers, sector breakdowns, comparisons, performance, refinement, filters, factsheets, CSV export, and archiving.

## Install

### Claude Code

```bash
claude plugin marketplace add noonum/plugin
claude plugin install noonum@noonum
```

Then run `/mcp`, pick `Noonum`, and sign in with your Noonum account.

### Claude.ai and Cowork

In Customize → Plugins, add the `noonum/plugin` GitHub repository as a marketplace and install Noonum. Then open the plugin's Connectors tab and click Connect next to Noonum to sign in.

## Examples

```
/noonum:build-strategy companies supplying the GLP-1 manufacturing chain
/noonum:strategy Semiconductors
/noonum:strategy why is Apple in Semiconductors?
```

You can also just ask: "show my strategies", "are there outliers in AI Memory & Storage?", "compare Uranium and Nuclear Energy".

## Credits

Each run spends one credit of your Noonum plan; a run that fails is not charged. The skills ask before spending a credit, archiving a strategy, or generating a factsheet. Reading results, changing filters, and renaming are free.

## Data

The plugin sends your objectives, questions, and strategy edits only to the Noonum connector (`mcp.noonum.ai`), under your Noonum account. Noonum handles them as described in its [Privacy Policy](https://noonum.ai/privacy), with the providers listed on its [Subprocessors](https://noonum.ai/subprocessors) page. When you give an article URL, your AI assistant reads the page itself; only the strategy drafted from it reaches Noonum. Results are research data, not investment advice. Use of Noonum is subject to its [Terms of Service](https://noonum.ai/terms).

## Support

Setup and troubleshooting: [docs.noonum.ai](https://docs.noonum.ai/base/agentic/mcp-client-setup). Questions and security reports: hello@noonum.com.

## Development

- Commits follow [Conventional Commits](https://www.conventionalcommits.org/).
- Bump `version` in `.claude-plugin/plugin.json` and `plugin.json` on every release.
- `plugin.json`, `mcp.json`, and `assets/` package the plugin for the OpenAI plugin directory. Build the upload from a commit, leaving out the Claude manifest:

  ```bash
  git archive -o noonum-openai.zip HEAD plugin.json mcp.json assets skills LICENSE
  ```

- Validate the manifests and the plugin directory checks:

  ```bash
  claude plugin validate .claude-plugin/plugin.json --strict
  claude plugin validate . --profile directory --strict
  ```

- Try local changes with `claude --plugin-dir .`.

### Evals

The suite under `evals/` runs the skills against fixed connector responses in `evals/mocks/Noonum/`, so it spends no credits and needs no sign-in.

```bash
# one case while iterating
claude plugin eval . --case build-confirm-gate --runs 1 --ablation none --judge-model sonnet --no-publish

# full suite: 3 runs per case, with and without the plugin
claude plugin eval . --judge-model sonnet --threshold 0.9
```

Use a Sonnet judge: the default Haiku judge fails replies that cite 2026 evidence dates. Results are written to `evals/results/` (gitignored).

## License

[Apache-2.0](LICENSE)
