---
layout: post
title: About political messages on the Rust blog.
---

Alright, let's mention the elephant in the room.

Earlier this week, the article announcing the 1.65 release of the Rust programming language included this little aside:

> Before going into the details of the new Rust release, we'd like to draw attention to the tragic [death of Mahsa Amini](https://en.wikipedia.org/wiki/Death_of_Mahsa_Amini) and the death and violent suppression of many others, by the religious morality police of Iran. See https://en.wikipedia.org/wiki/Mahsa_Amini_protests for more details. We stand in solidarity with the people in Iran struggling for human rights.

This isn't the first time the first blog has mentioned world events. In the 1.59 release announcement, they said this:

> Today's release falls on the day in which the world's attention is captured by the sudden invasion of Ukraine by Putin's forces. Before going into the details of the new Rust release, we'd like to state that we stand in solidarity with the people of Ukraine and express our support for all people affected by this conflict.

And earlier, for the 1.44 release:

> This is a shorter blog post than usual: in acknowledgement that taking a stand against the police brutality currently happening in the US and the world at large is more important than sharing tech knowledge, we decided to significantly scale back the amount of promotion we're doing for this release.

(There might have been others I missed.)

Each of these asides has been... controversial, to say the least. Many people are opposed to the use of a tech forum to broadcast political messages. And these asides have each been posted as-is, with no justification behind them and no hint that a justification is even necessary or wanted.

There's a clear disconnect in the community: for some people, these messages are so self-evident that they shouldn't even be considered political. For some people, they reflect a worldview that they don't necessarily agree with, and when they don't, still don't want to be pushed without their input in the name of a community they're participating in.

Also, let's stop mincing words for a second: a big part of the issue is that these messages are progressive, and reflect a progressive agenda. A lot of people in the Rust community aren't progressive, or not progressive of the specific US flavor that dominates western social media, and so feel bullied when the Rust blog makes these political statements.

(EDIT: I first said "and so feel extremely threatened". People pointed out that the turn of phrase was pretty rich given that people in Iran/Ukraine/Palestine/etc being, you know, *actually* threatened with death. I agree it was a poor turn of phrase and I apologize. I do think some people feel like that the progressive drift of political discourse is a threat to them and their values; but I think they're unreasonable (or outright toxic) and I don't want to validate them.)

Anyway, it's a difficult subject to discuss with a cool head, and the community has retreaded arguments for and against ad nauseam. So I thought I'd write a short summary of the arguments that come up most often.


## Are politics relevant in tech?

Some people don't want any political messaging at all.

They believe that Rust is a technical tool, and its release announcements shouldn't serve as a peanut gallery for ongoing civil unrest.

The main counter-argument is that technology is *inherently* political because tools such as machine learning can be used by governments or corporations in ways that are politically charged.

The counter-counter argument is that while *some* tech is directly political (eg face identification tech), in this specific instance, civil unrest in a single country has no obvious relation with a new compiler release of a general-purpose programming language.

Another argument I've see is that, even if Rust isn't usually political, human right issues are worth mentioning anyway *because* they are so important that they're worth crossing the usual boundaries of discussion.

Maybe it's worth making some people uncomfortable to signal-boost a message that people otherwise wouldn't hear because of their echo chamber.

The counter-argument to *that* is that issues mentioned on the blog have been high-profile and impossible to miss for anyone following mainstream US media. Arguably, the blog isn't so much signal-boosting "X happened" so much as signalling "we take this side on X".

(That last part has been a sore subject in tech circles lately, regarding the Black Lives Matter protests. A lot of sites have added banners supporting the movement. A common complaint was that these banners made the site's UX worse, not to help the user of the site, but to signal to everyone that the site owners were on the *good* political side.)


## The choice of subject

The three subjects that have been deemed worth a mention so far definitely feel partisan.

The asides are US-centric, focus on subjects of interest in the US discourse and are US-sided and progressive.

The blog has had an aside on the death of Mahsa Amini in Iran, but not on the brutal murder of Jamal Ahmad Khashoggi by US ally Saudi Arabia. More recently, the blog had nothing to say about the political persecution and extradition of Julian Assange to the US. The blog hasn't mentioned the criminal murder of Shireen Abu Akleh by the Israeli military.

Yes, the issues the blog mentioned were real human rights issues, but **selective coverage of human rights is propaganda**. It serves a specific, US-centric agenda. To people who don't agree with that agenda, it can feel like they're receiving propaganda they didn't ask for, that they don't agree with, and that they're still being made to give a platform to by participating in the Rust community.

Also, it deserves to be said, but sometimes people are wrong? Sometimes the message being boosted will not be consensual.

Imagine if the Rust team had, in 2002, broadcast a message saying "We stand with the US intervention in Irak to remove the weapons of mass destruction owned by the regime"? Or in 2020, "We stand with the peaceful protesters fighting for democracy in Washington DC, and we strongly believe that all accusations of election fraud should be taken seriously and investigated"?

It's not obvious what process the Release Team is using to decide whether an issue is partisan or consensual enough to be signal-boosted.


## My own take

That being said, it can be argued that some imperfect human rights coverage is better than no coverage at all.

As I said above, maybe human rights are important enough that we should talk about them even when it's not quite convenient or polite.

Here we get into my personal opinion: some things are *right* and some things are *wrong*, and if your mind is clear enough, it's possible to tell them apart regardless of your ideology.

Putin's unprovoked invasion of Ukraine and the tens of thousands of deaths that followed were *wrong*. Assange's imprisonment on bogus charges, and his sham extradition trial are *wrong*.

(EDIT: Some people have (very politely) taken issue with my formulation of Putin's invasion as "unprovoked". Well, too bad, I'm not a language's official blog, so I get to have strong political opinions. And my opinion is that "having wanted to join NATO for decades" does not constitute a "provocation", let alone one worth invading a country over. If you're mugging someone at gunpoint, and they call their neighbor with a hunting rifle, you don't get to shoot them and then complain that people call it "unprovoked murder".)

Sometimes things are wrong enough it's worth spending the time to hammer them in, even if that creates controversy and hurt feelings.

But there has to be a *process*. If you really want to push human rights, and not just a watered-down version of the US progressive agenda, you need to have some way of selecting worthy causes that isn't just whatever was last on mainstream channels. You need a set of criteria for what is and isn't worth mentioning. These should be documented, and ideally be something other than "Whenever the release team feels like it."

Tech doesn't have to be political. A compiler can just be a compiler. But if you're going to do politics, the least you can do is not half-ass it.

[Discussion on r/rust.](https://www.reddit.com/r/rust/comments/ymrpah/about_political_messages_on_the_rust_blog/)
