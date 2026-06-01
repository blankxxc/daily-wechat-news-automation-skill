# Direct WeChat draft creation pattern

Session-derived pattern for this user's WeChat Official Account article workflow.

## User preference

After a draft article is written, directly create a WeChat backend draft for review. Do not ask an extra “should I create the draft?” confirmation. The safety boundary is: create draft yes, auto-publish no unless explicitly requested.

Required defaults:

- `author`: `WECHAT_ACCOUNT_NAME` if set, otherwise `逆流手札`.
- Publishing mode: draft-only.
- Body images: 2-3 inline images when feasible.
- Engagement sentence: add this exact sentence near both the opening and ending of every news article: `如果觉得说得有道理，可以关注、点赞、评论、推荐。`
- Humanized writing: before creating the draft, remove obvious machine/LLM traces. Avoid formulaic phrases like “值得注意的是”, “从某种意义上说”, “这背后反映出”, neat three-part summaries, generic transitions, and empty grand conclusions. Prefer concrete details, varied rhythm, short mobile paragraphs, and a clear human judgment while keeping every factual claim sourced.
- Image policy: no AI-generated images; use official/open/licensed/public external materials only. Do not use self-made charts, assistant-created screenshots, or deterministic fallback diagrams for this user's public-account workflow unless the user explicitly overrides the image policy in the current session.

## API sequence

1. Get token via `cgi-bin/token`.
2. Prepare WeChat-compatible HTML with short mobile paragraphs.
3. Upload every inline image via `cgi-bin/media/uploadimg` and insert returned URLs into `<img src="...">` tags.
4. Upload cover via `cgi-bin/material/add_material?type=thumb` and set returned `thumb_media_id`.
5. Create draft via `cgi-bin/draft/add` using explicit UTF-8 JSON bytes.
6. Verify with `cgi-bin/draft/get`, decoding `response.content` as UTF-8.
7. Report draft `media_id`, author, title, inline image count, cover presence, and that `freepublish/submit` was not called.

## No self-made image fallback

For this user's public-account workflow, do not fall back to self-made charts, assistant-created screenshots, or deterministic diagrams when image downloads fail. If an image source fails or rate-limits, retry with another official/open/licensed/public external image source and update attribution. If no safe external image is available, use fewer images or stop at a draft/no-publish report rather than inventing visuals.

## External image download fallback

For Wikimedia Commons images, prefer the Commons redirect form first instead of raw `upload.wikimedia.org/.../thumb/...` URLs, because raw thumbnail URLs may intermittently return `429 text/html` while `Special:FilePath` succeeds:

- `https://commons.wikimedia.org/wiki/Special:FilePath/<URL-encoded FileName>?width=1200`

Recommended order:

1. Store the human-review source page separately, e.g. `https://commons.wikimedia.org/wiki/File:Example.jpg`.
2. Download via `Special:FilePath/<FileName>?width=1200` and verify HTTP 200 plus `Content-Type: image/*`.
3. If that fails, resolve `thumburl` through the Commons API (`action=query&prop=imageinfo&iiprop=url|mime&iiurlwidth=1200`) and retry after a short delay.
4. If both fail, switch to another official/open/licensed external source such as Pexels/Unsplash/Wikimedia, update the article's image attribution block, and proceed only after the downloaded response has an image content type.

Do not treat a transient Wikimedia 429 as permission failure; the durable lesson is to use `Special:FilePath`/API fallback and content-type verification.

## IP whitelist failure retry pattern

If `cgi-bin/token` fails with `40164 invalid ip ... not in whitelist`, do not treat it as bad credentials and do not imply a draft was created. Preserve the generated article Markdown, downloaded external images, and publish-result JSON; update the article's `发布结果` block with the exact WeChat error and outbound IP. Tell the user to add that IP in 微信公众号后台 → 设置与开发 → 基本配置 → IP 白名单, then rerun the same creation script after the whitelist is updated. This is especially useful because image sourcing and article writing may already have succeeded before the token call.

## Verification JSON shape

Useful final check to print/report:

```json
{
  "draft_id": "...",
  "publish_status": "draft_created",
  "draft_get_verification": {
    "title": "...",
    "author": "逆流手札",
    "digest": "...",
    "img_count": 3,
    "has_thumb": true,
    "errcode": null
  },
  "publish_note": "按用户要求只创建公众号草稿，不调用 freepublish/submit 自动发布接口。"
}
```
