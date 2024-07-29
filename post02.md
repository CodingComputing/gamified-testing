# Defining our Coding Game: Cash Dispenser

Growing up, I had loved to play sports and games, that had very different objectives,

- Football: Get the ball past the opposing goalpost
- Basketball: Pass the ball through a hoop
- GTA: Become the underworld boss
- Age of Empires: Build a kingdom
- Tycoon: Build a winning business

...and on and on and on.

What is the *only* common thing between all of these?

It is that there is a clear goal that you're aiming for.

Your actions are focussed on achieving that one goal.

That's how you set yourself up for success and enjoyment, because without the clear goal, you're just wandering about (not productive or exciting).

It's the same in coding and testing too.

*Before* we begin coding, we have to understand what we're aiming for. In software, we call this the *requirements*.

As the backdrop for testing, we're going to build a Cash Dispenser, that simulates a part of an ATM.
Let's fix the requirements for what we're building and testing.

To keep things simple, here is what we'll assume the following:
1. This component won't directly interact with the user, but with another software component. We'll call this other component the Driver (we won't create it in this tutorial).
1. The Driver and other software components will take care of user authentication, checking account balances, etc.
1. Our component is only concerned with keeping track of cash notes in the machine, and dispensing the amounts requested by the Driver.

Once we have defined this high-level purpose, it's time to figure out the specific duties of our component.
These are known as the *repsonsibilities* of the component.

Specifically, our component's main responsibilies are:
1. Keep track of how much cash (and in what denominations) is present in the ATM
1. Accept deposited cash
1. When an amount is requested, try to dispense that amount with the available notes, preferably with a small number of notes. If successful, return a message to the Driver about the Success, and which notes to dispense.
1. If the amount cannot be dispensed, then return a message to the Driver about the Failure, with any other helpful information


**But why are we discussing all this, instead of jumping directly to testing?**

That's because unless you have the responsibilities of your component, you'll have a hard time writing and passing the tests.

Think of requirements as the "objective" of the game. Once we have the objective set up properly, then we can move on to design the "missions" of our game, ie, the tests.

This is the easist, quickest, and the most natural progression:
```
Game objective --> Missions --> Play --> Win
```
Similarly:
```
Requirements --> Tests --> Code & Run tests --> Pass tests
```

Let's now move on to some planning so that we can design the "Missions" of our coding game.


## Planning the project

In almost any game worth enjoying, you don't fight the final boss as soon as you start.

Rather, you progress through levels, rising through ranks, collecting resources, or whatever.

In other words, you want a *plan*, where the overall objective is broken down into missions.

That's what we're going to do now: Break down the component responsibilies into "missions" that we'll conquer one at a time.

We want every "mission" to be very focused and specific. It's either a Pass or a Fail. Nothing else in the middle.

> [!TIP]
> Figure out most of what you're going to do *before* opening up your IDE.

So, let's do just that.
We'll break down our project into small, manageable, and, most importantly, *testable* chunks.

> [!IMPORTANT]
> **A *testable* chunk is just a focused mission.**
> It's clear what is expected, and there is an unambiguous Pass/Fail criteria.

There are multiple correct ways and approaches to this, and here is one way to go about it:

First of all, to bundle up all this functionality, we'd want to create a class `CashDispenser`.
Roughly, it needs to have:

- A `money` attribute to represent money. This is how the object will keep track of the money in the machine. We'll need to figure out a way to represent money, as a collection of specified denominations and quantities (eg. 4 100$ notes, 1 50$ note, 2 20$ notes, and so on.)

- An initializer, that will indicate how what are the different notes in there to start with. Primarily, this will initialize the `money` attribute.

- A `deposit` method, that will accept money and update the `money` of the machine.

- A `dispense` method, that will accept an amount from the driver, and attempt to dispense the given amount, using the available stash of notes in the machine's `money`. The method will return a status to the driver, that indicates Success or Failure, and other information.

That's about it.

As we think about the above, it is clear that money is the core component here.
It is almost as if every operation of `CashDispenser` is based upon some manipulation of money.

Therefore, it makes sense to have a dedicated `Money` class, which will have supporting operations.

We can give the following responsibilities to the `Money` class objects:

- A `cash` attribute that stores how many notes of which denominations are present. A dictionary is perfect for this.

- A `count` method that calculates the total amount of money in the object.

- Ability to add 2 `Money` objects to each other

- Ability to subtract one `Money` object from another

We'll figure out the specifics as we go, but this is a great foundation to start building.

With that, we're ready to go for it!


## Setting up

A clean directory structure is crucial for organizing the project.
Let's start there.

Create a folder `cash_dispenser_project`, on your Desktop or some place you'll remember.

If you know how to set up a virtual environment and git repo, this is a good time to do so.
If you don't know that those things are, just move on, they're not required to complete the tutorial.

Within this folder, create 2 more folders: `cash_dispenser` and `tests`.
(If this is going to be a big project, you'll do things a bit differently[^package], but this is fine for a learning project.)

[^package]: For big projects we prefer to turn the project into a Python package. For a beginner project this is overkill.

Your directory tree should look like this:
```
cash_dispenser_project/
├── cash_dispenser/
└── tests/
```

As the name suggests, `cash_dispenser` will contain our program code files, and `tests` will contain our test files.

This is good, we keep our tests separate from code.

If you haven't already done so, install `pytest` with

```
pip install -U pytest
```

Finally, it's time to run `pytest`, just to make sure it's there.

```
python3 -m pytest
```

You should get some output like so:
```
============================= test session starts ==============================
platform linux -- Python 3.10.12, pytest-8.1.1, pluggy-1.4.0
rootdir: /path/to/cash_dispenser_project
plugins: anyio-4.3.0
collected 0 items

============================ no tests ran in 0.00s =============================
```

Pytest has inbuilt functionality to look for tests.

If it sees a `tests/` directory, it knows that it should look for testing code in there.

Within this directory, it will look for files whose names start with `test`.

At the moment, Pytest is telling us `collected 0 items`, that means it didn't find any tests.
That should be obvious, because we haven't written any yet!

So let's write some dummy tests just to understand how things work.

At its core, a "test" is just a function, that effectively says "this is what should happen". 

This is done with the help of an `assert` statement.
Let's see this in action.

Go ahead and create a file in the `tests/` directory, and name it `test_dummy.py`.

In this file, we'll create 2 functions, like so:
```python
def test_passing():
    assert 3 == 3

def test_failing():
    assert 1 == 5
```

Just 2 functions. One says `assert 3 == 3`, the other says `assert 1 == 5`.

`assert` is how we say "this is what should happen to make this test pass".

Thus, `test_passing` will pass if `3 == 3`, else it will fail. (Obviously this will pass)

`test_failing` will pass if `1 == 5`, else it will fail. (Obviously this will fail)

Let's re-invoke Pytest.
```
python3 -m pytest
```

> [!IMPORTANT]
> Note that this command must be used from the `cash_dispenser_project` directory, not the `tests`.

Here's what we get:
```
============================= test session starts ==============================
platform linux -- Python 3.10.12, pytest-8.1.1, pluggy-1.4.0
rootdir: /path/to/cash_dispenser_project
plugins: anyio-4.3.0
collected 2 items

tests/test_dummy.py .F                                                   [100%]

=================================== FAILURES ===================================
_________________________________ test_failing _________________________________

    def test_failing():
>       assert 1 == 5
E       assert 1 == 5

tests/test_dummy.py:5: AssertionError
=========================== short test summary info ============================
FAILED tests/test_dummy.py::test_failing - assert 1 == 5
========================= 1 failed, 1 passed in 0.06s ==========================
```

Our test failed. So sad (*LOL!*)

Let's observe the output:
```
collected 2 items
```
means Pytest found 2 tests.
This was expected, since we wrote 2 functions in the `tests` directory.

Next we see
```
tests/test_dummy.py .F                                                   [100%]
```

The `tests/test_dummy.py` tells you which file(s) contained tests.

`.F` is the brief representation of test results. `.` indicates passed, `F` indicates failed.
Thus, `.F` means the first test (`test_passing`) passed, and the second test (`test_failing`) failed.
This was expected from the `assert` statements we wrote.

Finally, under the `FAILURES` banner, we get the details of the tests that failed.

This is not for discouragement, it is so that we can easily spot what needs to be fixed.
In our case, this is:
```
    def test_failing():
>       assert 1 == 5
E       assert 1 == 5
```
It says that this assertion went wrong, which we can see quite clearly.

Feel free to write some more tests with other assertions.

Once you are done with experimenting, delete this file.
We won't need it in the upcoming parts.

Now that you got a preliminary taste of `pytest`, the plan going ahead is:

1. For each responsibility, we'll pretend that we have working code fulfilling it.

2. Pretending just like that, we'll write tests for the responsibility. Obviously, tests will fail.

3. Then, we'll stop pretending, and play the game of writing code to make the tests pass.

And just like that, we'll have a working project! It's so simple that you'll need to see it to believe.

In the next post, I'll be showing you *exactly* how, so stay tuned!

---

[Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post01.md)
