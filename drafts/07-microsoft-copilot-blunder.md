---
layout: post
title: Rambling about Microsoft, and testing alternatives to Github Copilot.
---

I've recently been in a discussion where I argued that Microsoft had made a
strategic blunder by releasing Copilot the way they did, releasing a product
built from a large database of open-source project which itself was very much
proprietary.

To be clear, I wasn't saying that Microsoft had done something unforgiveably
immoral. I was guessing that, in te long term, that decision would not be in
Microsoft's best interests, and would in fact cost the more money than a more
open release would have, by pushing developers to use open alternatives instead.

This led me to wonder: how viable are the alternatives that _actually_ exist?
I'd kind of taken for granted that they were competently made or vaguely
comparable to the state of the art, which is a very dangerous assumption in
computer science, and especially in AI.

Before I give you what I found (spoiler: they're not great), let me give you a
quick recap of the debates around Copilot, and my hot take on them.

## Microsoft Code Simulator 2021

According to Wikipedia, Github Copilot's first alpha version came out in June
2021 (holy crap, it's been two years already?).

Copilot is a service built upon OpenAI's _Codex_ model; Codex itself is an
offshoot of GPT-3, OpenAI's groundbreaking text-generating AI. (Also, if you've
never heard of GPT, congratulation for succesfully waking up from your cryogenic
sleep!).

Copilot and Codex immediately stirred up controversy because of how they were
trained: they were given next-token prediction tasks, similar to GPT, with a
corpus consisting mostly of Github Python projects, according to the OpenAI
Codex paper:

> Our training dataset was collected in May 2020 from 54 million public software
> repositories hosted on GitHub, containing 179 GB of unique Python files under
> 1 MB. We filtered out files which were likely auto-generated, had average line
> length greater than 100, had maximum line length greater than 1000, or
> contained a small percentage of alphanumeric characters. After filtering, our
> final dataset totaled 159 GB

(OpenAI later trained successor models with code in other languages and data
from other websites)

Now, OpenAI did some filtering to ensure these projects were open-source, and
it's possible a great many of their authors would have consented to their
inclusion had they been asked. But instead what OpenAI did was adopt an attitude
of "better ask for forgiveness than permission", only without the "asking for
forgiveness" part.

This attitude is by no means unique to them, and is a result of the general
incentives of machine learning: data is often easily accessible, as long as you
don't need informed consent for it. Some of the data may be copyrighted, some of
the data may be open-source-with-attribution where the nature of your work means
attribution will be impossible to give reliably, some may be
open-source-only-for-open-source projects, and some data may be in the public
domain on a website that forbid scraping in its terms of service.

Even when data is in the public domain and accessible, its author may have
released it before generative AI became available, and may not be fine with it
being used to build a tool to automate their job away; so maybe it would be more
ethical (but legally unecessary) to ask for author permission before using even
public domain data.

The point is, if you want to get consent, you need to ask _a lot_ of people for
their data. Sometimes this is work these people forgot they did linked to an
email adress they no longer use; sometimes this is data that a lot of people
contributed to, and it's unclear who actually owns it. It's much easier to just
scrap the data and not worry about any of that.

And it's not clear this is even morally wrong? Copying isn't theft. Intellectual
property isn't a natural law, it's a system we put in place to incentivize
creating stuff, and arguably a pretty broken system. If I'm writing Spiderman
fanfiction, who am I to judge machine learning researchers? Maybe the difference
is that they're trying to make money, but I would be pretty happy selling my
Spiderman-Harry Potter crossovers for money, and screw Marvel and JK Rowling if
they want a cut.

Maybe the moral calculus changes when the perpetrator is a huge megacorporation
like Microsoft? People see moves like this and worry that, if we let them,
Microsoft & co will slowly trample the rights of the open-source community,
using the product of their work will contributing less and less to it while they
move all the value to their own private ecosystems. I don't really see it, but I
understand the concern.

Anyway, while I'm not sure Microsoft did something ethically wrong, they sure
did something impopular. A lot of open-source developers are _absolutely
furious_ at them, swore to never use microsoft product again, shouted that the
company was going back to its Embrace-Extend-Extinguish roots, etc.

The controversy got so bad that the small company I was in at the time decided
to delay moving their open-source code to Github, because they were worried of
giving Microsoft control over their code given the recent scandal. (No, I don't
think it was a very rational worry, but it was more about the negative
associations than cost-benefit analysis.)

Hence why I started to wonder: was it really worth it for them?

## The liberation of AI

To anyone paying attention to the field of AI, the pace of progress of
open-source development for the past year has been absolutely amazing. It seems
that every week there's a new open-source model, a new framework, or a new
optimization to run or fine-tune existing models with fewer resources.

OpenAI announced their Dall-E 2 model, the one that kicked off the current AI
craze, in April 2022. The first open-source alternative, Stable Diffusion, came
out in August 2022. That's _four months_ later.

I'm not going to enumerate all the other open-source achievements in machine
learning that have come out recently. If you want an article gushing about
open-source AI, I strongly recommend
[We have no moat, and never does OpenAI](https://www.semianalysis.com/p/google-we-have-no-moat-and-neither),
a leaked document from a Google insider on the subject.

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

- OPT, a 75B open-source model released in May 2022 by Facebook AI.
- BLOOM, a 176B open-source model released in July 2022 by model aggregator
  HuggingFace, in tandem with French research agencies CNRS and GENCI.
  - Hugging Face later released BLOOMZ in November 2022, a version fine-tuned
    for following instructions. Then, over the last four months, we've had:

- Pythia, a 12B open-source model released on February 13 2023 by Eleuther AI,
  focused on helping interpretability research.
- LLaMA, a 65B model released on February 24 2023 by Facebook AI, to academia at
  first, and quickly leaked to the public.
- FLAN UL2, a 20B model released on March 3 2023 by Google Brain, based on the
  "T5 with mixture-of-experts" architecture.
- Stanford Alpaca, a version of LLaMA fine-tuned for instruction-following,
  released on March 13 2023.
- Cerebras-GPT, a 13B open-source model released on March 28 2023 by
  AI-specialized cloud computing company Cerebras.
- Vicuna, another version of LLaMA fine-tuned for conversation, using
  user-shared conversations logs from ChatGPT, released on March 30 2023.
- Koala, a LLaMA fine-tuned for conversation, release by Berkeley researchers.
- MPT, a 7B open-source model released on May 5 2023 by AI cloud orchestrator
  MosaicML.

- March 28 Cerebras

(All the sizes are given in numbers of parameters; "a 42B model" means a model
with 42 billion weights in its neural network. Most of these models were also
released with smaller versions which I'm not listing.)

That's a lot. Each of the models I've listed is either open-source or "not
technically open but the weights can be found and nobody is going to put you in
jail if you use them" (at least for personal use; commercial use might be more
complicated).

So there's nothing stopping your from loading one of these models on your
computer or, more likely, a Google Colab notebook, and start experimenting with
them, to get results as good as you'd get from GPT-3 or ChatGPT.

Or, well, that's what's advertised.

## How good are these models?

each one boasting large parameter counts and performance "comparable to
ChatGPT".

GPT-3 is 175B parameters

Test

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

Copilot auto-completes with this:

```js
// <autocomplete starts here>
const recipes = input.slice(1).map((line) => line.split(" "));
const ingredients = recipes.reduce((acc, recipe) => {
  recipe.forEach((ingredient) => {
    if (!acc.includes(ingredient)) {
      acc.push(ingredient);
    }
  });
  return acc;
}, []);

console.log(ingredients.length);

// rewrite with Set
const ingredients2 = recipes.reduce((acc, recipe) => {
  recipe.forEach((ingredient) => acc.add(ingredient));
  return acc;
}, new Set());

console.log(ingredients2.size);

// rewrite with Set and flat
const ingredients3 = [...new Set(recipes.flat())];
console.log(ingredients3.length);
```

(The "rewrite with" comments are from me; although interestingly, Copilot was
able to guess "with Set" when I typed the first comment.)

---

Codeium

```
const N = +input[0];
const recipes = input.slice(1);

console.log(recipes.length);

const ingredients = recipes.map(recipe => recipe.split(' '));
const uniqueIngredients = new Set(ingredients.flat());
console.log(uniqueIngredients.size);
```

(online version)

Note that the first lines are garbage, but when prompted to continue, Codeium
does end up giving the right answer.

Also, Codeium was a bit shy about giving suggestions when given empty lines.
Just typing "const" did the trick. I expect this wouldn't be a problem in
practice.

---

MutableAI

```
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

Similar results as Copilot, subjectively a bit harder to get changes

(Also, when restarting VsCode I get a popup warning me that they're switching to
ChatGPT)

---

FauxPilot - self-hosted

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

(TODO) About 1 min per completion

---

CodeGenX

Didn't get API email, too bad

---

CodeGeeX

```
... [ [ syntax syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax. syntax.
```

---

CodeClippy BLOOM BLOOMZ CodeAlpaca Salesforce Codegen

Couldn't get any of the online versions to load

---

TabNine

```
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

Notes: not bad, but code is incorrect

---

Amazon Whisperer

```
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

(comments not from me) Light UX problems

---

BLOOMZ - 7B https://huggingface.co/Wauplin/bloomz-7b1.cpp

const uniqueIngredientsCountedByLineNumberOfTheListing = []; for (let line of
Array.from(input)) { let ingredientString = `${line}` .replace(/\n/g, '')
.replaceEach((s, (_, a) => s === ' '? 'a': 'b')) ;

const ingredientsSplitUpIntoGroupsByNumberOfSpacesInOneLine : [Array<string>][]
| null = uniqueIngredientsCountedByLineOfListing.filter( u => parseInt(u[2] -
(a + 1)) > 0 ).map((e, i) => e[i] || []) .sort();

(Basically worthless to try a smaller model; but I don't have access to the 167B
version)

---

MPT - 7B https://huggingface.co/LLukas22/mpt-7b-ggml

let result; for (var i=0, recipes=[];i<input.length-1 ;++recipes){ // skip first
recipe element which is the number of ingredients in list if (!(result =
input[2].indexOf(' ')) < 0) return console.log(Math\
./ log 2 ) // using Math library because we need an integer result and there are
no math methods for indexing String object recipes.push (input [i]) }

    return recipes;

---

Cerebras - 13B https://huggingface.co/cerebras/Cerebras-GPT-13B

Couldn't run it

---

Wizard-Vicuna - 13B

https://huggingface.co/TheBloke/wizard-vicuna-13B-GGML

✗ Failed to load model Error: 0: Could not load model 1: invalid file format
Ggjt(3)

---

LLaMA 33B

input.splice(0,1); // remove the line with N from start of array; let
ingredientsArr: any[] = []; for (const [i] of input) { const arr = i.replace(/
/g,'').toLowerCase() .split(' ').map((item)=> item !== '' ? 1 :0); // make sure
to convert the array back into number for later count! ingredientsArr +=arr; }
let uniqueIngredeints: any = Array.from({length:ingredientArr.reduce(function
(a,b){return a+b})}); uniqueIngredeints = []; // this is to reset the array for
new calculation! const countUniqueIngredients = () => { const i:any[]= [] ; let
counter : any; while (!(counter === uniqueIngredeints.reduce((a,b) => a+b))){ //
this is to make sure there are no duplicate ingreedient in the array! for (let j
= 0;j <ingredientsArr.length ; ++j ) { // iterate all the elements of input and
find how many times each element appears!
if(uniqueIngredeints[i].includes(counter)){ counter +=1} else{ i.push (counter)
} uniqueIngredeints.push([...ingredientsArr]); countUniqueIngredients(); }
return i; }} ; console .log(countUniqeIngredient);

It tries so hard =/

(I haven't tried Alpace or Vicuna, but I assume they will be the same)

---

StarCoder 13B model https://huggingface.co/spaces/bigcode/bigcode-playground

// Getting list of recipes const recipes = input.slice(1);

// Getting number of recipes const numberOfRecipes = parseInt(input[0]);

// Getting unique ingredients const uniqueIngredients = new Set();

// Looping through recipes for (let i = 0; i < numberOfRecipes; i++) { //
Getting ingredients const ingredients = recipes[i].split(' ');

// Adding ingredients to unique ingredients for (let j = 0; j <
ingredients.length; j++) { uniqueIngredients.add(ingredients[j]); } }

// Printing number of unique ingredients console.log(uniqueIngredients.size);

---

Replit https://huggingface.co/replit/replit-code-v1-3b 2.7B

const recipes = input.map((r) => r.split(' '));

const ingredients = recipes.map((r) => r.slice(0, 3));

const uniqueIngredients = ingredients.reduce((acc, r) => { if (!acc.includes(r))
{ acc.push(r); } return acc; }, []);

console.log(uniqueIngredients.length);

---

CodeGen https://huggingface.co/Salesforce/codegen-16B-mono

const recipes = ` .trim().split('\n');

const ingredients = ` .trim().split('\n');

const ingredients_count = 0;

for (int i = 0; i < recipes.length; i++) { ingredients_count++;
ingredients.splice(0, i); }

const ingredients_count_2 = 0;

for (int i = 0; i < ingredients.length; i++) { ingredients_count_2++;
ingredients.splice(i, 0); }
