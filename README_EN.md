# xiaohongshu-pilot

> A full-chain agent pipeline for Xiaohongshu (RED) operations: topic research → Gemini copywriting → banned-word scanning → official rule compliance check → de-AI human-tone polishing → Jimeng image generation → publishing via xiaohongshu-mcp → pulling data via MCP for retrospective analysis that feeds back into topic selection.

## Introduction

This is a WorkBuddy / CodeBuddy / Claude Code Skill that turns "publish a Xiaohongshu note" — deceptively simple, actually full of traps — into a seven-step pipeline that is inspectable and reversible at every stage. Each step produces a checkable intermediate artifact (topic report, draft copy, banned-word diff table, compliance checklist, polished draft, cover images, retrospective report), and the pipeline only advances after user confirmation.

**What problems it solves:**

- Topic selection by gut feeling → driven by competitor research plus viral-pattern analysis, so every topic has evidence behind it;
- AI-written copy that reeks of "AI tone" → a dedicated step removes AI-isms and adds human texture, keeping edits within 30% so the viral structure survives;
- Notes getting throttled or taken down → ships with a ten-category banned-word library, the official Community Guidelines 2.0 rulebook, and a pre-publish self-check list;
- Forgetting the AI-content declaration for generated images → enforced as a hard red line;
- Publish-and-forget → 24–48 hours after publishing, the pipeline pulls engagement data, produces a retrospective, and writes optimizations back into the topic library — a closed growth loop.

**Who it's for:** individual creators or small teams running Xiaohongshu content operations through WorkBuddy / CodeBuddy / Claude Code — especially creators in the AI niche.

**Trigger phrases:** 发小红书, 小红书笔记, 小红书选题, 小红书运营, 小红书复盘, xhs.

## Features

- **Seven-step closed-loop pipeline** (SKILL.md): service self-check → topic selection → copywriting → banned-word scan → rule compliance → polishing → image generation → publish + retrospective. Every step has a defined deliverable and a confirmation gate.
- **Research-fused topic selection**: uses xiaohongshu-mcp `search_feeds` (sorted by most-liked, past week) to pull the niche's TOP 10 notes and `get_feed_detail` for details, extracting recurring angles, title patterns, cover styles, and real pain points from comment sections; combined with viral title formulas (numeric / suspense / identity-projection), it produces a Topic Report of 3–5 candidates with evidence and a recommendation.
- **Gemini copywriting**: calls Gemini through the local New API gateway (OpenAI-compatible), feeding the topic report + user persona + viral formulas; outputs a title (≤18 chars for headroom), body (≤900 chars), 5–8 hashtags, and a storyboard for image generation; explicitly requires the Xiaohongshu voice (colloquial, moderate emoji, human details, no translation-tone).
- **Ten-category banned-word scan** (references/banned-words.md): absolute claims, false promises, inducement marketing, superstition, unearned authority endorsements, financial/investment claims, off-platform traffic diversion (including homophone variants like V / 薇 / 扣1), medical/efficacy overreach, exemption scenarios, and AI-review gray zones. Every hit gets an "original → rewritten" diff table.
- **Official rule compliance** (references/compliance-rules.md): checks against the Community Guidelines 2.0 three-pillar red lines, the 12 banned marketing tactics, account-matrix rules, the AI-content enforcement categories, and high-risk industry add-ons; ticks through the pre-publish self-check list item by item; if rules have changed, it runs a WebSearch to supplement before validating.
- **De-AI polishing**: strips corporate buzzwords ("赋能", "抓手", "总而言之"), breaks mechanical parallelism, adds colloquial short sentences and personal lived details (asks the user for 1–2 real anecdotes to embed), tolerates imperfection; re-scans banned words after rewriting.
- **Jimeng image generation**: invoked via the jimeng-generate skill, taking the storyboard plus the cover style found in research; at least 2 cover variants are produced for the user to pick; images are saved to the project directory with absolute paths recorded.
- **Safe publishing strategy**: `publish_content` defaults to `visibility=仅自己可见` (private) for the first publish; the user reviews in the App and **manually ticks the AI-content declaration** (MCP cannot auto-tick it) before switching to public; supports `schedule_at` timed publishing (1 hour to 14 days); logs title, publish time, and feed_id into the project's `.workbuddy/memory/` daily log.
- **Data retrospective loop**: 24–48 hours after publishing (or when the user says "复盘小红书"), pulls likes/saves/comments/shares via `user_profile` + `get_feed_detail`, compares expectation vs. reality, and produces a Retrospective Report (data table + 3 actionable optimizations) that feeds back into the topic library.

## How It Works / Tech Stack

- **Skill form**: pure prompt-engineering Skill — `SKILL.md` defines the pipeline and red lines; three documents under `references/` provide the word library, rulebook, and API cheat sheet. No executable scripts.
- **xiaohongshu-mcp local service** (references/mcp-api.md, v2.5.0, xpzouying/xiaohongshu-mcp): HTTP MCP protocol at `http://localhost:18060/mcp`; Windows binaries (login tool + MCP server); cookies live next to the exe and stay valid long-term; exposes 13 tools (login, image-text/video publishing, search, detail, comments, likes, favorites, user profile, etc.). Sandboxed shells reclaim the process, so it must be launched with a sandbox-exempt background invocation, then verified with `curl`.
- **New API gateway**: `http://localhost:3000/v1/models`, OpenAI-compatible, provides the Gemini models for the copywriting step.
- **Jimeng image generation**: reuses the jimeng-generate skill (also routed through the New API gateway).
- **Hard limits**: title ≤20 chars, body ≤1000 chars (publishing fails beyond these); feed_id and xsec_token always come in pairs — both are required; local absolute image paths are the most reliable; video must be a local path, recommended <1GB.

## Installation & Usage

### Installation

Copy this repository's contents into your skills directory, keeping the folder name `xiaohongshu-pilot`:

- WorkBuddy / CodeBuddy: `~/.workbuddy/skills/xiaohongshu-pilot/`
- Claude Code: `~/.claude/skills/xiaohongshu-pilot/`

Restart the session; the skill will then match on its trigger phrases automatically.

### Prerequisites (Step 0 service self-check)

1. **xiaohongshu-mcp service**: if `curl http://localhost:18060/mcp` doesn't respond, start `xiaohongshu-mcp-windows-amd64.exe` (in sandboxed environments, launch with `dangerouslyDisableSandbox=true` + `run_in_background=true`, then verify with curl).
2. **Login status**: call `check_login_status`; if not logged in, run `xiaohongshu-login-windows-amd64.exe` to pop a QR code for the user to scan (first run downloads a ~150MB headless browser — keep the network up). This is the only step requiring user action.
3. **New API gateway**: `curl http://localhost:3000/v1/models` and confirm a `gemini-*` model is available.
4. **Jimeng service**: self-check per the jimeng-generate skill.

### Daily use

Just say "发小红书", "帮我想个小红书选题", or "复盘小红书" to enter the pipeline. A typical run:

1. The skill produces a Topic Report (3–5 candidates + viral evidence + a recommendation) — you pick one;
2. Gemini drafts → banned-word diff table → compliance checklist → polished draft, confirmed step by step;
3. Jimeng produces at least 2 cover variants — you pick one;
4. Before publishing you see the final copy + images + checklist; on confirmation it publishes as "private only";
5. You review in the App and tick the AI declaration (Note → Settings → Content type declaration), then switch to public;
6. 24–48 hours later, say "复盘小红书" to get the Retrospective Report.

## Project Structure

```
xiaohongshu-pilot/
├── SKILL.md                        # Skill definition: the seven-step pipeline + red lines (YAML frontmatter with triggers)
├── README.md                       # Chinese README (detailed)
├── README_EN.md                    # This file
└── references/
    ├── banned-words.md             # Ten-category banned/sensitive word library (2024–2026, with exemptions & AI gray zones)
    ├── compliance-rules.md         # Official rulebook (Guidelines 2.0, 12 banned marketing tactics, matrix rules, AI enforcement, checklist)
    └── mcp-api.md                  # xiaohongshu-mcp local service cheat sheet (startup, 13-tool table, hard limits)
```

## Notes & Caveats

- **Publishing red lines (non-negotiable)**: the final copy + images + checklist results must be shown to the user and confirmed before any publish action; no multi-account matrix posting of identical content (official red line); no off-platform traffic-diversion language of any kind; AI-generated content must come with a reminder to tick the AI declaration.
- **AI declaration is mandatory**: images generated by Jimeng must be declared as "this note contains AI-synthesized content" at publish time; getting caught undeclared is judged as "AI fake persona / AI ad marketing" — throttling or even account ban; MCP cannot tick it automatically, so it must be done manually after publishing.
- **Hard length limits**: 20-char title, 1000-char body — exceeding either fails the publish; that's why the copywriting step targets ≤18 / ≤900 for headroom.
- **Tight environment coupling**: depends on the local Windows xiaohongshu-mcp binaries, the New API gateway (127.0.0.1:3000), and the jimeng-generate skill; the full pipeline needs all three.
- **Sandbox limitation**: the MCP service gets reclaimed inside sandboxes and must be launched as a sandbox-exempt background process.
- **Cost of violations**: a single violation means demotion (~80% recommendation traffic cut) + takedown; 3 accumulated violations mean a 7–30 day posting ban; recovering weight after throttling takes ~45 days on average — compliance steps are not ceremony.
- **Rule freshness**: the references word library and rules were compiled on 2026-09-15; platform rules change often, so Step 4 supplements with a WebSearch when newer rules exist.

## License

MIT License

## Author

sheen945
