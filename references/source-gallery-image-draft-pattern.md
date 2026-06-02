# Source-gallery image draft pattern

Use this when a news source page includes a photo gallery or source-page image URLs and the user wants a WeChat news-account draft with externally sourced, story-matched images.

## Pattern

1. Prefer source/news/official gallery images over generic stock images.
   - For ChinaNews-style gallery pages, inspect the HTML for `i2.chinanews.com.cn/simg/hd/...jpg` URLs and nearby captions such as `中国商火 供图`.
   - Record both the image URL and the gallery/source page URL.
   - Caption inline images with clear attribution, e.g. `图源：中新网 / 中国商火供图。...`.

2. Download and normalize before WeChat upload.
   - Some source images are large or inconsistently encoded; save them as JPEG.
   - Use PIL to convert to RGB, cap the longest side around 1280px, and progressively lower JPEG quality until under roughly 950 KB.
   - This keeps `media/uploadimg` and `material/add_material` less likely to fail on size/format constraints.

3. Upload sequence.
   - `media/uploadimg` for inline image URLs.
   - `material/add_material` with `type=thumb` for the cover.
   - `draft/add` using UTF-8 JSON bytes (`json.dumps(..., ensure_ascii=False).encode('utf-8')`) and `Content-Type: application/json; charset=utf-8`.
   - Never call `freepublish/submit` unless the user explicitly asks to publish.

4. Verify with `draft/get`.
   - Confirm title, author, digest, image count, thumb presence, and `errcode`.
   - For this user's news workflow, also check that the opening paragraphs do not contain the engagement sentence; the engagement sentence belongs only near the ending.
   - Check the HTML contains the expected source attribution string such as `图源：中新网`.

## Minimal verification fields

```json
{
  "publish_status": "draft_created",
  "draft_id": "...",
  "draft_get_verification": {
    "title": "...",
    "author": "逆流手札",
    "digest": "...",
    "img_count": 3,
    "has_thumb": true,
    "errcode": null,
    "contains_opening_engagement": false,
    "contains_source_note": true
  }
}
```

## Pitfalls

- Do not treat a source-gallery image as permission-cleared just because attribution exists; mark it as news-source image with medium risk unless the license is explicit.
- Do not fall back to Pexels/Unsplash while a directly relevant source/news gallery is available.
- Do not use assistant-created screenshots/charts as substitutes for source images in this workflow.
