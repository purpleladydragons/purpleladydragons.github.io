I recently got access to OpenAI's beta, and I've been playing around with it. 
It's not perfect, and at times I wondered if I was just reinventing the wheel with the stuff I was building. But it is really cool. I do think it'll transform our jobs quite soon.

(Code available [here](https://github.com/purpleladydragons/seriouseats-index))

As motivation, and to keep scope limited, I wanted to build a small search engine for recipes with natural language queries. 
I had a few thoughts going into this:
1. Results on Google are terrible because recipes are notoriously SEO'd 
2. I didn't want the AI to be generating its own recipe ideas. I wanted real, human recipes
3. Navigating a specific recipe site is not always great either (see below)

<img width="1265" alt="image" src="https://user-images.githubusercontent.com/1283020/201226969-742ec9d8-bc21-4bb8-8513-8f877f21b83b.png">

*Two of the top four results for "quick hearty meat" are vegetarian dishes*

A lot of the AI examples we see recently are generative and "creative". Like I said, I wanted to get real recommendations from the AI. 
I figured I had two options: 
1. I could finetune the model with example prompts like "I want to make a creamy tomato pasta" and give it specific examples
2. I could use embeddings to basically build a document search index on top of the model

I chose 2 because building the embeddings was relatively straightforward. I figured I'd just limit my scope to seriouseats.com.
I just scraped all the recipes on seriouseats.com, extracted the text, and embedded that.

It cost me like $15 to embed about 2,000 recipes. 

After that, I was able to enter queries like "quick hearty meat" and get back results. 

```shell
$ Enter a query: hearty meat dish quick

https://www.seriouseats.com/how-to-make-vegetarian-tamale-pie
https://www.seriouseats.com/flank-steak-with-bitter-greens-and-peaches-is-a-one-pan-wonder
https://www.seriouseats.com/hearty-winter-vegetable-soup-vegan-recipe
```

Okay that's not great either unfortunately.

But if I mention meat more then it starts to get better:

```shell

$ Enter a query: hearty meat meat meat

https://www.seriouseats.com/grilling-planked-meatloaf
https://www.seriouseats.com/hoisin-glazed-cocktail-meatballs
https://www.seriouseats.com/quick-easy-ground-beef-recipe-ideas-tacos-tamale-pie-meatloaf
```

If I do the same on seriouseats.com, the results are still lackluster:

<img width="1242" alt="image" src="https://user-images.githubusercontent.com/1283020/201227788-692da3c8-34f7-4e98-a92a-2f8ead83e8fe.png">

Other cool things I was able to use OpenAI for with this:

I was able to label every URL on seriouseats.com as a recipe or something else. If you search on seriouseats.com, you don't just get recipes, but stuff like knife guides.
I was able to finetune the model with several examples and it got very good at filtering out non-recipe links.

I was able to use OpenAI to parse ingredients from the HTML with a small prompt. 
While not particularly hard to do with normal scraping techniques, it's stil annoying to extract, format, and segment the text nicely while dealing with nested
divs and spans. It took much less time and effort to just prompt the AI. I was then able to send the ingredients over to a calorie counter app.

<img width="950" alt="image" src="https://user-images.githubusercontent.com/1283020/201228051-d04052e9-1e8f-4f40-abc7-387370f8e243.png">
<img width="950" alt="image" src="https://user-images.githubusercontent.com/1283020/201228094-041729a2-8d21-4f57-9827-9ea210fd06ee.png">

I was able to use the AI to recommend ingredient substitutions. 

<img width="953" alt="image" src="https://user-images.githubusercontent.com/1283020/201228392-b6ca7915-44d4-49a8-bb6d-e15fd305772c.png">

With this, you could imagine a fuzzy recipe finder/builder based on the ingredients you have at home. 
