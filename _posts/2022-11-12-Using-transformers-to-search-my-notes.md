Following up on the exploratory work I did for searching recipes with OpenAI. I quickly found this [blog](https://medium.com/@nils_reimers/openai-gpt-3-text-embeddings-really-a-new-state-of-the-art-in-dense-text-embeddings-6571fe3ec9d9) 
that benchmarked OpenAI's results against other, open-source, alternatives. Turns out you can do this stuff for much cheaper than I'd believed.

I've been taking a lot of notes on books and papers I've been reading about climbing physiology. I've got a Google doc that's about 120 pages now of bullet points.
I lose info in it a bit because docs only supports ctrl-f / exact keyword search. I got into the habit of using specific terms when reading about a specific theme
like "collagen synthesis" as a shoddy tagging system. 

Inspired by [this tweet from Dwarkesh Patel](https://twitter.com/dwarkesh_sp/status/1586853685119700992), I tried to do the same for my own notes: use 
one of these models to embed my notes into a semantic space to enable fuzzier, semantic search.

It was actually really easy!

I only had to download my notes as a txt, do a very small amount of formatting to break them into separate files, and then send them to the model.

```python
from sentence_transformers import SentenceTransformer, util

papers = {'paper1': 'summary1', ... 'paperN': 'summaryN'}

model = SentenceTransformer('msmarco-distilbert-base-v4')

docs = [(p[0], ' '.join(p[1])) for p in papers.items()]
embeddings = model.encode([s[1] for d in docs])

while True:
    q = input('Query: ')
    q_embedding = model.encode(q)

    scores = []

    for i, e in enumerate(embeddings):
        sc = util.cos_sim(e, q_embedding)
        scores.append((sc, i))

    scores.sort(reverse=True)

    for result in scores[:10]:
        paper = docs[result[1]][0]
        summary = papers[paper][:100]

        print(paper, result[0])
        print(summary)
```

This prints out the top matching papers/notes. Some of them are long, like full textbooks, so the search is unfortunately less useful for them, 
but I figure I could probably just break up long notes into smaller subsections and redo the embedding on the subsections, kind of like how google highlights 
the specific match it finds within the page.

Nothing super impressive on my end, but I think it's amazing how accessible this is now. I remember trying to setup a search engine for a Q&A site 
for a hackathon several years ago, and the easiest-out-of-the-box solution was postgres's trigram search. Otherwise we were looking at Elasticsearch.

I'm not sure how scalable embedding like this will be, but it feels like a powerful, personal alternative to google search now. 
