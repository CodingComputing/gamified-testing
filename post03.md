# Creating our Missions

Let's now go ahead an create our `Money` class.

Within the `cash_dispenser` folder, create a new file `money.py`.

In `money.py`, we'll create a blank class, `Money`, like so:
```python
class Money:
    pass
```

Let's get right down to the business of creating tests from responsibilities.

Our tests are going to work as follows:

1. We import classes, functions, etc. that we want to test
2. We use these classes (pretending they are complete) and describe how they need to behave

Let's quickly recap our responsibilities of the `Money` class

> - A `cash` attribute that stores how many notes of which denominations are present. A dictionary is perfect for this.
> 
> - A `count` method that calculates the total amount of money in the object.
> 
> - Ability to add 2 `Money` objects to each other
> 
> - Ability to subtract one `Money` object from another

Now it's time to write tests.

The process is easy.

Just *pretend* that the class is written perfectly, and fulfills all the responsibilities.

All we do in tests is to *describe* how this behaviour is.

At this point, we do not bother with how we're going to implement the functionality.
That is for later.

Begin by creating a new file `test_money.py` in the `tests/` folder.

Let's import the `Money` class in here:
```python
from cash_dispenser.money import Money
```
Here `cash_dispenser` is the name of the folder, `money` refers to the `money.py` module, and `Money` is the class defined in the module.

We are going to respresent `cash` as a dictionary that specifies how many notes of each denomination we have (eg, 4 notes of $100, 3 notes of $50, and so on), thus, it is of the form
```
{denomination: quantity}
```

Now, for this to work, the `Money` class needs to know what are the available denominations.

I'm going to assume that the available denominations are: `100, 50, 20, 10, 5, 2, 1`.

Feel free to adapt this to your currency system.

I'll just assume that there is this list of the available denominations in a constant `DENOMS` within the `money.py` module.

I'll import that too (although the object doesn't exist yet. I'm just pretending everything works.)

```python
from cash_dispenser.money import DENOMS, Money
```

Now, we'll start writing our test.

The first task within `Money` would be to initialize the object.
So I'll begin write a test function for that.

How would I want `Money` to be initialized?
I would want to pass it a dictionary of the form `{denomination: quantity}`.
The object initializer would utilize this to construct the `cash` attribute.
I'll name the test `test_init`:

```python
from cash_dispenser.money import DENOMS, Money

def test_init():
    money = Money({100:1, 50:2, 1:7})
    expected_cash = {100:1, 50:2, 20:0, 10:0, 5:0, 2:0, 1:7}
    assert money.cash == expected_cash
```

This is the first "Mission" of our game.
To successfully initialize our `Money` object.

Let's run this test with
```
python3 -m pytest
```

As you might have expected, the test doesn't run successfully.

But it isn't a Failure (ie, assert didn't work).
This is an Error, because the import didn't work.

Now, let's shift gears.

We wrote the test, ie, created a mission for ourselves.

Now the whole focus shifts to making this test pass.

So, let's get back to the `cash_dispenser/money.py` file, and play the game of passing our mission.

Currently, the problem is the broken import of `DENOMS`.

So, we fix it by defining `DENOMS`, like so
```python
DENOMS = (100, 50, 20, 10, 5, 2 ,1)

class Money:
    pass
```

And we re-run the test
```
python3 -m pytest
```

This time, you'll notice a different message.

It isn't an Error, like before, but a Failure.

It will say that it collected one test.

That means our test code ran successfully, but the test didn't pass.
That's an improvement over the Error!

Now, let's focus on turning this Fail to a Pass.

So, we write an `__init__` for `Money`, that takes an argument `cash`.

We'll first just include all denominations as keys, and give values as 0:
```python
DENOMS = (100, 50, 20, 10, 5, 2 ,1)

class Money:
    def __init__(self, cash):
        self.cash = {d:0 for d in DENOMS}
```

This isn't the final code, but let's run the test regardless to see what happens.
```
python3 -m pytest
```

You'll get some output that includes:
```
>       assert money.cash == expected_cash
E    assert {1: 0, 2: 0, 5: 0, 10: 0, ...} == {1: 7, 2: 0, 5: 0, 10: 0, ...}
E
E      Omitting 4 identical items, use -vv to show
E      Differing items:
E      {1: 0} != {1: 7}
E      {50: 0} != {50: 2}
E      {100: 0} != {100: 1}
E      Use -v to get more diff
```

This shows us why our test failed.

It's showing *exactly* the keys and values of the dictionaries that differ.
That is a great advantage of `pytest`.
It gives you a lot of helpful information in a Failure report.

And, you can get even more information by passing the `-v` or `-vv` flags. (`v` stands for `verbose`)
Let's try that to see what it looks like:
```
python3 -m pytest -vv
```

I won't reproduce that whole output here, but if you run it, you'll immediately see that it gives common dictionary items, and the differing dictionary items.

Such a detailed output is invaluable for speeding up the development iterations!

Okay, let's get down to play and fix up our `Money` initializer.
Currently we've set values for all keys to 0.
The test output tells us that we need to set the values to those from the `cash` argument.

```
DENOMS = (100, 50, 20, 10, 5, 2 ,1)

class Money:
    def __init__(self, cash):
        self.cash = {d:0 for d in DENOMS}
        for note in cash:
            self.cash[note] = cash[note]
```

That should do it.
Let's re-run the test to confirm:
```
python3 -m pytest
```

Aaannndd... the test passes!
Congrats for completing the first Mission.

Awesome. Now what we'll do is, we'll quickly convert the rest of the `Money` requirements into test functions:
> - A `count` method that calculates the total amount of money in the object.

I'll simply pretend that my `count` method works perfectly.

I'll whip up a test case with 4 notes of 100$, 3 notes of 50$, and 7 notes of 1$, which totals 557$.

```python
# tests/test_money.py
# imports and test_init just as before

def test_count():
    money = Money({100:4, 50:3, 1: 7})
    result = money.count()
    assert result == 557
```

Let's take this opportunity to study the structure of a test.
In general, a test has 4 parts, but note that every test need not have all of them:

1. Arrange: Where we set up whatever is needed to conduct the test.
    Here this is the line `money = Money({100:4, 50:3, 1: 7})`

2. Act: Where we invoke the exact feature we are looking to test.
    In this case it is the function call and assignment `result = money.count()`.

3. Assert: Where we check the obtained behaviour against a known, expected output.
    In this case it is the assert statement `assert result==557`. For very simple cases, Act and Assert may sometimes be lumped together for brevity, eg., `assert money.count() == 557`

4. Cleanup: Where we wrap up the test, by closing any opened files, etc.
    In this particular example we didn't do anything that requires a cleanup, so we don't have any cleanup code in this test.

At the minimum, a meaningful test should have at least an Act and Assert step.


Before we move on to hack on the code, we'll also make up the tests for these 2 requirements:

> 
> - Ability to add 2 `Money` objects to each other
> 
> - Ability to subtract one `Money` object from another

Let's take simple examples of adding `Money({100:1, 50:1, 20:1})` to `Money({50:2, 10:1})`, with the expected result `Money({100:1, 50:3, 20:1, 10:1})`.

For subtraction, we'll just use `Money({100:1, 50:3, 20:1, 10:1})` minus `Money({50:2, 10:1})`, and expect the result `Money({100:1, 50:1, 20:1})`.

*All we're doing is imagining how we want things to work on specific examples.*
Right now we're not conecerned with *how* the functionality will come.
We're just making up missions.
Playing and accomplishing them comes later.
To this end, we write the following tests:

```python
# tests/test_money.py (contd.)

def test_add():
    money1 = Money({100:1, 50:1, 20:1})
    money2 = Money({50:2, 10:1})
    total_money = money1 + money2
    expected_money = Money({100:1, 50:3, 20:1, 10:1})
    assert total_money.cash == expected_money.cash

def test_subtract():
    money1 = Money({100:1, 50:3, 20:1, 10:1})
    money2 = Money({50:2, 10:1})
    subtracted_money = money1 - money2
    expected_money = Money({100:1, 50:1, 20:1})
    assert subtracted_money.cash == expected_money.cash
```

Note that for the test cases, we do calculations independently, and put down the explicit numbers in the test.

> [!IMPORTANT]
> Do not cut corners and try to "programmatize" this, because:

1. We want our tests to be worked out independently (that's how we verify that the answer is accurate)
2. We want the tests to be as plain and simple as possible. You don't want test code so complex that you have *test the working of the test*.

For practice, in the above 2 tests, try to identify the *Arrange*, *Act*, and *Assert* portions.

Let's run the tests and see what happens. Try it out with
```
python3 -m pytest
```

and the result is `.FFF`.
First test passes (`test_init`) and the last 3 are failing tests.

That means we have 3 missions to complete.

Let's pause this post here.
Next time, we'll crush these 3 missions, and learn amazing `pytest` features.

---

- [Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post02.md)
- Next Post (coming soon)
- [List of All Posts](https://github.com/CodingComputing/gamified-testing/blob/main/README.md)
