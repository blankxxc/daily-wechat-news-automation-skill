---
name: daily-wechat-news-automation
description: Use when building or running a scheduled workflow that turns yesterday's news hotspots into a sharp, niche Chinese WeChat public-account article with title research, image sourcing, fact checks, and guarded publishing.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [wechat, news, cron, publishing, editorial, chinese-writing, social-media]
    related_skills: [blogwatcher, humanizer, google-workspace]
---

# Daily WeChat News Automation

## Overview

This skill governs an end-to-end daily editorial automation workflow for Chinese WeChat public-account publishing. It collects yesterday's news hotspots, chooses a topic with a sharp but defensible niche angle, studies comparable low-follower viral titles, sources or generates images with attribution, drafts and reviews the article, then optionally creates and publishes a WeChat draft through the Official Account API.

The workflow should produce a publishable article, not a generic news digest. The default editorial target is: 热点足够大、角度足够小、判断足够锐利、证据足够硬、表达不标题党。

User-specific hard constraint: do not use AI-generated images. Images must be sourced from official/licensed/open/public materials, public screenshots, or self-made charts based on real data, with attribution and risk review.

Default stance: generate a high-quality reviewed draft. User preference for this workflow: after drafting an article, directly create a ready-to-review WeChat draft without asking for an extra confirmation step; do not call `freepublish/submit` or otherwise auto-publish unless the user explicitly asks to enable automatic publishing again in that session.

## When to Use

Use this skill when:

- The user wants a scheduled daily article based on yesterday's news hotspots.
- The user wants a sharp, niche, opinionated Chinese WeChat article rather than a neutral summary.
- The workflow needs title research from comparable viral posts.
- The article needs cover/inline images with source attribution.
- The user wants draft creation or automatic publishing to a WeChat Official Account.
- A cron job needs a self-contained prompt for recurring article production.

Do not use this skill for:

- Minute-by-minute breaking news alerts.
- Pure SEO/content-farm output.
- Unverified rumor amplification.
- Legal, medical, investment, criminal, disaster, or minor-related claims without authoritative sources and human review.
- Fully unattended publishing unless the user explicitly chooses it and provides credentials.

## Recommended Architecture

Prefer a staged pipeline instead of one monolithic cron run:

1. **Collect yesterday's news** — RSS, search, hot lists, official notices, and source pages.
2. **Cluster and verify events** — deduplicate URL/title/event variants and extract source-backed claims.
3. **Select topic** — score candidate events for heat, niche angle, source confidence, structural significance, and risk.
4. **Mine title references** — collect comparable low-follower/high-performing titles; extract patterns without copying.
5. **Draft article** — write one thesis-driven WeChat article with source-grounded facts.
6. **Source images** — select official/licensed/open/public images, public screenshots, or self-made charts; record attribution and risk.
7. **Review** — fact check, title-risk check, image-rights check, WeChat formatting check.
8. **Publish** — create draft, then publish only if the configured gate passes.

For Hermes cron, split into at least two jobs when possible:

- Collector job around 07:00-07:30 local time: outputs structured candidates JSON.
- Writer/publisher job around 08:00-08:30 local time: consumes collector output via `context_from` and produces/publishes the article.

## Source Strategy

Use multiple source classes:

### Fact Anchors

Prefer these for claims:

- Official government/regulator pages
- Company announcements, financial reports, exchange filings
- Court documents and policy originals
- Official statements from involved parties
- Reputable original reporting

### Heat and Public Sentiment

Use for trend detection, not as final proof:

- Weibo/Baidu/Zhihu/Toutiao/Bilibili/Douyin hot lists
- WeChat search results and article performance signals
- Industry community discussions
- Comment/high-like sentiment summaries

### Background and Niche Angle

Use to deepen interpretation:

- Historical reports
- Industry reports
- Research papers
- Comparable cases
- Company history and prior filings
- Policy context

Every candidate event should record: title, source, URL, published time, summary, source type, confidence, claims, risk flags, and why it matters.

## Candidate Scoring

Do not automatically pick the biggest headline. Prefer:

`hot enough to matter + neglected enough to say something original + sourced enough to publish`

Suggested scoring:

- Heat / public attention: 25%
- Contrarian or under-discussed angle: 25%
- Verifiability / source strength: 20%
- Niche WeChat readability: 20%
- Freshness: 10%
- Risk penalty: subtract up to 20%

Reject candidates that are pure outrage, single-source rumors, too legally risky, already over-written without a new angle, or impossible to verify.

## Title Intelligence from Low-Follower Viral Posts

Before finalizing titles, study comparable posts in the same topic class, especially small or mid-size accounts whose posts outperformed their account baseline.

Collect 10-30 comparable titles when feasible. For each, store:

- Title
- URL/source page
- Platform/account name
- Estimated account size if available
- Performance signal: reads/likes/comments/rank/reposts or qualitative evidence
- Topic class
- Why the title worked
- Pattern type: contradiction, diagnosis, hidden victim, reversal, question, cold judgment, crowd pain

Important rules:

- Extract title structures; do not copy unique wording.
- Never imply a fact that the article cannot prove.
- Avoid low-quality clickbait such as “全网炸了”, “惊天内幕”, “细思极恐”, “彻底变天”.
- Score title candidates for clickability, accuracy, niche sharpness, article fit, and title-party risk.

Useful title patterns:

- 这不是 X 的胜利，而是 Y 的退场
- 所有人都在骂 X，但真正的问题不在 X
- 昨天这件事，最刺痛人的不是结果
- 你以为这是偶发，其实它早就开始了
- 真正该担心的不是 X，而是 X 之后的默认规则

## Article Structure

Default structure:

1. **Title + subtitle** — sharp, accurate, non-clickbait.
2. **Opening hook** — begin with a concrete detail, not “近日/引发热议”.
3. **What happened yesterday** — 3-5 source-backed facts; mark uncertainty.
4. **What most people got wrong** — identify the shallow mainstream reading.
5. **Core thesis** — one sentence the reader can remember and argue with.
6. **Structural analysis** — individual, institutional/platform/business, and public-emotion layers.
7. **Who benefits / who pays** — stakeholders and tradeoffs.
8. **Ending judgment** — concrete, restrained, not slogan or chicken soup.
9. **Editor/source block** — sources, risk flags, image attributions, manual-review items.

Style:

- Simplified Chinese.
- Short mobile-friendly paragraphs.
- Sharp but restrained.
- Opinionated but sourced.
- Specific nouns over abstract slogans.
- Avoid officialese, thesis prose, and AI clichés.
- For this user's news公众号 drafts, add this exact engagement sentence near both the ending: `如果觉得说得有道理，可以关注、点赞、评论、推荐。`
- Run a humanizing pass before draft creation: remove machine-sounding scaffolding such as “值得注意的是”, “从某种意义上说”, “这背后反映出”, generic three-part summaries, formulaic transitions, and empty elevation. Prefer concrete details, varied rhythm, short mobile paragraphs, and a visible human judgment without inventing facts.

## Image Sourcing and Attribution

Every article should have a cover image and optional inline images. Image attribution is mandatory, but attribution alone does not guarantee usage rights. Prefer low-risk sources.

Priority order:

1. Official image from a government/company/project/newsroom page.
2. Licensed open image: Wikimedia Commons, Unsplash, Pexels, etc.
3. Self-made chart or screenshot of official public document/page.
4. Licensed/open public-domain visual material that fits the article tone.
5. News article image only as a medium-risk candidate requiring careful attribution and review.

Do not use AI-generated images for cover or inline article images. If no safe image is available, use fewer images, use official screenshots/self-made data charts, or stop at draft/no-publish instead of inventing visuals.

For every image record:

- `image_url` or local path
- `source_page`
- `source_name`
- `license` or permission status
- `caption`
- `attribution`
- `usage_role`: cover / inline / reference-only / do-not-use
- `risk_level`: low / medium / high

Rules:

- Do not use unclear-rights news photos as automatic cover images when a safer alternative exists.
- Avoid celebrity photos, minors, accident/disaster photos, medical/criminal scene photos, and watermarked copyrighted images.
- For sensitive social topics, prefer abstract/generated images or self-made diagrams.
- Inline image captions should include clear attribution such as `图源：XXX 官网 / XXX 公告 / Wikimedia Commons`.

## WeChat Official Account Publishing

Only use API publishing when the user has explicitly requested it and configured credentials.

Required secrets should be environment variables or profile config, never hardcoded in the skill:

- `WECHAT_APP_ID`
- `WECHAT_APP_SECRET`

Typical API workflow:

1. Get/cache `access_token`.
2. Render article Markdown to WeChat-compatible HTML.
3. Upload inline images and replace URLs with WeChat image URLs.
4. Upload cover image as material and get `thumb_media_id`.
5. Create draft article.
6. If publish gate passes, submit draft for publishing.
7. Poll publish status.
8. Return draft ID, publish ID, article URL if available, or exact failure reason.

WeChat article fields usually need:

- title
- author
- digest
- content HTML
- content source URL
- thumb media ID
- comment settings

Publishing modes:

- `draft_only` — safest; creates draft for human review. This is the user's current preferred mode for WeChat Official Account work.
- `auto_publish_if_pass` — only use if the user explicitly asks to re-enable automatic submission and the publish gate passes.
- `auto_publish` — only when the user explicitly accepts unattended publishing risk.

Recommended publish gate:

- Minimum 3 credible sources for the main topic.
- Fact score >= 80.
- Editorial score >= 80.
- Title-party risk <= medium.
- Image risk is not high.
- High-risk topic classes require human review.
- No unresolved critical source conflicts.

If the gate fails, create/save a draft or report instead of publishing.

## Output Format

Default deliverable:

```markdown
# 推荐标题

副标题：...
选题日期：YYYY-MM-DD
生成时间：YYYY-MM-DD HH:mm
发布模式：draft_only / auto_publish_if_pass / auto_publish

## 正文

...

## 备选标题

1. ...
2. ...
3. ...
4. ...
5. ...

## 标题参考模式

- 参考模式：...
- 借鉴点：...
- 未直接复制的说明：...

## 图片与来源

- 封面图：...
- 来源：...
- 授权/风险：...
- 图注：...

## 编辑备注

- 核心论点：...
- 为什么选它：...
- 小众切口：...
- 主要风险：...
- 是否建议发布：...

## Sources for editor review

1. Source — URL — key fact used
2. Source — URL — key fact used
3. Source — URL — key fact used

## 发布结果

- draft_id: ...
- publish_id: ...
- article_url: ...
- failure_reason: ...
```

## Cron Prompt Skeleton

Use a self-contained prompt for scheduled jobs. See `templates/cron-prompt.md` if present.

Minimum prompt requirements:

- State date window: yesterday 00:00-23:59 in the configured timezone.
- Require source-backed facts and no fabrication.
- Require comparable title analysis.
- Require image sourcing and attribution.
- Require WeChat publish gate before auto-publishing.
- Require a no-topic/no-publish report when evidence is insufficient.

## Common Pitfalls

1. **Writing a news digest instead of an essay.** The output must have a thesis and one main angle.
2. **Choosing the loudest topic.** Big heat without source confidence or a fresh angle is not enough.
3. **Using viral-title research as plagiarism.** Borrow patterns, not wording.
4. **Treating attribution as image permission.** Source credit reduces ambiguity but does not automatically clear copyright.
5. **Publishing screenshots or news photos automatically.** Use official/open/licensed images, public-source screenshots, or self-made charts when possible.
6. **Letting LLM invent sources, numbers, quotes, or URLs.** If not verified, mark unknown or omit.
7. **Writing “锐利” as insult.** Be precise, not abusive or conspiratorial.
8. **Forgetting WeChat HTML constraints.** Markdown is not enough for publishing; images must be uploaded/rehosted.
9. **Silent cron failure.** Always deliver a failure reason or downgraded brief.
10. **Publishing when risk gates fail.** In that case create a draft or no-publish report.
11. **Misdiagnosing WeChat `40164 invalid ip ... not in whitelist`.** This means the current outbound IP is missing from the Official Account developer IP whitelist, not necessarily that credentials are wrong. Report the exact IP from the error, tell the user to add it in the WeChat backend, and do not claim a draft/publish happened unless later calls returned IDs.
12. **Ignoring WeChat field byte limits.** `45003 title size out of limit` and `45004 description size out of limit` can occur even for visually short Chinese text because byte limits are stricter than character intuition. Shorten title/digest aggressively and retry without rewriting the article.
13. **Confusing draft success with publish success.** If `draft/add` returns `media_id` but `freepublish/submit` returns `48001 api unauthorized`, the draft exists but formal API publishing is unauthorized. Provide the draft ID and tell the user to check 设置与开发 → 接口权限 for 发布能力/群发接口/freepublish permission; unsupported personal/unverified accounts may need manual backend publishing.
13. **Mistaking response-display mojibake for stored draft mojibake.** WeChat draft APIs may return `Content-Type: text/plain` without charset, causing Python `requests` to decode response text as ISO-8859-1 and display Chinese as `æ...`. Inspect `response.content.decode('utf-8')` and/or `draft/get` before assuming the saved draft is corrupted.
14. **Letting `requests.post(..., json=payload)` rely on implicit encoding.** When creating Chinese drafts, explicitly send UTF-8 JSON bytes with `Content-Type: application/json; charset=utf-8`, `json.dumps(payload, ensure_ascii=False).encode('utf-8')`.
15. **Mixing MSYS paths with Python paths on Windows.** Shell commands may use `/c/Users/...`, but Python publishing scripts should write to native `C:/Users/...` paths so Hermes file tools and Windows apps can find the artifacts reliably.

## Verification Checklist

Before finalizing or publishing:

- [ ] Date window was computed from tool-observed current date/time.
- [ ] Topic has a concrete yesterday trigger.
- [ ] Main facts have at least 3 credible sources or are marked for review.
- [ ] Single-source or uncertain claims are not used as core argument.
- [ ] Article has one clear thesis sentence.
- [ ] It is not a generic news summary or hotspot list.
- [ ] Comparable title examples were analyzed without copying unique wording.
- [ ] Final title is accurate and non-clickbait.
- [ ] Cover/inline images have source, attribution, license/risk notes.
- [ ] High-risk images are not used automatically.
- [ ] WeChat HTML renders correctly.
- [ ] Draft creation succeeded before publish submission.
- [ ] Publish gate passed if auto-publishing.
- [ ] Result includes draft ID, publish ID, article URL, or exact failure reason.

## Supporting Files

Useful support files for this skill:

- `references/title-and-image-workflow.md` — condensed guidance for viral-title research and image sourcing/attribution.
- `references/wechat-api-publishing-pitfalls.md` — WeChat API publishing failure handling, especially IP whitelist 40164 and Windows path pitfalls.
- `references/direct-draft-creation-pattern.md` — user-specific direct draft creation flow: author default, 2-3 inline images, strict external-image-only policy, Wikimedia `Special:FilePath` retry pattern, and draft verification shape.
- `templates/cron-prompt.md` — self-contained daily cron prompt.
- `templates/review-prompt.md` — fact/editorial/title/image review prompt.
