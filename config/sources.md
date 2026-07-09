# Collector Sources

collector-agent fetches from these. Edit this file directly to add/remove sources — no code changes needed. If a source is unreachable or returns nothing useful (dead feed, blocked page, empty response), skip it and note the skip in the output — don't fail the whole run over one broken source.

## Hacker News
- Front page RSS: https://hnrss.org/frontpage (top ~30 stories)
- Higher-bar RSS: https://hnrss.org/best
- Fallback if hnrss.org is unreachable: official API — https://hacker-news.firebaseio.com/v0/topstories.json (list of IDs), then https://hacker-news.firebaseio.com/v0/item/<id>.json per story

## GitHub Trending
- Daily overall: https://github.com/trending?since=daily
- Daily, Python (AI/ML tooling skews here): https://github.com/trending/python?since=daily
- Parse repo name, description, star count. Only include repos plausibly relevant to AI/dev tools — skip unrelated trending repos (games, dotfiles, wallpaper collections, etc.)

## arXiv
- API: `http://export.arxiv.org/api/query?search_query=cat:cs.AI+OR+cat:cs.CL+OR+cat:cs.LG&sortBy=submittedDate&sortOrder=descending&max_results=25`
- arXiv posts a high volume — only include papers from the last 24–48h (check the `<published>` field) and be selective: prefer papers from known labs or with clear relevance to configured topics over routine incremental papers.

## Reddit
- r/MachineLearning: https://www.reddit.com/r/MachineLearning/.rss
- r/artificial: https://www.reddit.com/r/artificial/.rss
- r/LocalLLaMA: https://www.reddit.com/r/LocalLLaMA/.rss
- r/programming: https://www.reddit.com/r/programming/.rss
- Reddit sometimes blocks non-browser fetches (403/empty body) — if that happens, skip it rather than retrying repeatedly.

## Company / lab blogs
- OpenAI: https://openai.com/news/
- Anthropic: https://www.anthropic.com/news
- Google DeepMind: https://deepmind.google/discover/blog/
- Meta AI: https://ai.meta.com/blog/
- Hugging Face (RSS, stable): https://huggingface.co/blog/feed.xml
- These pages are often JS-rendered. If a direct WebFetch comes back empty/unusable, fall back to WebSearch (e.g. `site:openai.com/news`) to find recent posts instead of giving up on the source entirely.

## General tech RSS (fills gaps outside the above)
- TechCrunch AI: https://techcrunch.com/category/artificial-intelligence/feed/
- The Verge: https://www.theverge.com/rss/index.xml
- Ars Technica: https://feeds.arstechnica.com/arstechnica/index
