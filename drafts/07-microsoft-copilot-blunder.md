---
layout: post
title: How good are alternatives to Github Copilot?
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
