# Policy-source news draft fallback pattern

Use this when the selected news topic is an official policy/regulation page whose source page is text-heavy and has no usable article images.

## When this applies

- The anchor source is a government/regulator/policy page.
- The page is authoritative for facts but has no story-matched image, gallery, or downloadable press photo.
- The user still wants a ready-to-review WeChat draft.

## Pattern

1. Keep the article facts anchored to the official page. Do not replace the official source with commentary just because commentary pages have images.
2. Search for source/news/official images first. If none are usable, document that the original source page had no suitable image.
3. Use open-licensed external images as fallback, preferably Wikimedia Commons via API or `Special:FilePath`.
   - Query Commons with topic-adjacent visual concepts, not fabricated event imagery.
   - Download and verify `Content-Type: image/*`.
   - Normalize to JPEG under roughly 950 KB before WeChat upload.
4. Caption each image with clear attribution, e.g. `图源：Wikimedia Commons。...`.
5. Add a short public `信息来源`/source note if needed, but do not include draft-operation status such as “本文只创建草稿” or “不自动发布” in the public body.
6. Verify after `draft/get`:
   - `img_count` matches uploaded inline images.
   - `contains_opening_engagement` is false.
   - `contains_source_note` is true when using fallback images.
   - `contains_draft_status_note` is false.

## Good default wording for result JSON

```json
{
  "image_policy_note": "Official source page had no usable article images; used open-licensed Wikimedia Commons fallback. No AI/self-made images or screenshots used."
}
```

## Pitfalls

- Do not use AI-generated visuals, assistant-made charts, or screenshots as substitutes for a policy page with no images.
- Do not overstate what commentary/news search results prove. If you only used them to confirm heat or framing, say so in editor metadata and keep claims anchored to the official source.
- Do not put publish/draft status notes in WeChat正文; report them only in JSON or assistant summary.
