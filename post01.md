# "Ugh, I don't wanna learn this Testing stuff!"

Just about a decade ago, that was my exact reaction when I came across a chapter on Testing in a Python book.

And mind you, this wasn't your regular Python book, that just explained stuff to you.

Nope, this was a slightly *unusual* sort of a book[^book] (still very good though!)
This was a book that literally *challenged* you.
The author insisted in each chapter that you type the code by hand (no copy-paste).
He insisted that you read the docs, and threw all sorts of challenges at you at every point.
Although highly beneficial, *every single chapter was a grind*.

[^book]: For those who're curious, the book was *Learn Python the Hard Way*, by Zed Shaw. The book is awesome, but a bit unconventional in its approach. The lessons learned from the 40+ chapters of the book help me to this day. Nothing but love and gratitude for Zed's work.

As a youngster that loved challenges, I had soldiered through quite a big chunk of the book.
In maybe 10-12 weeks, I'd been through almost 40+ chapters at that point (if I recall correctly).
But, as you might imagine, exhaustion and impatience had started to creep in.
In particular, the previous chapter had been heavy.

At that time, my brain worked extra hard, and made a rapidfire list of excuses, faster than ChatGPT.

 - *"This is boring, I want something exciting..."*

- *"I always check the final output of my code. If my code produces that correct output, I'm done. I don't need to put in time to learn testing"*

- *"I want to write code, to build cool stuff, not run some tests. Who in the world wants to learn how to write tests?"*

- *"Anyway, I didn't understand the previous chapter so well either. Why not just skip this chapter and make my text-based game?*"

You see, I wanted to build a text-based adventure game (This is the kind of game where you write commands to take actions: like `go north` to navigate, `punch monster` to fight monsters, etc., inspired by Zork[^zork])

[^zork]: [Zork](https://en.wikipedia.org/wiki/Zork) is a text-based adventure game from the 1970s, still loved after all these years. You can try it [here](https://classicreload.com/zork-i.html#).

So, I talked myself into skipping the Testing chapter, and went off to build the game instead.

What happened is that the game proved to be way more complex than I thought.

I kept making mistakes. But, that wasn't even the main problem.

The main problem was that, because this was a game, if I wanted to check if a certain move or action works, I needed to "play" that action to test it.

This meant typing 2-3 words (something like `pick key`, `open door`, etc.) for each test.
It's some work, but how bad can that be?

The answer is: *Very* bad.

It was a nightmare typing this every time I needed to test a tiny tweak.

I'd make a mistake in the code, **and** I'd make typos in the game commands in a hurry.

It was *insanely* repetitive and frustrating.

Sadly, at some point, this became too much hassle for little enjoyment.
Other things in life took priority.
I gave up on the game project, and also quit the book.

**A couple of years later...**

Push comes to shove.

I *needed* to learn automated testing in an internship.

I started out quite grudgingly.
But then, I started seeing the advantages for myself.

## Why bother with testing?

I was aware of some of the advantages, but hadn't paid attention to them, like...

Unless you write tests for your code,

1. You can't be sure if it works correctly
1. You can't convince others that it works correctly
1. You can't know if a change broke existing functionality

**Testing is a must for any half-serious project.**

BUT, that was not all.

There is one extremely special advantage of testing that I didn't even know existed.

This special advantage?

> [!TIP]
> **Testing turns coding into a game!**

Yes, that is true.

Each test is like a Level or Mission that you set up for yourself.

Step-by-step, you write code that completes these little "Missions".

Watching a test turn green (Pass) is as satisfying as clearing a Mission in a game.

You stack a W with each passing test.
Your brain *loves* to win.
And testing gives it the excitement of multiple wins.

That was when I had this realization:

> I didn't need to write a game to make coding exciting, *testing is enough to **gamify** coding*.

> [!IMPORTANT]
> **Over the next couple of days, I'll be sharing the steps to turn a coding project into a game.**

So, let's dive into it...

## Do I have to change the way I code, in order to test?

No and Yes. Here's why:

No, because you'll write code in Python using exactly the same language features (functions, classes, etc.)

Yes, because coding in a certain style is more suitable to testing.

The great news is that code written in this style is cleaner and better anyway.

> [!NOTE]
> We'll soon explore about **writing *test-able* code, which will also be more maintainable as a bonus**


## Why automate tests with a framework?

1. You *might* actually run them as often as you should
1. You get helpful features (more on that later)
1. Others can run them conveniently

> [!NOTE]
> A good testing framework makes life simpler


## Why is `pytest` a good framework?

`pytest` is a Python testing framework, helping your testing by:
1. Brief tests
1. Simple tests
1. Readable tests
1. Understandable outputs

For every beginner, I'd highly recommend `pytest`.

That is what we'll be using in this tutorial.

Install `pytest` with
```
pip install -U pytest
```
(You can use a virtual environment if you like, but that's not required.)


## How will we learn testing and `pytest`?

Not much point in testing unless we're dealing with a coding project (including mistakes and bugs).

So, let's build a small project, namely a **Cash Dispenser**, piece-by-piece.
As we build, we'll iterate it with testing.

It's a toy project, but it it offers a solid learning opportunity.

Specifically, we'll go through:
1. Setting initial project specifications (aka Defining our Coding Game)
2. Planning, to break down the problem (aka creating Missions in our Coding Game)
3. Initially writing crude code that has mistakes; *this is by design.* (aka Starting the Game)
4. Testing, iterating, and improving (aka Playing and Winning the Game)

Going through this process, you'll get to see for yourself how our testing streamlines things.

> [!IMPORTANT]
> Again, the focus is *not* on building a huge or realistic project here.
> It is on
>  1. learning to write testable code (for quality assurance), and
>  1. getting a *gamified coding* experience

I'm really excited to go through this with you. In the next post, we'll dive into the project!
