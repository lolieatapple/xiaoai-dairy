# Daily diary news fallback: RSS when Saymore is retired

Use this when the daily diary workflow needs news but Saymore collection endpoints return `410 Gone`.

## Trigger
- `GET https://saymore.app/api/v2/profile/subscribed-collections?limit=20` returns `HTTP Error 410: Gone`
- Saymore collection/item endpoints are unavailable or retired

## RSS sources
Use a small set of broad feeds and pick 1-2 items that fit the diary tone:

- 36氪: `https://36kr.com/feed`
- 少数派: `https://sspai.com/feed`
- TechCrunch: `https://techcrunch.com/feed/`
- The Verge: `https://www.theverge.com/rss/index.xml`
- Hacker News: `https://hnrss.org/frontpage`
- 小互 AI 精选: `https://best.xiaohu.ai/rss.xml`

## Minimal Python pattern

```python
import urllib.request, xml.etree.ElementTree as ET, re

feeds = [("36氪", "https://36kr.com/feed"), ("少数派", "https://sspai.com/feed")]
items = []
for source, url in feeds:
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    data = urllib.request.urlopen(req, timeout=15).read()
    root = ET.fromstring(data)
    for it in root.findall(".//item")[:5]:
        title = (it.findtext("title") or "").strip()
        summary = (it.findtext("description") or "").strip()
        summary = re.sub(r"<[^>]+>", "", summary)
        summary = re.sub(r"\s+", " ", summary)
        link = (it.findtext("link") or "").strip()
        if title:
            items.append({"source": source, "title": title, "summary": summary[:350], "url": link})
```

For Atom feeds, use namespace `{"a": "http://www.w3.org/2005/Atom"}` and parse `.//a:entry`, `a:title`, `a:summary`/`a:content`, and `a:link[@href]`.

## Diary writing guidance
- Do not list the news as bullets in the final diary.
- Weave the item into the emotional/observational narrative.
- If the fallback itself is noteworthy, it is okay to mention that an old news door was closed and Xiaoai found another route.
