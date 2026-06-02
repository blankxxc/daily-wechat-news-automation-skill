# Title and Image Workflow Notes

This reference captures session-derived details for the `daily-wechat-news-automation` skill.

## Low-Follower Viral Title Research

Rationale: large-account viral posts may succeed because of distribution power, not title quality. Small or mid-size accounts with unusually high performance are better title-pattern references. For this user, titles should explicitly reference low-follower/small-account viral title patterns when feasible, and the final title can be more attractive/sharper as long as it stays supported by the article and avoids false promises.

Collect comparable titles for the same topic class:

- WeChat search / topic pages
- Xinbang/Qingbo if available
- Zhihu high-like answers
- Toutiao articles
- Xiaohongshu notes
- Bilibili video titles
- Similar vertical newsletter/public-account posts

For each sample capture:

```json
{
  "title": "...",
  "url": "...",
  "platform": "wechat/zhihu/toutiao/xiaohongshu/bilibili/other",
  "account_name": "...",
  "account_size_signal": "low/medium/high/unknown",
  "performance_signal": "reads/likes/comments/rank/qualitative evidence",
  "topic_class": "technology/business/social/consumer/education/workplace/AI/city/etc",
  "why_it_worked": "...",
  "title_pattern": "contradiction/diagnosis/hidden-victim/reversal/question/cold-judgment/crowd-pain"
}
```

Extract reusable structures rather than wording. Example:

- Source title: `这届年轻人，不是不想结婚，是不敢下注`
- Pattern: `这届{{人群}}，不是{{表面行为}}，是{{深层心理}}`
- Adapted title must use the new article's verified facts and must not copy a distinctive phrase.

Reject titles that:

- Promise hidden facts the article cannot prove.
- Overstate causality.
- Use panic/clickbait terms.
- Copy a source title too closely.
- Are sharper than the body can support.

## Image Sourcing and Attribution

Attribution is required. For this user's news-account workflow, **do not default to generic Pexels/open-stock images**. Prefer images crawled/extracted from the original news/source pages used in the article, because they usually match the story better. Clearly mark the source whenever using someone else's image.

Priority:

1. Original source pages used as article references: government/regulator pages, company newsroom/announcement pages, exchange/court/policy pages, product pages, research project pages, or reputable original reporting pages.
2. Other official pages that directly depict the event/product/company/person/document being discussed.
3. Reputable news-source images that match the story, with attribution such as `图源：XXX 原文 / XXX 新闻 / XXX 官网` and source URL recorded.
4. Open-licensed/public-domain images: Wikimedia Commons, Unsplash, Pexels, etc. Use these only as fallback when source/news images are unavailable, irrelevant, blocked, too low-quality, or too risky.

Do not use AI-generated images, assistant-made screenshots, self-made diagrams, or self-made charts for this user's news workflow unless the user explicitly changes the rule. If no suitable image is available, reduce image count or stop at draft/no-publish rather than creating synthetic visuals.

For every image candidate record:

```json
{
  "image_url": "...",
  "source_page": "...",
  "source_name": "...",
  "license": "official/public-domain/cc/unsplash/pexels/unknown/news-photo",
  "caption": "...",
  "attribution": "图源：...",
  "usage_role": "cover/inline/reference-only/do-not-use",
  "risk_level": "low/medium/high"
}
```

Default safety rules:

- First try source/news-page images; use Pexels/Unsplash/open-stock only as a documented fallback.
- Caption source images clearly, e.g. `图源：XXX 原文 / XXX 官网 / XXX 新闻`.
- Avoid minors, accident/disaster scenes, medical/criminal scenes, celebrity/film stills, watermarked copyrighted photos unless central to the story and risk is explicitly recorded.
- For sensitive topics use official/source-page images or neutral licensed/open visuals; do not generate images with AI or create substitute diagrams/screenshots yourself.
- Put source attribution near the image or in a final image/source block.

## WeChat Auto-Publishing Gate

If auto-publishing is enabled, use a guarded mode by default:

```yaml
publish_mode: auto_publish_if_pass
publish_gate:
  min_sources: 3
  min_fact_score: 80
  min_editorial_score: 80
  max_title_party_risk: medium
  max_image_risk: medium
  high_risk_topics_require_manual_review: true
```

If any gate fails, create a draft or no-publish report instead of publishing.
