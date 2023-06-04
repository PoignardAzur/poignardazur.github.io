---
layout: post
title: Rambling about Microsoft, and testing alternatives to Github Copilot.
---

*Content note - This is a bit more unstructured than my usual writing. [Jump here](#benchmarking-code-completion-models) if you only care about the model comparisons.*

I've recently been in a discussion where I argued that Microsoft had made a
strategic blunder by releasing Copilot the way they did, releasing a product
built from a large database of open-source projects while keeping the project itself very much proprietary.

To be clear, I wasn't saying that Microsoft had done something unforgiveably
immoral. I was guessing that, in the long term, that decision would not be in
Microsoft's best interests, and would in fact cost them more money than a more
open release would have, by pushing developers to use open alternatives instead.

This led me to wonder: how viable are the alternatives that _actually_ exist?
I'd kind of taken for granted that they were competently made or vaguely
comparable to the state of the art, which is a very dangerous assumption in
computer science, and especially in AI.

Before I tell you what I found (spoiler: there's a lot of range), let me give you a
quick recap of the debates around Copilot, and my hot take on them.

## Microsoft Code Simulator 2021

According to Wikipedia, Github Copilot's first alpha version came out in June
2021 (holy crap, it's been two years already?).

Copilot is a service built upon OpenAI's _Codex_ model; Codex itself is an
offshoot of GPT-3, OpenAI's groundbreaking text-generating AI. (Also, if you've
never heard of GPT, congratulation for succesfully waking up from your cryogenic
sleep!).

Copilot and Codex immediately stirred up controversy because of how they were
trained: they were given next-token prediction tasks, same as GPT, with a
corpus consisting mostly of Github Python projects, according to [the OpenAI Codex paper](https://arxiv.org/abs/2107.03374):

> Our training dataset was collected in May 2020 from 54 million public software
> repositories hosted on GitHub, containing 179 GB of unique Python files under
> 1 MB. We filtered out files which were likely auto-generated, had average line
> length greater than 100, had maximum line length greater than 1000, or
> contained a small percentage of alphanumeric characters. After filtering, our
> final dataset totaled 159 GB

(OpenAI later trained successor models with code in other languages and data
from other websites.)

Now, OpenAI did some filtering to ensure these projects were open-source, and
it's possible a great many of their authors would have consented to their
inclusion had they been asked. But instead what OpenAI did was adopt an attitude
of "better ask for forgiveness than permission", only without the "asking for
forgiveness" part.

### Nice data you got there

This attitude is by no means unique to them, and is a result of the general
incentives of machine learning: data is often easily accessible, as long as you
don't need informed consent for it. Some of the data may be copyrighted, some of
the data may be open-source-with-attribution where the nature of your work means
attribution will be impossible to give reliably, some may be
open-source-only-for-open-source projects, and some data may be in the public
domain on a website that forbids scraping in its terms of service.

Even when data is in the public domain and accessible, its author may have
released it before generative AI became available, and may not be fine with it
being used to build a tool to automate their job away; so maybe it would be more
ethical (but legally unecessary) to ask for author permission before using even
public domain data.

The point is, if you want to get consent, you need to ask _a lot_ of people for
their data. Sometimes the people who own the data forgot the data even existed,
and the linked email adress is no longer used; sometimes this is data that a lot of people
contributed to, and it's unclear who actually owns it. It's much easier to just
scrap the data and not worry about any of that.

And it's not clear this is even morally wrong? Copying isn't theft. Intellectual
property isn't a natural law, it's a system we put in place to incentivize
creating stuff, and arguably a pretty broken system. If I'm writing Spiderman
fanfiction, who am I to judge machine learning researchers?

Maybe the difference is that fanfiction is amateur, but OpenAI is trying to make money? Except I would be pretty happy selling my Spiderman-Harry Potter crossovers for money, and screw Marvel and JK Rowling if they want a cut.

### My hot take: pirate everything

In an ideal world, I don't think AI developers should be expected to ask for consent whenever they're using data for a machine learning project.

When we're taking a photograph of a crowd, we don't consider that every single person in the crowd should have a veto right on publishing the photo, especially if the crowd is blurry or distance enough that individuals can't be recognized. If you take a panorama photo of a city, you're not expected to give credit to every single architect who contributed to the buildings seen in the photo. (Or maybe you are, depending on your country's laws. But you're probably not going to.)

The individual contribution of any single person in the crowd is small enough that it's not worth tracking down any single one of them to ask for their permission. Moreover, sharing the photo doesn't take any *incentives* away from the people in the crowd. Republishing a movie without permission hurts the incentives of the people financing the movie, and makes them less likely to finance more movies; taking a photo of a crowd without permission doesn't make anyone in the crowd less likely to be in a public space ever again.

I think intellectual property in general should be seen through the lens of "does it incentivize people to make more stuff", and "are those incentives enough to justify the costs and annoyances to other people wanting to make/get stuff?". Through that lens, it's obvious that copyright terms are *way* too long, and there's an argument to be made the patents are counter-productive and [have been for a while](https://idlewords.com/2003/12/100_years_of_turbulence.htm).

Through that lens, how does machine learning fare?

I'd say pretty freaking good!

Like, it's easy to lose the forest for the trees, to complain about wrong numbers of fingers and hallucinations and alignment and intellectual property so much that your forget that we've created magic rocks that can code and draw stuff for us! Many things that would have been too hard to bother with last year are trivial today.

I follow an author who publishes various stories online; one of them is a series of worldbuilding documents (think SCP without the horror), updated daily, where each document starts with an illustration. Before AI, if the author had to commission an artist to illustrate his series, he would just have gone without illustrations. He definitely wouldn't have paid an artist to produce an image a day for a freely available story online.

Using Copilot has helped me learn about common coding patterns in JavaScript and Python; using ChatGPT has helped me start projects with technologies I was unfamiliar with, by giving me that small boost at the beginning that helped me get past my initial lack of confidence.

People naturally feel wary about OpenAI and its ownership by Microsoft. Microsoft is a huge megacorporation, one that has been known to engage in aggressive anti-competitive anti-open-source behavior in the past. So a lot of people see Copilot as a ploy, a disguised attempt to trample the rights of the open-source community,
using the community's work while contributing little in return, ultimately all the value to their own private ecosystems.

I understand the concern, but I don't really see it. One, because Copilot on its own isn't enough to eg make a fully proprietary version of, say, GCC for evil proprietary goals. Yes, Copilot sometimes leaks open-source code in very specific occasions, but you can't just prompt Copilot with "make a proprietary clone of \<some OSS project> without attribution" and get working software.

And two, OpenAI has created something amazing! For all the bugs and all the concerns I've list and those I haven't, to me they almost all come second to the sheer marvel I feel when I start writing code and Copilot offers me exactly what I was about to write. The machine is *reading my mind*.

Yes, the Microsoft executives and the market forces driving them all have selfish goals, but I think it's not too much to assume that the basic motivation behind the entire project is "create something awesome", and we shouldn't just assume that the only possible reason they could have done it is to sabotage the OSS ecosystem.

So I don't think Microsoft did something ethically wrong. But was it strategically wrong?

### Consequences for Microsoft

Because even if Microsoft did nothing wrong, they sure
did something impopular. A lot of open-source developers were _absolutely
furious_ at them, swore to never use microsoft products again, shouted that the
company was going back to its Embrace-Extend-Extinguish roots, etc.

The controversy got so bad that the small company I was in at the time decided
to delay moving their open-source code to Github, because they were worried of
giving Microsoft control over their code given the recent scandal. (No, I don't
think it was a very rational worry, but it was more about the negative
associations than cost-benefit analysis.)

This feels like something Microsoft should care about. They've spent a lot to shed their old reputation as the enemy of open-source. They've started financing OSS projects, releasing more and more of their code as open-source (which isn't necessarily altruistic so much as [comoditizing your complement](https://gwern.net/complement)); they bought Github and kept it free for open-source projects.

Their message has clearly been "don't worry, open-source developers, you're safe with us", with the hopes that developers would stay in the Microsoft ecosystem and keep using Github, DotNet, Azure, and other products, and then convince their managers of buying the enterprise versions of these products.

It's hard to put a dollar amount on this commitment to open-source, but it feels non-zero. At the very least, it's larger than the open-source budget of early-2000s Microsoft. Good PR with the open source community clearly has monetary value for Microsoft.

So a few months ago, I've started to wonder: was the revenue generated from Copilot really worth throwing away that goodwill they took years to accumulate?

The thing is, I don't even think Microsoft execs considered the question. I suspect throwing that goodwill away was an accident, not a deliberate calculation on their part. I think they didn't anticipate the amount of backlash Copilot would generate, and, once presented with the fait accompli, decided to double down.

And you might argue that if they *had* done the cost-benefit analysis, their decision would have been the same. Backlash might not hurt sales that much, and now they have the most powerful code completion model on the market, something you *can* put a dollar amount on. Maybe the benefits of being the only company selling a working code assistant outweighs the costs of bad PR.

Except... they might not be the only company on that market for long.

## AI is going open source

To anyone paying attention to the field of AI, the pace of progress of
open-source development for the past year has been absolutely amazing. It seems
that every week there's a new open-source model, a new framework, or a new
optimization to run or fine-tune existing models with fewer resources.

OpenAI announced their Dall-E 2 model, the one that kicked off the current AI
craze, in April 2022. The first open-source alternative, Stable Diffusion, came
out in August 2022. That's _four months_ later.

Other people have noticed the same trend. If you want an article gushing about
open-source AI, I strongly recommend
[We have no moat, and never does OpenAI](https://www.semianalysis.com/p/google-we-have-no-moat-and-neither),
a leaked document on the subject from a Google insider.

The document is extremely enthusiastic about open-source AI developments:

> I’m talking, of course, about open source. Plainly put, they are lapping us.
> Things we consider “major open problems” are solved and in people’s hands
> today. Just to name a few:
>
> - LLMs on a Phone: People are running foundation models on a Pixel 6 at 5
  > tokens / sec.
> - Scalable Personal AI: You can finetune a personalized AI on your laptop in
  > an evening.
> - Responsible Release: This one isn’t “solved” so much as “obviated”. There
  > are entire websites full of art models with no restrictions whatsoever, and
  > text is not far behind.
> - Multimodality: The current multimodal ScienceQA SOTA was trained in an hour.

The memo is great and I recommend reading all of it (at least the part that
isn't behind a paywall), but I'm not sure I share the author's
enthusiasm/fatalism about open-source models.

As I said, it feels like a new open-source LLM (Large Language Model) is coming
out every week, each one boasting of its huge parameter count and its training
corpus of billions of tokens. Last year, we had two big open-source models:

- **OPT**, a 75B open-source model released in May 2022 by Facebook AI.
- **BLOOM**, a 176B open-source model released in July 2022 by model aggregator
  HuggingFace, in tandem with French research agencies CNRS and GENCI.
  - Hugging Face later released **BLOOMZ** in November 2022, a version fine-tuned
    for following instructions.

Then, over the last four months, we've had:

- **Pythia**, a 12B open-source model released on February 13 2023 by Eleuther AI,
  focused on helping interpretability research.
- **LLaMA**, a 65B model released on February 24 2023 by Facebook AI, to academia at
  first, and quickly leaked to the public.
- **FLAN UL2**, a 20B model released on March 3 2023 by Google Brain, based on the
  "T5 with mixture-of-experts" architecture.
- **Stanford Alpaca**, a version of LLaMA fine-tuned for instruction-following,
  released on March 13 2023.
- **Cerebras-GPT**, a 13B open-source model released on March 28 2023 by
  AI-specialized cloud computing company Cerebras.
- **Vicuna**, another version of LLaMA fine-tuned for conversation, using
  user-shared conversation logs from ChatGPT, released on March 30 2023.
- **Koala**, a LLaMA version fine-tuned for conversation, released on April 4 2023 by Berkeley researchers.
- **Replit-Code**, a 3B open-source model trained from scratch on code completions, released on April 26 2023 by SAAS company Replit.
- **StarCoder**, a 15B open-source model trained from scratch on code completions, released on May 4 2023 by HuggingFace.
- **MPT**, a 7B open-source model released on May 5 2023 by AI cloud orchestrator
  MosaicML.

(All the sizes are given in numbers of parameters; "a 42B model" means a model
with 42 billion weights in its neural network. Most of these models were also
released with smaller versions which I'm not listing.)

That's a lot. Each of the models I've listed is available online in one form or another; most are open-source and some even allow commercial use.

So there's nothing stopping your from loading one of these models on your
computer or, more likely, a cloud VM, and start experimenting with
them, to get results as good as you'd get from GPT-3 or ChatGPT.

Or, well, that's what's advertised.

## Benchmarking code completion models

Companies releasing large language models love to claim results "comparable to ChatGPT" or "better than ChatGPT", sometimes even with benchmarks, but I've seen very few experience reports from people actually using these models, what little I've seen make me skeptical (eg the [ChatGPT vs Open Source on harder tasks](https://github.com/microsoft/guidance/blob/8677f3aa269e05ecbb942585560a44db51d507ca/notebooks/chatgpt_vs_open_source_on_harder_tasks.ipynb) post).

So, while I was originally going to post the above rant and leave it here, at some point I had to ask myself "Wait, are open-source alternatives really a threat to Copilot's business model?". Microsoft won't have trouble selling subscriptions if all the open-source alternatives are crap.

Well, a week later (I put waaaayyy too much time into this), I've tried every code assistant service and large language model I could get my hands on.

To compare these models, I had them all write a relatively simple program from
[round 1 of the Master Dev Code 2023 hackaton](https://www.isograd-testingservices.com/FR/solutions-challenges-de-code?cts_id=99&reg_typ_id=2&que_str_id=&cli_id=45alrk6jpdnaguf3oa3gto2875),
with the instructions translated to english to avoid penalizing english-only
models:

```js
// We have a list of recipes with the given format:
// - Line 0: integer N which is the number of recipes in list
// - Line 1-N: a recipe, with three ingredients separated with spaces
//
// Given the recipes as input, write the number of unique ingredients on standard output
//
// EXAMPLE INPUT:
// 3
// tomato mozzarella ham
// tomato pineapple ham
// cream pepper mushroom
// EXPECTED OUTPUT:
// 7

const input = `
3
tomato mozzarella ham
tomato pineapple ham
cream pepper mushroom
`.trim().split("\n");

// <autocomplete starts here>
```

(To be clear, the text they were asked to complete never had the string `<autocomplete starts here>`; in some cases I did add a "complete this code" prompt at the begginning for chatgpt-style models.)

### Copilot

Using the
[Copilot extension for VsCode](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
(which Github lets me use for free, because they apparently consider me to be an
open-source maintainer) I get this:

```js
// <autocomplete starts here>
const recipes = input.slice(1).map(line => line.split(' '));
const ingredients = recipes.reduce((acc, recipe) => {
    recipe.forEach(ingredient => {
        if (!acc.includes(ingredient)) {
            acc.push(ingredient);
        }
    });
    return acc;
}, []);

console.log(ingredients.length);

// rewrite with Set
const ingredients2 = recipes.reduce((acc, recipe) => {
    recipe.forEach(ingredient => acc.add(ingredient));
    return acc;
}
, new Set());

console.log(ingredients2.size);

// rewrite with Set and flat
const ingredients3 = [...new Set(recipes.flat())];
console.log(ingredients3.length);
```

(The "rewrite with" comments are from me; although interestingly, Copilot was
able to guess "with Set" when I typed the first comment.)

Overall, this is a solid performance. Copilot gives me code that performs the task I asked, and I was easily able to get small adjustments from it.

### Codeium

Next, I pasted the code in the [Codeium playground](https://codeium.com/playground):

```js
// <autocomplete starts here>
const N = +input[0];
const recipes = input.slice(1);

console.log(recipes.length);

const ingredients = recipes.map(recipe => recipe.split(' '));
const uniqueIngredients = new Set(ingredients.flat());
console.log(uniqueIngredients.size);
```

The first console.log is useless, but when prompted to continue, Codeium
does end up giving the right answer.

Also, Codeium was a bit shy about providing suggestions when given empty lines.
Just typing "const" did the trick. I expect this wouldn't be a problem in
practice.

### MutableAI

Next I installed the [MutableAI VsCode extension](https://marketplace.visualstudio.com/items?itemName=mutable-ai.mutable-ai).

```js
// <autocomplete starts here>
const recipes = input.slice(1).map(line => line.split(' '));

const ingredients = recipes.reduce((acc, recipe) => {
    return acc.concat(recipe);
    }
, []);

const uniqueIngredients = ingredients.reduce((acc, ingredient) => {
    if (!acc.includes(ingredient)) {
        acc.push(ingredient);
    }
    return acc;
}
, []);

console.log(uniqueIngredients.length);

// Rewrite the code above using a Set
const ingredientsSet = new Set(ingredients);

// Rewrite the code above using a Set and flat
const ingredientsSet2 = new Set(recipes.flat());
```

I got similar results as Copilot; getting MutableAI to implement small changes felt subjectively a bit harder, though it's hard to determine.

Also, when restarting VsCode I got a popup warning me that MutableAI was switching from the Codex API to the ChatGPT API. This means MutableAI acts as a distributor for OpenAI's product; this is a perfectly valid business model, but I do wish they were more upfront about this connection; it's not mentioned anywhere on their website or on the extension's README.

### FauxPilot

After a *lot* of hiccups, I managed to install and self-host [FauxPilot](https://github.com/fauxpilot/fauxpilot/), and get the Github Copilot extension to connect to my Fauxpilot server.

```js
// <autocomplete starts here>
    const recipe = input.slice(0, 1);
    const ingredients = recipe.slice(1, 3);

    console.log(`${recipe}`);
    console.log(`${ingredients}`);

    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(omic.format(input));
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
    console.log(`${input.slice(1, 3)}`);
```

(Getting that completion took one minute during which I couldn't type anything or switch windows, on pain of losing the completion. This is going to be a bit of a pattern: self-hosting any LLM is almost always a pain, and never competitive with using an online API.)

... Yeah, let's move on.

### CodeGenX

Getting CodeGenX apparently requires that you go to [their website](https://www.deepgenx.com/), give them your email adress, and get an API key. I gave them my email, and didn't get anything.

Since their Github repository hasn't had a commit for almost a year, I'm just going to assume the project is dead.

### CodeGeeX

I tried [the CodeGeeX playground](https://codegeex.cn/playground) and got this result:

```
// <autocomplete starts here>
... [ [ syntax syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax.
```

... Yeah, this is weird. I suspect this is a front-end bug or some sort of enconding problem rather than a LLM problem.

Anyway, I tried again a few days later, and got this:

```js
// <autocomplete starts here>
const unique = (arr) => arr.length === arr.filter((e,i) => arr.indexOf(e) === i).length;

console.log(unique(input));
```

This is pretty much useless. Nothing else to say.

### TabNine

One of the oldest Copilot alternatives AFAIK. I went and installed their [VsCode extension](https://www.tabnine.com/install/vscode).

```js
// <autocomplete starts here>
const recipes = input.map(line => line.split(' '));
console.log(recipes.length);

const uniqueIngredients = recipes.reduce((acc, recipe) => {
    const ingredients = recipe[1].split(' ');
    ingredients.forEach(ingredient => {
        if (!acc.includes(ingredient)) {
            acc.push(ingredient);
        }
    });
    return acc;

}, []);

console.log(uniqueIngredients.length);

// Rewrite this solution to use Set

const uniqueIngredients2 = recipes.reduce((acc, recipe) => {
    const ingredients = recipe[1].split(' ');
    ingredients.forEach(ingredient => {
        if (!acc.has(ingredient)) {
            acc.add(ingredient);
        }
    });
    return acc;

}, new Set());
console.log(uniqueIngredients2.size);
```

This is not bad, although the code is incorrect (the `recipe[1]` part shouldn't be here; and the `if (!acc.has(ingredient))` check in the second solution is redundant).

Unfortunately, nothing about TabNine is open-source, but if you're looking for an alternative that doesn't go through Microsoft with an UX roughly as good as Copilot's, TabNine seems like a good option.

### Amazon Whisperer

Did you know Amazon had their own code generation model? I didn't!

I followed the instructions on their [Getting Started page](https://aws.amazon.com/codewhisperer/resources/#Getting_started) (unlike TabNine, they don't ask for you to take a monthly susbscription to try the product, which I definitely appreciate) and installed it through the VsCode AWS extension.

```js
// <autocomplete starts here>
const recipes = input.slice(1).map(line => line.split(' '));
const ingredients = new Set();
for (const recipe of recipes) {
    for (const ingredient of recipe) {
    ingredients.add(ingredient);
  }
}
console.log(ingredients.size);

// Alternative solution:
const recipes = input.slice(1).map(line => line.split(' '));
const ingredients = new Set(recipes.reduce((acc, recipe) => acc.concat(recipe), []));
console.log(ingredients.size);

// Alternative solution:
const recipes = input.slice(1).map(line => line.split(' '));
const ingredients = new Set(recipes.flat());
console.log(ingredients.size);
```

(Note that, unlike in the previous snippets, Amazon proposed the alternative solutions on its own without prompting. Odd.)

The proposed completions have bad indentation, and the extension has some UX problems where it's a bit fiddly to get the extension to propose completions (I think TabNine had the same problem), but other than that this felt like the bext Copilot alternative I'd tried so far.

The first solution is the one I'd pick if I wrote the code myself (or the third if I wanted to stay concise); the completions are proposed quickly and I got this feeling that the AI "understood" me that I didn't really get from the other models except for Copilot.

Of course, this comes with the caveat that you're trading one corporate overlord for another. But if you're already dependent on AWS, Whisperer is probably the Copilot alternative you want.

### BLOOMZ-7B

Enough with the pre-packaged VsCode extensions using a proprietary API. What about open-source models?

Well, here I have to warn you: I'm extremely new at machine learning, and most of the models I found I failed to get running on my machine.

Fortunately, the GGML ecosystem and the Rust ecosystem both came to my rescue, in the form of [the llm crate](https://github.com/rustformers/llm), a tool helping you run language models using [the GGML format](https://github.com/ggerganov/ggml) in the CPU. (Basically the Rust equivalent of [llama.cpp](https://github.com/ggerganov/llama.cpp))

Even that was no guarantee, and there were quite a few models that llm couldn't load, but the llm repo came with [a list of models known to work](https://github.com/rustformers/llm/blob/main/doc/known-good-models.md), which helped me get started.

Anyway, the first serious model I tried was [BLOOMZ](https://huggingface.co/bigscience/bloomz), a fine-tune of HuggingFace's own BLOOM model. Specifically, I tried [a 4bit-quantized version of the 7B model](https://huggingface.co/Wauplin/bloomz-7b1.cpp).

```js
// <autocomplete starts here>
const  uniqueIngredientsCountedByLineNumberOfTheListing = [];
for (let line of Array.from(input)) {
  let ingredientString = ` ${line}`
     .replace(/\n/g, '')
     .replaceEach((s, (_, a) => s === ' '? 'a': 'b'))
      ;

  const ingredientsSplitUpIntoGroupsByNumberOfSpacesInOneLine
      : [Array<string>][] | null = uniqueIngredientsCountedByLineOfListing.filter(
          u => parseInt(u[2] - (a + 1)) > 0
      ).map((e, i) => e[i] || [])
     .sort();
```

... Yeah, this is basically worthless. There's a reason BLOOM has basically languished in obscurity since it's come out. The model is desperately trying to pretend it knows what's going on, and failing.

Maybe the 176B version would fare better, but (1) I couldn't find one with the GGML format (2) I don't have 176 GB of RAM.

### MPT-7B

It's okay, though, I didn't have much hope for BLOOM. Maybe [MPT](https://www.mosaicml.com/blog/mpt-7b) will do better?

I downloaded a few versions of [MPT-7B in GGML format](https://huggingface.co/LLukas22/mpt-7b-ggml).

First the 4bit-quantized version.

```js
// <autocomplete starts here>
let result;
 	for (var i=0, recipes=[];i<input.length-1 ;++recipes){
	    // skip first recipe element which is the number of ingredients in list 
	   if (!(result = input[2].indexOf(' ')) < 0) return console.log(Math  
            ./ log 2 ) // using Math library because we need an integer result and there are no math methods for indexing String object
	    recipes.push (input [i]) 
	} 
		
	   return recipes;
```

Yeesh. Maybe the quantization hurts it? Here's the f16 version:

```js
// <autocomplete starts here>
console.log("input",  `${Array}.join(' ')}`) // Array [ 3, "tomatoe ", muzarella, and,"ham" ] 0:3 1:"mozzeralla 2:",,"tomato pineapple ham"]
```

Nope, it's even worse.

Before I ran these experiments, I had a suspicion that scale mattered more than anything else for these generations model, and anything under 10B would be useless for code generation. For comparison, GPT-3 has 175 billion parameters, and LLaMA has up to 65 billions.

My results with BLOOMZ-7B and MPT-7B confirmed this impression.

### Cerebras-13B

I wanted to try [Cerebras-13B](https://huggingface.co/cerebras/Cerebras-GPT-13B), the third open-source language model on the "very big" scale, but I coudln't find a GGML-compatible version.

Also, the model is way too heavy for me to run it locally. It uses f32 parameters, meaning you need (13 x 4) 52GB of memory to run it. That's ridiculous! Virtually everybody in ML uses some form of 16-bits float these days, at least once they're done training. You can even find discussions where [people argue](https://news.ycombinator.com/item?id=36041022) that storing 32-bits weights is obsolete, and training should use exclusively 16-bits floats.

That's a shame; Cerebras went through all the trouble of training and open-sourcing a very large, presumably very expensive language model, to serve as a showcase for their datacenter architecture; and then they didn't bother to do the basic post-processing that would have made people more likely to actually use the model.

I just hope someone comes along and makes a GGML version of the model with f16 weights or even quantization.

### LLaMA 33B

Finally, I decided to go big and try the LLaMA model.

After a bit of asking around on the HuggingFace discord, I was able to find some GGML-compatible models. However, even running on CPU, most of the models were way too big for my 32GB RAM. I'm starting to seriously consider upgrading my rig, which is something I thought I wouldn't be saying for *years*.

Anyway, I settled for a q5 quantization for the 33B model.

```js
// <autocomplete starts here>
input.splice(0,1); // remove the line with N from start of array;
let ingredientsArr: any[] = [];
for (const [i] of input) {
    const arr = i.replace(/ /g,'').toLowerCase() .split(' ').map((item)=> item !== '' ? 1 :0); // make sure to convert the array back into number for later count!
    ingredientsArr +=arr;
}
let uniqueIngredeints: any = Array.from({length:ingredientArr.reduce(function (a,b){return a+b})});
uniqueIngredeints = [];  // this is to reset the array for new calculation!
const countUniqueIngredients = () => {
    const i:any[]= [] ; let counter : any;
        while (!(counter === uniqueIngredeints.reduce((a,b) => a+b))){   // this is to make sure there are no duplicate ingreedient in the array!
            for (let j = 0;j <ingredientsArr.length ; ++j ) {    // iterate all the elements of input and find how many times each element appears!
                if(uniqueIngredeints[i].includes(counter)){     counter +=1}      else{   i.push (counter) }  uniqueIngredeints.push([...ingredientsArr]);        countUniqueIngredients();         }       return i;    }} ;
console .log(countUniqeIngredient);
```

It tries so hard 😕

This is the only example I've had so far where it felt quantization might be seriously hurting the model.

Like, other failing models felt like a non-technical person trying to pretend to code after seeing a few program. This one feels like a beginner trying to program while recovering from severe alcohol poisonning. It feels like the general understanding is *there*, it understands that it needs unique ingredients, that it needs to add them in a loop except if they're already there, etc. But the understanding doesn't quite converge to an actual valid program.

(I haven't tried Alpace or Vicuna, but I assume results would be the same. Their fine-tuning is very light, so I assume it would impact the model's surface behavior, not its deep understanding of concepts.)

---

### StarCoder

[StarCoder](https://huggingface.co/bigcode/starcoder) is an open-source model I wasn't aware of at all before I started this article. In fact, I almost published the article without including it, until I read the post announcing it on HuggingFace's blog.

(I typed the code directly into the inference API widget in the model's main page; StarCoder also has [a code playground](https://huggingface.co/spaces/bigcode/bigcode-playground) which didn't work for me.)

It's 13B model trained specifically on code. Does that training make up for the model's comparatively small size?

```js
// <autocomplete starts here>
// Getting list
of recipes
const recipes = input.slice(1);

// Getting number of recipes
const numberOfRecipes = parseInt(input[0]);

// Getting unique ingredients
const uniqueIngredients = new Set();

// Looping through recipes
for (let i = 0; i < numberOfRecipes; i++) {
  // Getting ingredients
  const ingredients = recipes[i].split(' ');

  // Adding ingredients to unique ingredients
  for (let j = 0; j < ingredients.length; j++) {
    uniqueIngredients.add(ingredients[j]);
  }
}

// Printing number of unique ingredients
console.log(uniqueIngredients.size);
```

Oh my. Yes, yes it does.

StarCoder is a pure text-completion model, not an instruction-following model, so it takes a bit of effort to get it to write code. By default, if you just write instructions, it'll just complete your prompt with more instructions in the same style. Once it gets started, though, it works like a charm.

With a bit of nudging I got this result:

```js
// <autocomplete starts here>
// We can use a set to store the unique ingredients
const uniqueIngredients = new Set();

// We can use a for-of loop to iterate over the recipes
for (const recipe of input.slice(1)) {
  // We can use a for-of loop to iterate over the ingredients in the recipe
  for (const ingredient of recipe.split(' ')) {
    // We can use the set's add method to add the ingredient to the set
    uniqueIngredients.add(ingredient);
  }
}

// We can use the set's size property to get the number of unique ingredients
console.log(uniqueIngredients.size);
```

(My only interventions here were the two slashes of the first comment, and splicing in "for-of loop" in the second comment.)

It's interesting to see that once it starts commenting its code, every important line must be commented. I suspect this makes the model more powerful, in a manner similar to chain-of-thought prompting.

Anyway, this is good stuff. Remove the comments and it's basically the code I'd write.

### Replit-Code

This is another suprisingly small model I wasn't aware of.

Replit is a small SAAS company providing a CodeMirror-based online IDE and associated software. They recently released [replit-code-v1-3b](https://huggingface.co/replit/replit-code-v1-3b), a 2.7B (!!!) parameters model focused on code completions, suggestions, answering questions about code, etc.

This is a *shockingly small* model by today's standards. Can [the demo](https://huggingface.co/spaces/replit/replit-code-v1-3b-demo) still complete my code?

```js
// <autocomplete starts here>
const recipes = input.map((r) => r.split(' '));

const ingredients = recipes.map((r) => r.slice(0, 3));

const uniqueIngredients = ingredients.reduce((acc, r) => {
  if (!acc.includes(r)) {
    acc.push(r);
  }
  return acc;
}, []);

console.log(uniqueIngredients.length);
```

It can!

This is not the way I would write that code, though. The first lines are a bit odd, and I hate the `reduce` method. Can we get it to rewrite it?

```js
const recipes = input.map((r) => r.split(' '));

const ingredients = recipes.map((r) => r.slice(0, 3));

// get unique ingredients with Set and for-of loops
// <autocomplete starts here>
const uniqueIngredients = new Set();
for (const ingredient of ingredients) {
  uniqueIngredients.add(ingredient.join(' '));
}

console.log(uniqueIngredients.size);
```

(Comment is mine, the rest is Replit completions.)

Oops. The algorithm is no longer correct. What if I ask for nested for-loops instead?

```js
const recipes = input.map((r) => r.split(' '));

const ingredients = recipes.map((r) => r.slice(0, 3));

// get unique ingredients with Set and nested for-of loops
// <autocomplete starts here>
const uniqueIngredients = new Set();
for (const ingredient of ingredients) {
  for (const ing of ingredient) {
    uniqueIngredients.add(ing);
  }
}
console.log(uniqueIngredients.size);
```

Now we're getting somewhere, but the variable names and the general code structure are a bit *off*. What if I put the comment at the top of the completion instead?

```js
// <autocomplete starts here>
// get unique ingredients with Set and nested for-of loops
const ingredients = new Set();
for (const recipe of input) {
  const ingredientsList = recipe.split(' ');
  for (const ingredient of ingredientsList) {
    ingredients.add(ingredient);
  }
}

console.log(ingredients.size);
```

That's much better!

I'm very curious about Replit now. The small size seems like it would make it very convenient for local execution.

### CodeGen

After that I tried to run Salesforce's Codegen, which is where things got complicated. The models I was interested in didn't have a GGML version and the online playgrounds didn't work, which means I had to finally learn how to load a model in Python.

Fortunately, HuggingFace provides a `transformers` library which does most of the work for you, and each model page has a "Use in Transformers" button which in theory gives you the code you need to import the model from your Python code. In practice it's pretty unreliable, but it gave me a starting point, and with some help from ChatGPT I was able to get something working.

Unfortunately, most of the Salesforce models are too big to fit on my GPU, and even those that should have fit in CPU mode gave some error. Eventually I decide I'd spent enough time on this already and I gave up.

Still, here's what Codegen-350M-mono gives me (given a python version of the prompt)

```py
# <autocomplete starts here>
ingredients = []

for i in input:
    ingredients.append(i.split(' '))

ingredients.sort()

print(len(ingredients))

for i in ingredients:
    print(i[0], i[1])
```

It doesn't solve the task, but it gets somewhere close. Overall pretty good for a 350M model!

Here's what Codegen2-1B gave me:

```js
const recipes = input.map((line) => {
  const [ingredients,...rest] = line.split(' ');
  return { ingredients, rest };
});

const recipesWithUniqueIngredients = recipes.filter((recipe) => {
  const uniqueIngredients = new Set(recipe.ingredients);
  return uniqueIngredients.size === 1;
});

console.log(recipesWithUniqueIngredients.length);
```

Not great. It's even less close to solving the task than the previous example.

It's surprising that a successor model with more parameters gives worse results here. I suspect that having to only understand one single language gives Codegen-350M-mono an edge compared the more generalist model.

### Others

There's a bunch of models I haven't tried:

- The original Codex
- CodeClippy
- CodeAlpaca
- Wizard-Vicuna
- Probably a bunch of others

Each of these models either wasn't available, was too hard to install, didn't work once installed, or had some other problem I didn't want to sink more time into.

## Conclusion

I learned a lot while writing this article.

I previously leaned towards the "scale is all you need" crowd, and I thought scale was absolutely necessary to get actual value out of a language parameter. The results I got from StarCoder and ReplitCode are making me question that assumption.

Small scale models did prove unable to provide usable completions when they weren't trained specifically on code completions. So I suppose the lesson is that scale does help a lot, but lack of scale can be compensated with domain-specific training.

Out of all the models tools I've tested, Copilot is still the one I'll keep using, because it works out of the box and has a very convenient UX (though Amazon Whisperer comes close). But a year or two from now, when an equivalent tool is developed around a model like StarCoder? I could see myself using it.

### Does Microsoft still have a moat?

Nope, they're screwed.

Or, well, "screwed" is a strong word. They'll still make billions in revenue no matter what the open-source community does.

But I'm starting to think the key points from [We have no moat, and never does OpenAI](https://www.semianalysis.com/p/google-we-have-no-moat-and-neither) are correct.

OpenAI does have a head start compared to the rest of the field, and maybe they'll still have a head start ten years from now. Maybe GPT-4 will stay ahead of the curve for a long time, because no-one else will have the budget, the talent and the motivation to build a one-trillion-parameters model. Or maybe by the time someone does, they'll have released GPT-5 and left the competition in the dust again.

It doesn't matter, though, because for code completion you don't *need* a bleeding-edge model. You need a tool that's fast, adaptable, with good UI and good-enough results.

This is a playground where open source tends to dominate. And for fast experimentation, the small size of open-source models is actually a benefit, not a handicap. So I wouldn't be surprised if two or three years from now a new code assistant had made Copilot obsolete, the same way Stable Diffusion made everyone forget about Dall-E.

In the meantime, if any Microsoft executives happen to be reading this, here are my recommendations:

- Acknowledge the distrust your actions have created. An apology is probably too much to ask, especially given the current court cases surrounding AI, but at least admit you could have handled things in a more cooperative way.
- Announce a new open-source code model trained on a corpus of projects on an opt-in basis. No, MIT licence isn't enough.
- Give project owners incentives to opt-in. (Doesn't need to be financial incentives. You don't need everybody, just a lot of projects.)
- As a sign that you're confident the model isn't going to regurgitate existing code, train the model on your own code as well. It's kind of hard to argue that AI training doesn't hurt hurt intellectual property if you're not willing to put your own IP on the line.
- Build B2B offerings for companies who want to sell their Copilot alternative on the cloud. If a company like Replit already has a trained model, you might as well get them to run it on Azure. Provide a vertically integrated API so that they only have to give you the model weights, and they get access to an API endpoint that the VsCode Copilot extension can connect to.

Oh, and if you happen to be looking for a new CEO, feel free to send me an email.

To anyone else: it's a pretty exciting time for sure, and a pretty scary one as well. A lot of the developments I've mentioned happened in the last two months. I have *no idea* where the ecosystem will be in two years.

For now, I guess we should just stay informed, and use the shiny new tools as they come out.

[Discussion on Hacker News.](https://news.ycombinator.com/item?id=36184860)
