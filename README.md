# slide-visual-reviewer

A Claude Code skill that verifies slide visual quality from PDF/PNG output. Detects text overflow, element overlap, margin issues, and other layout problems via visual inspection of PNG images.

## What it does

- **PNG generation**: Converts slide source (Marp Markdown, PPTX, etc.) to PNG images for inspection
- **Visual checks**: Detects text wrapping issues, element collision, insufficient/excessive margins, color contrast problems
- **Severity classification**: Categorizes issues by severity (Critical / Major / Minor) for prioritized fixing
- **Report generation**: Outputs a structured review report with screenshots and recommended fixes

## When to use

- After Marp PDF/PNG export — verify slides render correctly before publishing
- After PPTX edits — confirm visual integrity after batch changes
- As a sub-skill: invoked from `marp` or `pptx` QA workflows
- Standalone: when you suspect specific layout issues on existing slides

## When not to use

- Content-level review (wording, structure, story arc) → use a `presentation` or `review-content` skill
- Marp/PPTX file editing itself → use the `marp` or `pptx` skill

## Installation

Install via the eruto-skills marketplace:

```
/plugin marketplace update eruto-skills
/plugin install slide-visual-reviewer@eruto-skills
```

Or clone this repo directly into your project's `.claude/skills/` directory.

## Requirements

- For Marp slide PNG generation: a Marp CLI script accessible in the host project (e.g., `tech/marp/scripts/marp-validate.js --png` in the presentations monorepo)
- For PPTX PNG generation: LibreOffice (`soffice`) available in PATH

## License

MIT. See [LICENSE](LICENSE).
