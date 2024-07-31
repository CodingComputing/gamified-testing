# Choosing our Missions

We're back as promised, and excited to pass the 3 missions in the `Money` class.

But before we go on to crush the missions, notice what will happen if we keep running the tests.

The `test_init` will keep running each time.

Now, if I'm working on `count`, it's wasteful to run the `init` test every single time.

Also, if I've not worked on `add` and `subtract`, why should I waste time running them every single time?

Can I tell `pytest` to run just a specific test, say `test_count`?
*Yes*

```
python3 -m pytest tests/test_money.py::test_count
```

That specifies first the path to the test file `tests/test_money.py`, and the after the double colon `::`, you write the name of the specific test you want to run.

This will run exactly the `test_count` test, and nothing else.

This is fine, but it gets better.

`pytest` offers much smarter features through its command-line options.

First of all, to skip the previously passing tests, we can employ the `--last-failed` option to the `pytest` invocation.
When we use this options, `pytest` remembers which were the failing tests, and the re-runs happen only on these tests.
This ensures that the tests which were passed in the last run are skipped.
So, only the failing tests are re-run.
`--last-failed` can be abbreviated to `--lf`.

Now, if we wish to work on passing one test at a time, it doesn't make sense to run multiple failing tests over and over again either.
Pytest has an option to prevent that as well.
That option is `-x` (short for `--exitfirst`).
This *exits* pytest at the *first* failure, thus preventing multiple failing tests from running.

Thus, the command

```
python3 -m pytest -x --lf
```
would run just the failing tests, and stop at the first test that fails.

This allows us to run (and focus) on exactly the test that is relevant to our current goal.
By avoding unneeded tests, the test run and feedback is quick.

Now that we know how to run tests efficiently, we're ready to play the game of making them pass.

> [!NOTE]
>
> Our *immediate* goal at this point *isn't* to produce super-high quality (highly readable, optimized, or elegant) code.
>
> The immediate goal is to pass the tests, with just about readable code.
>
> Improvements will come when we revise. Right now, a pass is what we want.

Remember the quote:
> Books aren't written, they're re-written
> - Michael Crichton

Code is similar, it goes through drafts and re-drafts.
At this point, we want a basic working draft, that can be improved on.

Why aren't we aiming for excellence in the first shot?

I have been there and questioned this.

The thing is, aiming for excellence when you're looking at a blank IDE screen *paralyzes* you.

You're thinking of two things: How to make it *work* and how to make it *good*.

Our approach is to first fully commit to making it *work*.
Once it works, we can direct the energy to making it *good*.

*Let's bring it on.*


## Count method

Our first failing right now is the test for `count`.
The purpose of the `count` method is to calculate the total amount represented by the `Money` object.
This is a simple calculation, let's do it.
The first solution that comes to mind is:

```python
# In cash_dispenser/money.py, `Money` class

    def count(self):
        total = 0
        for note in DENOMS:
            total += note*self.cash[note]
        return total
```

Let's now run our test:
```
python3 -m pytest -x --lf
```

Result? `.F`

This means that the first test passed (which was the one that failed last time).
So, bingo! We passed this mission.


## Addition method

But, the next test failed, which is `test_add`.
The reason for Failure is in the pytest report: `TypeError: unsupported operand type(s) for +: 'Money' and 'Money'`

There, that's the next mission, make this one pass.

In order to support the addition of `Money` objects to each other, we need to implement the `__add__` method in the `Money` class.
In case you're not familiar with this, `__add__` is a special method that gets called automatically when you use the `+` operator.
For any two `Money` objects `obj1` and `obj2`,
```python
obj1 + obj2
```
is equivalent to
```python
obj1.__add__(obj2)
```
which is equivent to,
```python
Money.__add__(obj1, obj2)
```

Even if you don't understand that fully, you'll be able to follow along.

Let's first create an `__add__` method, that takes two arguments, so that `+` operation between `Money` objects would be supported.
Right now we won't put any logic in there right now, just define the function

```python
# In cash_dispenser/money.py, `Money` class

    def __add__(self, other):
        pass
```

and run the test
```
python3 -m pytest -x --lf
```

The test fails again (expected because we had only a blank method), but it goes a bit beyond the previous point of failure.
That sounds like progress!

Now, let's implement the method in all seriousness.
We have 2 `Money` objects, `self` and `other`.
As a result of the addition, we need to return another `Money` object.
This object should represent the "sum" of the 2 Money objects.
We can calculate the resultant `cash`, and then create and return a `Money` object, like so:
```python
# In cash_dispenser/money.py, `Money` class,
# replace the __add__ method from earlier

    def __add__(self, other):
        result_cash = {}
        for note in DENOMS:
            result_cash[note] = self.cash[note] + other.cash[note]
        return Money(result_cash)
```

Re-run the last failing test, and see what we get. `.F`

Great, so the test for addition passed!

## Subtraction method

The one that failed is the final subtraction test in the `test_money` module.

Subtraction is totally analogous to addition, and we can jump right to implement it.
`__sub__` is the special subtraction method, and we just need to replace the `+` by `-` in the calculation:
```python
# In cash_dispenser/money.py, `Money` class

    def __sub__(self, other):
        result_cash = {}
        for note in DENOMS:
            result_cash[note] = self.cash[note] - other.cash[note]
        return Money(result_cash)
```

Just re-run and see the result.

YES, the last failing test passed, and that was the final test in the module!

Now, let's just run all tests again, so we can be sure that our new work doesn't mess up the earlier work.
So, we'll run `pytest` without any options, so that all tests are collected and run.
```
python3 -m pytest
```

**Result: 4 passed**

**We did it.**

The `Money` class now has all the functionality that we need.

You should feel like a boss for having compeleted this set of missions!

Notice how the whole process was extremely methodical.

There was never any guesswork about what to do next.

Each line of code that we wrote was totally *intentional*, geared towards passing a test.

The credit for this smoothness goes 100% to 2 things we did earlier:
1. Planned the project and compenents properly
2. Followed a test-first approach, describing the expected behaviour through a test

If you're using git, this is a great time to make a commit!


## Improvements

Now that we have the functionality achieved, let's go for some revisions.
We'll take a second look and see if we can make some improvements.
This is called "refactoring", where we're improving the code structure or efficiency, without any change to functionality.

Looking over the code, I notice that the `count` method can be made more concise.

We can do away with the loop, and use `sum` with a comprehension instead. So, let's try
```python
# cash_dispenser/money.py, replace `count` method inside Money class

	def count(self):
        return sum(note*qty for note,qty in self.cash.items())
```

That's much shorter, but let's check whether it's correct or not.
Checking is extremely easy, with
```
python3 -m pytest
```

All 4 tests still pass!

You see how easy the refactor was.
We just hack at the code, and the tests will tell us if something's wrong.

Similarly, I have an idea for `__init__` as well.
We could replace the `for` loop with a dictionary comprehension.
Worth a try:
```python
# cash_dispenser/money.py, replace `__init__` method inside Money class

    def __init__(self, cash):
        self.cash = {d:cash.get(d, 0) for d in DENOMS}
```

Please note that the point here is not the specific changes I'm making.

The point I want to emphasize is that:
> [!TIP]
> Test allow you to refactor with confidence.

You don't have to wonder if your refactor breaks something.
If it does, the tests will let you know, so nothing to worry about.

Let's run the tests and make sure that things work.
```
python3 -m pytest
```

Sure enough, all 4 tests are green.

So, we completed 4 missions, and we did them well.
That is something to be proud of!

In the next post, we'll explore some more pytest features, and make the `Money` class more robust.

---

- [Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post03.md)
- Next Post (coming soon)
- [List of All Posts](https://github.com/CodingComputing/gamified-testing/blob/main/README.md)
