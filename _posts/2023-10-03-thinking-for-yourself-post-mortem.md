---
layout: post
title: Thinking for Yourself in the Face of AI - post-mortem on my post-mortem
---

*Content note: This is mostly something I've written for myself. I still did my best to make it readable, but it might be too long, too rambly, or too focused on specific points. You've been warned.*

About a month ago, I wrote a post-mortem titled [When to not use Rust - post-mortem on my wiki converter project](/2023/09/05/convert-wiki-post-mortem/) and posted it [on r/rust](https://www.reddit.com/r/rust/comments/16apiv2/when_to_not_use_rust_postmortem_on_my_wiki/). Reactions were... critical, to say the least.

Some excerpts:

> ChatGPT used for a non-trivial project. Sigh.

> TL;DR: "I didn't want to do research about source problem and.. decided to use Rust".

> This writeup is an exhaustive laundry list of inexperienced mistakes. The most ignorant of all, though, is one of the post-mortem conclusions, since by this point the author should have learned their lesson

> Probably could’ve written this with Perl in a day

These are the most egregious bits, but something that's harder to convey with quotes is the amount of *hostility* and condescension from that thread. Stuff like all my answers being heavily downvoted even when their content boiled down to "You're right, I'm sorry, I shouldn't have done that", or people making assumptions about my skill level.

I don't want to overstate my case either: some comments were very polite and thoughtful, and even some of the more aggressive critics raised interesting points in a cordial manner. This is a surprising level of hostility *by the standards of the Rust community*, but clearly not the most incendiary behavior I've seen from an open-source community.

Still, if you're wondering why there aren't more people posting about their learning process online, this is why. It takes a certain amount of vulnerability and revealing one's mistakes to do that; that vulnerability gives random people online ammunition against you which they'll absolutely fire. 

I don't want my clinical tone to sugar-coat it: that behavior stings. It's discouraging and depressing, the kind of experience that makes you go "Well I've learned my lesson" and avoid engaging with a community from then on.

But since I'm a slow learner, I'm going to engage again. Now that I've had a few weeks to cool off, I'd like to revisit the subject with a cool head and examine the response to the post-mortem.

In particular, there are two questions I'd like to ask:

- Why did the thread attract so much hostility?
- Was it a bad idea to use ChatGPT to help code my converter project?

Let's start with the hostility.


## Why were reactions so negative?

Let's address the elephant in the room right away.

When you're an author and people tell you "Your work sucks", it's very easy to look for neat convenient explanations like "Oh, they're just biased", "They can't handle my style of writing" or "They won't accept anything that doesn't flatter their ego". It's pretty difficult for the mind to examine the answer "It's because my work sucked".

Like, literally. It's almost a physical reaction. The answer slides off your mind, and your brain tries to pick up any other train of thought. I know this from personal experience, both from myself and other people.

So let's seriously examine the hypothesis "My post was that bad".

### How bad was the post?

First, I'd say even a post being bad doesn't warrant contempt or condescension. Some people were half-politely saying things like "You had problems because you did everything wrong, do X and Y and Z instead and you won't have problems", which is constructive but grating. That's not what I'm referring to. It's the naked contempt from remarks like:

> The most ignorant of all, though, is one of the post-mortem conclusions, since by this point the author should have learned their lesson

I don't think that's an appropriate answer to any piece of writing, no matter how incompetent the author.

That being said, after mulling it over, I think the post wasn't very good as a piece of technical writing. It was *very* useful to me, as the process of writing helped me get a lot of thoughts in order; the discussion it spurred was constructive too, though I didn't like the ratio of information-to-negativity too much.

But whatever insights it had, it was bad at conveying them. I think my personal experience on the converter project wasn't that digestible. The goal wasn't compelling enough, the end result wasn't very exciting, there wasn't any major plot twist, and my retelling was too dry; I just recounted each step without really making it into a strong narrative.

This isn't a unique failure. I have the same problems with a lot of fasterthanlime's posts; I think the format of "step through this project with me" isn't that good. To some extent fasterthanlime makes it work, I didn't.

The format isn't what the audience was mad about, but I think it being uncompelling aggravated a lot of the audience's biases.

### Rust chauvinism

The article was posted under the title "When to not use Rust - post-mortem on my wiki converter project".

In retrospect it was clickbait: Rust didn't play *that* big a role in the project. I would have had similar results with Go or C++. Rust felt important to me for reasons I outlined in the article; but even then, I knew I was picking the title to be incendiary and didn't stop to reconsider.

I'm not saying it's all my fault, though: the Rust community doesn't like articles about how Rust shouldn't be used sometimes, and they *will* manifest that dislike loudly.

I legitimately think the article would have received much kinder / less abrasive responses if I had omitted any criticism of Rust. A lot of the responses felt defensive, or like the commenters felt a need to put me down to "defend" the legitimacy of Rust, eg:

> However.. none of \[the mistakes you made\] is related to the choice of programming languages.
>
> Things that are related to Rust include eg.: How to handle something that is not Sync. Maybe don't start writing multithreaded programs if that isn't known yet.

Then again, it's hard to be sure about this. It's not like I'm in the head of the people who wrote these answers. I do feel like I've struck a nerve, though.

### Hostility against ChatGPT

The post-mortem details how I used ChatGPT to write the scaffolding of various segments of the project, and as a rubber duck when I was feeling stuck. A lot of my workflow was "do thing -> get stuck on doing thing -> ask ChatGPT what to do next -> read documentation on the things ChatGPT pointed me towards".

(Though I didn't mention the "read documentation" part in the post-mortem, which probably gave the wrong impression.)

And really, that was the whole point: it was a disposable project that I used to experiment with AI-based workflows and see if they could help me progress faster on subjects outside my comfort zone.

People had *opinions* about that. A lot of people thought I had learned the whole lesson from the experiment, and that AI was a shortcut that cost me time in the long run - I'll examine the claim on its merits in a bit.

On an emotional level, though, I think *this* was the source of a lot of the contempt and hostility I saw in the thread. A lot of people *hate* the idea of relying on AI for a programming project.

I've seen this a lot in recent months, but it's always been hard to put my finger on it. As an example, here's [a Hacker News comment](https://news.ycombinator.com/item?id=37643095) that's an almost caricatural example of the mindset I'm noticing:

> Even "smart people" I work with seem to have gulped down the LLM cool aid because it's convenient and it's "cool".
>
> Sometimes I honestly think: "just surrender to it all, believe in all the machine tells you unquestionably, **forget the fact checking, it feels good to be ignorant**... it will be fine...". I just can't do it though.

To me, this is not someone examining competing tools and methodologies with a level head and considering their trade-offs. This is someone caught in a societal trend they don't understand and don't like, someone who feels intensely uncomfortable that people around them are leaving them behind, for better or for worse.

(I mean, the paragraph above is super likely to apply to anyone who uses the words "gulped down the cool aid" unironically.)

This is someone who has staked some of their self-worth, some of their personal pride, in not being a user of LLMs, and is now caught between wanting to preserve that self-worth and wanting to "surrender" to fit in with people following the trend.

To be clear, responses in the post-mortem's reddit thread were a lot more nuanced than this, and I understand that there's something arrogant in saying "People who disagree with me are stuck in the past". Take my words with a spoonful of salt.

But it's an emotional core I've seen before, and I believe I've seen it again in that reddit thread. When someone comments:

> I’ve found that people overly impressed with the AI tools haven’t used them long, or aren’t trying to do anything very complicated. Anyone who is quickly runs into issues and it becomes faster to do it yourself. You still have to know how to implement the thing you’re asking ChatGPT to help with (sorry, business folks, but not sorry).

And someone answers

> My impression is that it mostly feels amazing to people who are beginning to program. They vaguely describe an idea and immediately get a full page of code back that looks reasonable.

To me, this is again people saying "I've staked self-worth in being a better programmer than a mere robot". It certainly doesn't feel like a dispassionate evaluation of tradeoffs.

It gives me the same vibes as an early-2000s professor judging Wikipedia as worthless because random people can edit articles, and there are factual errors sometimes.

The professor has some valid arguments, true. Wikipedia *does* have factual errors. But the professor's position is fundamentally flawed because it comes not from a place of intellectual curiosity, but from a place of comfort with the status quo and rejection of a tool that comes to disrupt it. The professor has been made less essential, and thus feels an urge to emphasize the shortcomings of the tool that would (if only partially) displace her.

Like, let's be real for a moment. ChatGPT has existed for slightly less than *a year*. Even if we hit another AI winter and the underlying LLM never improves, people have *absolutely no business* claiming that it's fundamentally impossible to use it productively.

Maybe if you use it naively (like I did!) you do lose more time than you save double-checking its claims or wasting effort on a dead end because you didn't double-check.

But *even* if we grant that's always the case right now, nothing is telling us that experience can't make us better at using ChatGPT (like it made us better at using Google or Wikipedia!) until it becomes a time-saver. Failure to use ChatGPT to save time should be a source of curiosity and experimentation, not dismissal.

Like, come on! It's a pile of sand that talks! It's a pile of sand that can write shitty code! Have some fucking curiosity! Stop getting on your high horse and claiming loudly that the talking pile of sand couldn't possibly be useful to a *real* programmer and that being impressed by the pile of sand and trying to extract more from it is a sign of naivety!

(To be clear, if you're someone who has legitimately tried for more than 5 minutes to get utility from ChatGPT, if you've applied some creativity to the problem, if you've run actual experiments, and you've found that, nope, it didn't really help for your problem domain, this rant isn't aimed at you. But keep in mind that other people might have gotten more utility from it than you did.)


## Was ChatGPT a net negative to my converter project?

Philosophical grand claims are cool and all, but let's talk about the empirical problem at hand.

As a reminder, my last post was about me writing a converter that would take a wiki and convert it to markdown format. The project was a bit more complicated than just piping to Pandoc: I needed to preserve the revision history of the wiki (hence working with git-oxide/libgit2) and I needed to do some fixing up the links, nothing too bad.

The part that I had the hardest time with was getting past HTTP authentication:

> Now the last step would be to run it on the Mill w-
>
> ```sh
>    Compiling convert_wiki v0.1.0 (/home/olivier-faure-mill/Projects/convert_wiki)
>     Finished dev [unoptimized + debuginfo] target(s) in 4.31s
>      Running `target/debug/convert_wiki 'http://private.millcomputing.com/' mill -r 2`
> 2023-08-25T15:52:49.948354Z  INFO task_get_pages{url="http://private.millcomputing.com//api.php"}: convert_wiki: Fetching pages
> thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: reqwest::Error { kind: Request, url: Url { scheme: "http", cannot_be_a_base: false, username: "", password: None, host: Some(Domain("private.millcomputing.com")), port: None, path: "//api.php", query: Some("list=allpages&aplimit=30&action=query&apnamespace=0&format=json"), fragment: None }, source: hyper::Error(Connect, ConnectError("tcp connect error", Os { code: 113, kind: HostUnreachable, message: "No route to host" })) }', src/main.rs:173:35
> ```
> 
> That means before I could access the Wiki's API endpoints, I would need to figure out how I can get `reqwest` to somehow authenticate to the site.
>
> Uuuuuuuuuuuuugh.

After a conversation with a colleague, this led me to start over and try a simpler solution they'd already made progress on.

As it turned out, this solution didn't completely free me from having to figure out authenticated fetches. So I had a second stab at it:

> The first thing I tried was, wait for it, writing a Rust program.
>
> Since I could connect from the browser, I tried to use Firefox Devtools to record a request and see if I saw anything I could copy into my code.

I fumbled around and found the "Authorization" header, decided to copy-paste it into a very simple Rust scrapper, and got this error:

> ```sh
>    Compiling download_files v0.1.0 (/home/olivier-faure-mill/Projects/download_files)
>     Finished dev [unoptimized + debuginfo] target(s) in 1.98s
>      Running `target/debug/download_files`
> Error: reqwest::Error { kind: Request, url: Url { scheme: "https", cannot_be_a_base: false, username: "", password: None, host: Some(Domain("your-wiki-url.com")), port: None, path: "/w/api.php", query: Some("action=query&titles=File:2004-02-exec-summary-public.pdf&prop=imageinfo&iiprop=url&format=json"), fragment: None }, source: hyper::Error(Connect, ConnectError("dns error", Custom { kind: Uncategorized, error: "failed to lookup address information: Name or service not known" })) }
> ```

I took this as a sign that my approach was fundamentally flawed, but when writing the post-mortem I realized that it was actually almost working; I'd just forgotten to replace the placeholder `your-wiki-url.com` with an *actual* URL.

At the time, I threw the towel again and asked ChatGPT for more advice. My mindset was "Since I can log in from the browser, let's do browser automation", which led me to Selenium, etc.

As far as I can tell, these two sections where I gave up on HTTP auth are the most unpopular parts of the post.

A common takeaway was "You relied on AI instead of thinking for yourself, so you gave up as soon as anything got hard" or more charitably "You relied on AI to avoid writing large amounts of code, so you made errors because you were unfamiliar with the code the AI wrote for you". Either way, the conclusion is "Using AI cost you more time than it saved you".

Is that conclusion correct? Let's examine it.


## Thinking for yourself in the face of unknown unknowns

Let's make one thing clear right away: the problem I was struggling with wasn't hard.

Re-reading my previous article, my "Uuuuuuuuuuuuugh" makes me cringe. That's an expression of laziness in the face of having to learn how something works, when it turns out how it works really wasn't too complicated.

The thing I needed to solve this problem is called the HTTP Basic authentication scheme. It's called that way for a reason.

But!

Hindsight bias is a hell of a drug.

At the time I wrote that program, I had no webdev experience with HTTP authentication, but I did read documentation and error messages.

I read the [MDN page on HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication), which told me I should expect an HTTP 401 error. When I tried to connect to the company server with my browser and sent no password, I got an HTTP 401 error as expected.

When I tried the same with reqwest, I got a `TCP connect error 113 (host unreachable)`. This led me to believe something was wrong between reqwest and the server, not my code.

One reddit user suggested a list of steps they might have followed in my place:

> - When the browser displayed that login window that you show in your blog too, quick narrowing down what this is, to know what I need to replicate in my Rust program. OS, Network, Proxy? No, it is specific to that website. Login text fields like on Reddit, OAuth, JS popups, ...? Nothing like this, it is something the browser knows by itself. Do I have any addons that might treat the site special? No. => Http-level maybe?
> - If I had never heard of "HTTP authentication" before, I might have written that in a search engine, and eg. on Wikipedia or MDN I would've seen that it exists and the most common type is "basic" auth which means username+password. That's how this window looked, yeah.
>
> About EHOSTUNREACH / no route to host / 113, that should be completely unrelated to HTTP auth. (And btw., it's certainly not "identical" to the other DNS error later when you actually had the wrong url)
>
> Some of the things I could ask myself / check, when getting that 113 error:
>
> - Did I change any network config recently
> - Do I have some general internet connectivity
> - Can I reach that wiki website in the browser, right now
> - For that wiki website, or the job in general, do I have some company proxy/VPN configured, either somwhere on OS level or maybe just inside of the browser?
> - What does a DNS resolve in the terminal gives me, with the domain copied 1:1 from my code.

[Can we get serious now?](https://www.youtube.com/watch?v=N1fVL4AQEW8)

While the commenter is being helpful and genuinely answering a question I asked, the above is not a realistic depiction of a debugging process, especially not a process that could have happened with the technical knowledge that I had.

Eg:

- They are assuming that I would connect "needing to input a password" to typing "HTTP authentication" in a search engine (though that's maybe a more reasonable assumption if you're a native English speaker). I might instead have searched for "browser password" or "browser credentials" or "http login", all of which could have sent me down dead ends.
- They are assuming that I would immediately realize that the EHOSTUNREACH was unrelated to authentication. Given that I got this error in cases where authentication failed and didn't get it when authentication succeeded, this isn't at all obvious to me.
- They are carefully including "with the domain copied 1:1 from my code", but it might not have occurred to me that the domain name in my code was wrong. I might have tried typing the domain name by hand and not realized something was wrong.
- They are helpfully pointing out that a DNS error is different from a TCP error. Yes, I noticed that, thanks! The point is that I (perhaps foolishly) skimmed the second error message, because it looked very similar at a glance, and didn't notice the different error types.

My point is, the commenter is replying from the perspective of someone who already knows what the bug turns out to be, and listing out steps that would have made me zero in on that bug.

That's not what happens in real life, especially when you're working outside your comfort zone!

When you're outside your comfort zone, bugs could come from *anywhere*. You have a vague understanding of everything you're using, whether you wrote it yourself or sourced it from AI, and therefore any time a bug pops up you have a much larger "attack surface" to worry about.

(This is something Rust helps with a lot, by the way. By removing entire classes of problems, like UB and stale builds, the language helps you focus on a narrower range of potential error sources.)


## Should I have tried debugging anyway?

Still, even on a difficult project, debugging is always possible. I *could* have spent more time re-reading my code, running small experiments, trying to reach the server with curl, re-writing my scrapper from scratch (it was small enough), etc.

Instead I picked the caveman approach of "Thing A didn't work, thing B worked, I'll try thing B again", and went for browser automation.

In retrospect, the initial approach (using reqwest) was one line change away from working. Still, I think my choice was the right one given the info I had.

Sometimes you don't *know* things are very close to working. Unless your strategy is "never give up or change tactics, ever" then sometimes you're going to give up on things that are more in your reach than you think.

And starting over from scratch has a bunch of advantages. It helps you reexamine your assumptions. It helps you tack the same problem from a different angle, and get more information about it.

In my case, it also gave me experience with Selenium, which is a cool tool that I expect to reuse in other projects.


## Did I make more mistakes because of AI?

Ok, this one I'm legitimately unsure about.

As a reminder, the core mistake I made was using a placeholder `your-wiki-url.com` instead of an actual URL.

This is one of the core worries people have about AI-generated content: that people are going to blindly copy-paste text written by AI and uncritically propagate their mistakes.

(Before ChatGPT, a common concern was "junior developers blindly copy-pasting StackOverflow answers" instead.)

On the one hand, dumb mistakes happen. I've made mistakes like "I wrote a dummy value when I started coding and forgot to change it" before. On the other hand, maybe that mistake would have been more obvious to me had I been the one to write the dummy value instead of copy-pasting it.

Here we come back to the subject of the comfort zone: in a project outside your comfort zone, you're much more likely to get stuck on these dumb mistakes, because you don't have an instinctive understanding of where the mistakes come from.

I can see it when I'm helping my little brother with his projects: I can spot errors much faster than him, even though he's the one who wrote the code, because I know when a certain type of bug is likely to be caused by eg off-by-one errors.

So I don't think the project would have gone faster without AI. I wouldn't have made this one mistake, but I would have probably made other dumb mistakes, and I would have spent more time getting to them.


## Iteration time is king

I think a lot of critics of AI-assisted coding have this model where the best coders are the ones who produce the best code on their first try.

I'm more of a mindset where improvement comes from making errors as fast as possible, Shadok-style.

Chatbot-produced code should be seen with suspicion because it's more likely to contain errors, but for a non-critical project like my wiki converter, making errors is part of the production process.

Getting to the state where you find your first bugs early is important for two reasons:

- It means you spend less time in a state where the project is entirely in your head, and more time on the parts where the rubber meets the road.
- Because you didn't spend too much time on it, you're less emotionally attached to the part you wrote, and it's easier to throw it away and start over.

That second part is major. I sincerely believe that, for the vast majority of projects, doing something wrong ten times before you get it right is better than doing it right once. You learn more, you go faster, you engage with more trade-offs, you notice more bugs.

And it's infinitely better to start a project and get it wrong than to not start at all because you weren't sure what the right way was.

With that mindset, using ChatGPT makes a lot of sense. No, it's not a substitute for programming skills, but it's an excellent aid to start new projects. In particular, I maintain that ChatGPT is a great tool for rubber-ducking.

I can spend days stuck on a project with blank page syndrome because I don't even know where to start or how to frame the problem in my head. There are projects I've given up on because I couldn't get past that writer's block. I know I'm not the only one.

Chatbot AIs can help you get beyond that block. They're worth using for that alone.


# That being said...

The worst sin any open-source developer can ever commit, the sin that leads to all others, is that of hubris.

So let's eat some humble pie, shall we?

If a lot of people got mad at me for using AI and making a bunch of mistakes that AI-skeptics would have expected me to make, maybe these people have a tiny bit of a point?

So let's examine things I should have done differently (besides what I already pointed out in the initial post-mortem).

## Doing too little research

Reading the initial post-mortem, it's easy to assume I didn't read any documentation: I never mention it.

I did read quite a lot, though: a quick search in my browsing history for `docs.rs` gives me `mediawiki`, `pandoc`, `gix`, `bstr`, `git2`, `time`, `tokio`, `tracing-subscriber`, `tracing-bunyan`, `clap`, `serde-xml` and `regex`.

Notably missing from that list is `reqwest`, a pretty bad omission given that most of my program was based on scrapping data through HTTP fetches.

I also didn't read the non-rust documentation for libgit2 or pandoc, but I did read a lot of documentation for the mediawiki REST API.

That said, I think the real problem was the mindset I had when reading documentation. I was too goal-oriented, trying to figure out how to do the thing I wanted quickly, instead of building a holistic understanding of the tools I was using.

That was a mindset that wasn't directly caused by, but was linked to my usage of ChatGPT. I kept asking questions like "How can I do X?" or "How can I modify X to do Y?" and acting on the answers.

These are good questions to start with, mind you: when trying to learn a new tool, "How can I use it to do stuff I want" is the most information-dense question you can ask. By comparison, the tool's official documentation will often have a lot of information the tool's author cares about that you don't really need right now.

But when you've acquired a basic grasp of the tool, reading the documentation becomes essential. "Fail fast, fail often" is cool and all, but it's only useful when you get meaningful information from your failures. Going in with only the information ChatGPT gave you makes your failure harder to learn from.

And when you trap yourself in that mindset, it becomes easy to obsess about fixing the next bug in front of you, and lose track of the big picture.

I think the mindset I had, where I kept asking ChatGPT for a solution for the next problem, while it allowed me to iterate fast, also made it easier to skip doing my homework, even when that homework might have saved me time in the long run.


## Copy-pasting from ChatGPT

In retrospect, a lot of the usefulness I got from ChatGPT was in getting a sense of what direction to go in.

I used the chatbot as a rubber duck, and I asked for boilerplate code to get an idea of what structure the code should have. This felt immensely helpful.

The code itself, though? I didn't actually need it.

I'm starting to consider a workflow where I ask an AI a bunch of questions, throw the answers away, and just use the questions as a launching point to do my own research.

(Kinda like the LessWrong thing with Bayesian calculations, actually.)

Because once you have the direction and the structure in mind, writing the code is extremely fast, and copy-pasting doesn't save you that much time. And there are other benefits to writing the code yourself: it's much easier to memorize something you've written than something you've read, for instance.

And yes, had I not copy-pasted code from ChatGPT, I would not have made the big mistake I mentioned earlier with the dummy URL. Or, well, I could still have made that mistake, but I would probably have caught it sooner.


# Conclusion

This is an article I've written for me, not for other people. I don't intend to advertise it beyond posting it here.

The question I'm pondering is "Given the uncertainty I was navigating, did I apply the optimal strategy (that would also work for equally uncertain situations I don't know about yet)?".

I think this is a hard question to answer, where it's easy to fall prey to hindsight bias. It's easy to make resolutions that are basically equivalent to "next time I'll just avoid making obvious mistakes!" that don't help you improve.

And with that mindset, I think the right strategy is:

- Try a lot of stuff. Make a lot of small experiments, don't commit too much to one. AI helps with that.

- When using a new tool, get the cliff's notes on how the tool is generally used and why. Read the tool's documentation. Here AI can be a trap because it encourages the "how can it solve my immediate problem?" mindset.

- Move towards tools and strategies that give you a lot of feedback. That's why I found Selenium invaluable: maybe it wasn't the most efficient solution for an expert, but it helped me quickly see what was working and what wasn't, and what my code did internally.

  - In particular, you really want good error messages. Try to reduce error noise. One of the things that screwed me was that, using `.unwrap()` the way I was, there was little visual difference between a TCP error and a DNS error. If I'd written my error-reporting code better, I might have noticed the change in error type and thought "Oh, there must be a typo in the URL".

- Avoid directly pulling AI-generated code. If you're using AI generation, compensate by reproducing its output from scratch to be more familiar with it.

And in any case, document everything you can. Documenting things is tedious (though again, ChatGPT helps here by keeping a record of questions you asked the AI) but extremely valuable when you try to improve yourself.

And that's about it. Keep grinding at it until you're the best!

Or something. Do what's best for you.
