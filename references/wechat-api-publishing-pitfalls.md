# WeChat API publishing pitfalls

Concise session-derived notes for publishing WeChat Official Account drafts/articles via API.

## Draft-only mode is the user's preferred default

The user wants a ready-to-review WeChat draft created directly after the article is prepared. Do not stop at “I can create a draft if you approve”; create the draft and report the returned draft `media_id`. Still do not call `cgi-bin/freepublish/submit` unless the user explicitly asks to re-enable automatic publishing.

Successful draft-only checkpoints:

1. `cgi-bin/token` returns `access_token` / `expires_in`.
2. `cgi-bin/material/add_material?type=thumb` returns `media_id` for the cover image.
3. `cgi-bin/draft/add` returns draft `media_id`.
4. Optional verification: `cgi-bin/draft/get` or `draft/batchget` returns normal UTF-8 Chinese title/digest/content.

Report the draft `media_id` and tell the user the draft is ready for manual review/publishing in the WeChat backend.

## Successful auto-publish checkpoints

Only use this path if the user explicitly asks to auto-publish again. A healthy auto-publish attempt should progress in this order:

1. `cgi-bin/token` returns `access_token` / `expires_in`.
2. `cgi-bin/material/add_material?type=thumb` returns `media_id` for the cover image.
3. `cgi-bin/draft/add` returns draft `media_id`.
4. `cgi-bin/freepublish/submit` returns `publish_id`.
5. Optional: `cgi-bin/freepublish/get` returns publish status / article info.

Report the deepest successful checkpoint. Do not claim publication if only the draft step succeeded.

## UTF-8 JSON and false mojibake

When creating Chinese WeChat drafts from Python, send JSON as explicit UTF-8 bytes instead of relying on implicit request encoding:

```python
def wechat_post_json(url, token, payload, timeout=60):
    body = json.dumps(payload, ensure_ascii=False, separators=(',', ':')).encode('utf-8')
    return requests.post(
        url,
        params={'access_token': token},
        data=body,
        headers={'Content-Type': 'application/json; charset=utf-8'},
        timeout=timeout,
    )
```

When verifying drafts, beware that WeChat may respond with `Content-Type: text/plain` and no charset. Python `requests` may set `response.encoding` to `ISO-8859-1`, making `response.text` look like mojibake (`æ...`). Decode the raw bytes explicitly:

```python
text = response.content.decode('utf-8')
data = json.loads(text)
```

Then inspect `title`, `digest`, and the start of `content`. Do not tell the user the draft is corrupted unless the UTF-8-decoded payload is actually corrupted.

## 40164: invalid ip ... not in whitelist

Example:

```json
{
  "errcode": 40164,
  "errmsg": "invalid ip 142.249.36.62 ... not in whitelist"
}
```

Meaning: the current outbound IP is not in the Official Account developer IP whitelist. It does not prove the AppID/AppSecret are wrong.

Action:

- Extract and report the exact IP from the error.
- Tell the user to add it in WeChat Official Account backend: 设置与开发 → 基本配置 / 开发者设置 → IP 白名单.
- Retry the same script after the whitelist update.

## 45003 / 45004: title or description too long

Examples:

- `45003 title size out of limit`
- `45004 description size out of limit`

Meaning: WeChat field byte limits were exceeded. Chinese characters count as multiple bytes, so a visually short Chinese title/summary may still exceed the API limit.

Action:

- Shorten `title` aggressively first. Prefer a compact factual title under roughly 10-12 Chinese chars when debugging.
- Shorten `digest` / `description` aggressively. When blocked, use a short neutral digest such as `空间站样品返回，科研开始周转。`.
- Retry from the existing article content; no need to rewrite the whole article.

## Cover thumbnail vs in-article images

WeChat treats these as two separate things:

- `cgi-bin/material/add_material?type=thumb` uploads the article cover thumbnail and returns `thumb_media_id`. This makes the cover available in draft metadata, but it does **not** place an image inside the article body.
- `cgi-bin/media/uploadimg` uploads an image for use inside article HTML and returns a URL. To show an image in the draft body, insert `<img src="...">` using this returned URL before calling `draft/add`.

For draft-only articles that should visually contain an image, do both:

1. Upload cover via `media/uploadimg`, prepend the returned URL to `content` as an `<img>` plus attribution caption.
2. Upload the same or another image via `material/add_material?type=thumb`, set `thumb_media_id` in the draft payload.

Avoid using a placeholder/generated abstract cover if the user says it looks strange. For this user, all images must be externally sourced official/open/licensed/public materials with attribution; do not use AI-generated images, self-made charts, self-made screenshots, or other assistant-created images. Prefer Wikimedia Commons, official newsroom/government/project images, or clearly licensed public materials. Include 2-3 in-article images when feasible, not just a cover thumbnail. If image sourcing/download is blocked, use fewer images or stop with a clear blocker instead of generating or making images yourself.

## Author field

Do not hardcode the author as `blankxxc`. For this user’s WeChat workflow, the public-account/author name is `逆流手札`; allow `WECHAT_ACCOUNT_NAME` to override when explicitly set. If working in a different account context and the name is unknown, leave the author field empty and ask the user for the exact公众号名 before creating the final draft.

## 48001: api unauthorized on freepublish/submit

Example:

```json
{
  "errcode": 48001,
  "errmsg": "api unauthorized"
}
```

If token, cover upload, and draft creation succeeded, then credentials/IP/material permissions are mostly OK. The blocker is specifically the formal publish/group-send permission for `freepublish/submit`.

Action:

- Report that the draft was created successfully and provide the draft `media_id`.
- Tell the user to check WeChat backend: 设置与开发 → 接口权限.
- Look for 发布能力 / 群发接口 / 群发与通知 / 草稿发布 / freepublish permissions.
- If the account is a personal or unverified subscription account, API auto-publish may not be available; the durable fallback is automatic draft creation plus manual backend publishing.
- If using a third-party platform authorization flow, ensure the authorization scope includes group-send/publish plus material/draft permissions.

## Windows path pitfall

On this user's Windows/Git-Bash setup, shell commands can use MSYS paths like `/c/Users/...`, but Python scripts that create deliverables should write to native paths like `C:/Users/blankxxc/...`. This keeps Hermes file tools and Windows apps aligned.
