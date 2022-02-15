---
layout: post
title: Object-oriented smells
---

[Object-Oriented Programming](https://en.wikipedia.org/wiki/Object-oriented_programming) is widespread in industrial codebases.

This is in part because of inertia: OOP has been the dominant paradigm for decades; an entire generation of programmers has been taught from the onset that OOP was The Right Way To Program.

There are also strong reasons to do OOP: objects are somewhat intuitive abstractions, and encapsulation is a convenient way to separate responsibilities so that a very large team can work on the same code without bumping into each other; inheritance and abstract classes add modularity and reusability to the mix.

But OOP also has strong flaws, which I rarely see discussed. People who criticize OOP tend to do so from the perspective of performance: OOP leads to vtables, heterogenous collections, and sprawling object graphs which put a lot of pressure on CPU caches. But people rarely discuss what I see as the main problem with object-oriented codebases:

**A rigid OOP mindset leads to writing systematically bad abstractions for people reading your code**.

I've noticed these bad abstractions for years, and even written a few drafts for an article like this; but each time I've struggled putting the patterns I was seeing into words. I've recently come across Casey Muratori's [Semantic Compression](https://caseymuratori.com/blog_0015), and it really helped me put a finger on the problems I was seeing:

> First, you look at the real world problem  —  say, a payroll system  —  and you see that it has some plural nouns in it: “employees”, “managers”, etc. So the first thing you need to do is make classes for each of these nouns. There should be an employee class and a manager class, at least.
>
> But really, both of those are just people. So we probably need a base class called “person”, so that things in our program that don’t care whether you’re an employee or a manager can just treat you as a person. This is very humanizing, and makes the other classes feel less like cogs in a corporate machine!
>
> There’s a bit of a problem, though. Isn’t a manager also an employee? So manager should probably inherit from employee, and then employee can inherit from person. Now we’re really getting somewhere! We haven’t actually thought about how to write any code, sure, but we’re modeling the objects that are involved, and once we have those solid, the code is just going to write itself.

I think the above excerpt is a good illustration of "a rigid OOP mindset". It's the mindset where programming has to be about idealized concepts; objects or agents or actors or whatever. Where the programmer shouldn't initially worry about what the specific operations performed by the machine: after all, these operations could be changed at any time to accommodate new requirements!

Instead, in OOP the developer tries to first describe their program as generally as they can, using broad nouns and verbs to describe what their program does (maybe with a UML diagram thrown in for good measure). Only *then* does the developer fill in the gaps, keeping in mind that anything they're writing could be replaced at any time, and so should only be exposed through APIs describing the program's behaviour in very broad strokes.

As an alternate way of thinking, Casey proposes to think in terms of Semantic Compression. I really recommend reading the whole article, but as a tl;dr:

- A lot of developers, including professional developers with lots of experience and a decent salary, have a tendency to create bloated over-engineered abstractions.
- Good code, in its first iterations, is ridiculously simple to read. If your algorithm is "do X, then Y, then Z", then the function that implements it should just have three paragraphs / function calls / whatever for X, Y, and Z, with no abstraction getting in the way.
- By writing code this way, you can notice places where code is copy-pasted, where the same thing is done multiple times, etc, and factor these bits out.
- The factoring is a kind of **compression**; except you're not compressing by bytes, but by **semantic** meaning.
- That compression *should never be done ahead of time*. You should write the uncompressed code, and then refactor it. Premature abstraction comes from the same mindset that produces premature optimization, and is just as harmful.

(See also [John Carmack's mail about inlining code](http://number-none.com/blow/john_carmack_on_inlined_code.html), which makes a similar point about simplicity)

Anyway, I'm not going to retread the points from the above article any further. Instead, I'm going to give you examples of bad abstractions associated with the OOP mindset, which I'm calling "OOP smells". I'll use the notion of semantic compression to analyze these smells.


## Smell 1: Kicking responsibility up the call stack

This is something I've seen *a lot* in OOP codebases.

What I mean is, in a case where a method executing a task needs additional data/resources to complete it, the method will often call some other method of a class that owns the data/resources, and give that other method both the responsibility of completing the task as well as all the arguments it received.

To demonstrate this smell, let's imagine a simple use-case: we're building a chat service, and we want a function that will open a salon owned by a given user. Given a user's id, the function fetches the user's metadata in a database, then sends the metadata to an opaque service that will open the salon.

Let's say we already have a `UserManager` class, a `Database` class with a `fetchMetadata` method, and an `OpaqueServiceWrapper` class with an `openSalon` method. A simple solution might look like this pseudocode:

```c++
void ChatService::openSalon(int userId) {
    auto userData = m_userManager.getMetadata(userId);
    m_opaqueServiceWrapper.openSalon(userData);
}

Metadata UserManager::getMetadata(int userId) {
    return m_database.fetchMetadata("TABLE=users", userId);
}
```

But many OOP pundits will argue for a different approach.

They will say that this code isn't future-proof. What if the way to open a salon changes, and requires other information from the user database? The ChatService class shouldn't have to worry about what metadata UserManager stores, after all. It should [only tell](https://www.martinfowler.com/bliki/TellDontAsk.html) UserManager to open a salon, and UserManager can decide what information to fetch from the database.

The extreme OOP approach would look more like this:

```c++
void ChatService::openSalon(int userId) {
    m_userManager.openSalon(userId, m_opaqueServiceWrapper);
}

void UserManager::openSalon(int userId, OpaqueServiceWrapper& service) {
    auto userData = m_database.fetchMetadata("TABLE=users", userId);
    service.openSalon(userData);
}
```

Now, the code that gets executed is exactly the same. We have a `fetchMetadata` call followed by a `openSalon` call.

But I would argue that the second approach is a strictly worse one:

- Reading `openSalon` alone, it's more difficult to understand what resources are used.
- The second signature went from  `Metadata UserManager::getMetadata(int userId)` to `UserManager::openSalon(int userId, OpaqueServiceWrapper& service)`, which is less expressive.
- Instead of `openSalon` and `getMetadata`, our methods are named `openSalon` and `openSalon`.

In a nutshell, **our code lost information density**. It became harder to grok the broader structure by looking at few lines of code. We changed our code to be "more future-proof", but it didn't compress information that was being reused, and it's unclear whether it could actually help compress future code.

Outside of simplified examples, this smell manifests in real-life codebases that are full of one-line functions that perform almost no computation and have the same name, and just pass on their arguments to another identically-named function, eg:

```c++
void GuiManagerSystem::openTab(Foo foo) {
    LocalGuiSystem::openTab(foo);
}

// ...

void LocalGuiSystem::openTab(Foo foo) {
    WindowManager::openTab(foo, getBar());
}

// ...

void WindowManager::openTab(Foo foo, Bar bar) {
    getSelectedWindow().openTab(foo, bar);
}

// and so on...
```

(I wish I had actual real-life examples to show, but my experience with this was on corporate codebases that I didn't have permission to share and have no longer access to; if anyone has access to a real example of that smell, I'm super interested)

Other symptoms of this smell:

- Limited code reuse.
- Unit tests are hard to write and require shimming.
- Deep call stacks.
- Lots of virtual method implementations that do nothing but call another implementation of the same method.
- Bloated leaf functions.


## Smell 2: Optional side-effects instead of optional returns

This smell occurs when a function needs to produce some data in some use cases and produce nothing in others, and differentiating them is non-trivial.

The functional way to achieve that is to return an optional type (`std::optional<T>`, `Maybe<T>`, etc) with the data. The imperative way, often used in OOP code, is to mutate a reference or call a side effect inside an 'if' block.

For instance, let's imagine we're in a video game, and we want to upload the replay of a player's best moments at the end of a game; except in cases where the game can't generate a replay (maybe the player was on a bad day), in which case we send nothing.

A functional-ish way to do this might look like:

```c++
void GameHandler::endOfGame() {
    std::optional<Replay> replay = m_gameEngine.createReplay();
    if (replay) {
        m_server.sendReplay(*replay);
    }
}

std::optional<Replay> GameEngine::createReplay() {
    if (!m_internalData.hasPlayerActions)
        return std::nullopt;

    let actions = m_internalData.getPlayerActions();
    if (actions.is_empty())
        return std::nullopt;

    actions.sortByCoolness();
    if (!actions.first().isCoolEnough())
        return std::nullopt;

    return makeReplayFromAction(actions.first());
}
```

In the OOP mindset, this is bad because we're *asking* the game engine for data, instead of *telling* the engine what to do with it. An OOP approach might look like:

```c++
void GameHandler::endOfGame() {
    m_gameEngine.sendReplayToServer(m_server);
}

std::optional<Replay> GameEngine::sendReplay(Server& server) {
    if (!m_internalData.hasPlayerActions)
        return;

    let actions = m_internalData.getPlayerActions();
    if (actions.is_empty())
        return;

    actions.sortByCoolness();
    if (!actions.first().isCoolEnough())
        return;

    Replay replay = makeReplayFromAction(actions.first())
    m_server.sendReplay(replay);
}
```

I think the OOP approach is flawed, because, aside from the problems I already mentioned in the first section, it conflates producing the optional data with using it.

Let's say the requirements change:

- We want to discard the data in some use-cases (maybe with some settings we don't send replays older than 5 minutes).
- If we don't generate a replay, we want to send a default value instead.
- In some cases, we want to edit the replay before sending it.

The first approach will look like:

```c++
void GameHandler::endOfGame() {
    std::optional<Replay> replay = m_gameEngine.createReplay();
    if (m_filterOldReplays && time::now() - replay.timestamp() > time::minutes(5))
        replay = std::nullopt;
    if (replay && date.isHalloween()) {
        replay->giveEveryoneSpookyHats();
    }

    if (replay) {
        m_server.sendReplay(*replay);
    }
    else {
        m_server.sendReplay(ShrugEmote());
    }
}

std::optional<Replay> GameEngine::createReplay() {
    // same
}
```

With the OOP solution, we do this:

```c++
void GameHandler::endOfGame() {
    m_gameEngine.sendReplayToServer(m_server, m_filterOldReplays, date.isHalloween(), ShrugEmote());
}

std::optional<Replay> GameEngine::sendReplay(Server& server, bool filterOldReplays, bool isHalloween, Replay placeholder) {
    if (!m_internalData.hasPlayerActions) {
        m_server.sendReplay(placeholder);
        return;
    }

    let actions = m_internalData.getPlayerActions();
    if (actions.is_empty()) {
        m_server.sendReplay(placeholder);
        return;
    }

    actions.sortByCoolness();
    if (!actions.first().isCoolEnough()) {
        m_server.sendReplay(placeholder);
        return;
    }

    Replay replay = makeReplayFromAction(actions.first())

    if (filterOldReplays && time::now() - replay.timestamp() > time::minutes(5)) {
        m_server.sendReplay(placeholder);
        return;
    }

    if (isHalloween)
        replay.giveEveryoneSpookyHats();

    m_server.sendReplay(replay);
}
```

Notice that our `GameEngine::sendReplay` method has suddenly become a lot more complicated. We must add function arguments for conditions that GameEngine doesn't have access to (user settings, whether a game event is live, etc), which complicates the prototype. At the same time, we now have to bundle every return with a `m_server.sendReplay(placeholder);` (unless we use the `goto fail` approach, or give up on early returns and just use increasingly nested if blocks).

By contrast, in the functional approach, our `createReplay` method stayed unchanged, because the logic that creates a replay stayed the same. Only the logic that decides how to use this replay changed, and the logic is cleanly separated in the `endOfGame` method.

I have a harder time fitting this OOP smell into a neat moral lesson about how semantic compression The Right Way To Code. It feels a little tangential.

But still, this *is* an example of an anti-pattern that I've seen in the wild, sometimes put there by people who actively justified its presence with OOP truisms. It's a side-effect of people thinking of abstraction not in terms of what existing logic they can combine, but in terms of what program behaviours they want to represent, and then adding more parameters and special cases to these behaviours.


## Smell 3: Lots of virtual methods that do nothing

Object-oriented enthusiasts are often wary of switch blocks and chained else-ifs. One of the precepts of OOP is that, if you have multiple ways to do something, you should have an interface with a `doSomething()` method, and multiple classes implementing that interface.

This starts breaking down when you add a lot of possible behaviours, and most of them are optional.

For instance, let's imagine we're building a game engine, and we're working on an entity class. At first our entity only has a few methods, like `update()`, `getPosition()`, `getSpeed()`, `render()`, etc.

Then, as the game progresses, we add new methods. Maybe the game needs entities to react to explosions (players take damage, physic objects are blown away), so we add a `reactToExplosion()` method. Maybe we add other environmental effects, so we add a `catchFire()` and a `getFrozen()` method; maybe we want to add a special way to move entities by teleporting them.

At the end, our base class might look like:

```c++
class Entity {
public:
    virtual void update();
    virtual Vec3 getPosition();
    virtual Vec3 getSpeed();

    virtual void setPosition(Vec3);
    virtual void teleport(Vec3);

    virtual void render();
    virtual void startAnimation(string animationName, bool loop);
    virtual Animation* getAnimation();
    virtual float getAnimationProgress();

    virtual void catchFire();
    virtual void getFrozen();
    virtual void takeDamage(int damage);
    virtual void getKilled();
    virtual int getHealth();
    virtual bool isDead();

    // etc
}
```

The problem with the above is that we might have a lot of entities that implement some, but not all of these methods, so the method does nothing or returns a default value. A physics object doesn't have health or animations. Maybe a player object manages its position internally and setPosition should never be called on it.

This is bad because it makes it harder to locate where the code *actually* does things; by trying to be extremely general, you've drowned out useful code into a sea of unused abstractions.

To continue the compression metaphor, this is like compressing a file with an ill-suited algorithm. Not only have you made the file larger, reading it has become slower because first you must uncompress it to remove the additional metadata cruft.

One solution is to split the Entity class into multiple base class (like an `AnimatedEntity` class, a `KillableEntity`, a `PlayerEntity`, etc, and have all these classes inherit a base `Entity` class). This adds the problem of diamond inheritance (eg what to do if an entity is both animated and killable?), and still makes it hard to reason about what a piece of code can or cannot do.

So we want to add lots of extensible behaviours to an entity, without creating an unreadable class hierarchy. The common solution for this is "Compose, don't inherit". In modern game engines, this usually takes the form of ECS (Entity Component System), a framework where every entity can optionally store instances of components, and whether or not a behaviour is implemented depends on whether the entity has the matching components.

So in our example, we might have a `Killable` component, an `AnimationHolder` component, etc.

ECS comes with its own downsides, which could fill an entire article. Speaking from my very limited experience, I'm not actually sure ECS is actually a good tool for semantic compression. But it has many advantages (especially in terms of performance), and thus it is ubiquitous in modern game engine design.


## Conclusion

This article is a bit more unstructured that I would have liked.

I've been brewing it for a few years, but reading Casey's own article is what helped me put most of my gripes and preferences into words.

If there's one point I want people to take away from it, it's that **abstractions has a cost**.

Just like you can't compress arbitrary files with arbitrary algorithms without sometimes ending up with a larger file, you can abstract code away blindly without sometimes ending up with obtuse code. Encapsulation isn't an automatically good thing.

Write dumb code first. Then compress it. It will make your life easier.
