# Organizing the Tests

So far, we've been working on the `Money` class.
Before we finally move on from the `Money` class, let's organize the tests to make things neater.
We'll take a look at the test file.

After the additions we made in the last post, the file is just over 70 lines.

Not too big, but not tiny either.

I want you to just go through the test functions.
1. `test_init`
1. `test_init_invalid`
1. `test_count`
1. `test_add`
1. `test_subtract`

These are the five.

Now, do you see how some of these tests are strongly related?

For instance, `test_init` and `test_init_invalid` are both related to testing the initialization.

Similarly, `test_add` and `test_subtract` are related to operations on `Money`.
Also, they share their test cases.

Wouldn't it be nice to be able to group these logically, in a hierarchy, instead of them being spread flat in a file?

And, pytest has a solution for that.

What we can do is, group related functions into classes.

For instance, let's put `test_add` and `test_subtract` into a new class, `TestOperations`, like so
```python
class TestOperations:
    test_add_cases = [
		# same as earlier
    ]

    @pytest.mark.parametrize("cash1, cash2, cash_expected", test_add_cases)
    def test_add(self, cash1, cash2, cash_expected):
        # same as earlier

    @pytest.mark.parametrize("cash1, cash2, cash_total", test_add_cases)
    def test_subtract(self, cash1, cash2, cash_total):
        # same as earlier
```

Note that we need to add `self` to each test function, but that's about it.

Similarly, we may group together the two initialization test functions, in a single class `TestInit`:
```python
class TestInit:
    test_init_cases = [
        # same as before
    ]
    @pytest.mark.parametrize("cash, expected", test_init_cases)
    def test_init(self, cash, expected):
        # same as before

    test_init_invalid_cases = [
        # same as before
    ]
    @pytest.mark.parametrize("cash", test_init_invalid_cases)
    def test_init_invalid(self, cash):
        # same as before
```

Now, if you want to run just the `TestInit` tests, you can do so by:
```
python3 -m pytest tests/test_money.py::TestInit
```

If you'd like to run only the test for invalid inputs, that's done with
```
python3 -m pytest tests/test_money.py::TestInit::test_init_invalid
```

That gives you finer control over which test or group of tests you want to run.

But then, what if you want to run a specific subset of tests, which may not be in a single class?

Suppose you are doing some tricky refactoring across multiple methods in the class.
Things are breaking frequently, and the brekage isn't limited to a single test funciton.
At such a time, running just one test isn't sufficient.
Running the last failing test isn't sufficient either.
But you don't want to run the full test suite each time.
You need to run the following 3 tests together: `test_init`, `test_count`, and `test_add`.
Let's call these "sanity tests".
These aren't exhaustive, but they're fast and check basic functionality.
Pytest has a neat feature to select subsets of tests, called *markers*.

Think of a marker as a tag or label on some tests.

You can ask pytest to run only the tests which have a particular marker on them.

We can use any marker name we want. Here is how we mark our sanity tests with a marker called "sanity"
```python
@pytest.mark.sanity
```

Insert this line just above the `def` for `test_init`, `test_count`, and `test_add`.

Now, we'll tell pytest to run only the sanity tests, by using the `-m` commandline option.

```
python3 -m pytest -m sanity
```

You can now see that only 15 tests are run out out 25.

However, `pytest` is giving warnings, saying that you need to register your marker, ie, tell `pytest` that this is a marker that you want to use.

Let's quickly register our `sanity` marker.
Create a new file `pytest.ini`, and paste the following into it:
```
[pytest]
markers =
    sanity: Quick sanity check
```

This tells `pytest` that you'd like to use `sanity` as a marker, and give a little bit information about it, for a human reader.
Let's try the sanity tests again:
```
python3 -m pytest -m sanity
```

And there, the warning is gone. 15 tests are run.
But, how do we know which were the 15?
We can employ the `-v` flag to find out:
```
python3 -m pytest -m sanity -v
```
This gives us the names of tests that have been run, and we can verify that it's indeed the sanity tests.

Now, markers can do a lot more for you.
You can create powerful conditions based on `and`, `or`, or `not` logical operators based on markers.
For instance, to run all non-sanity tests, you'd simply do:
```
python3 -m pytest 'not sanity'
```

Using markers sensibly can give a great deal of flexibility because you can select arbitrary sets from across classes and files.

Another way to select tests is based on the name of the test function or class.
This technique makes use of of the `-k` flag.
Let's see it in action

For instance,
```
python3 -m pytest -k 'count'
```

Will run only the test functions and classes whose name contains `count` (case-insensitive).

You can combine `-k` with `and`, `or` and `not` operators.
You'd probably be thinking we can avoid the markers, and run our sanity tests by name, like so:
```
python3 -m pytest -k 'init or count or add' -v
```
This runs tests who's name contains any of `init`, `count`, or `add`.

Try it.

The problem is, this run `init_invalid` test as well, because it has `init` in the name.

We could fix that by
```
python3 -m pytest -k 'init and not invalid or count or add' -v
```

> [!IMPORTANT]
> As you can see, selection by names is powerful, but tricky.
> Unless you can keep track of all test functions' and classes' names (which I'm terrible at :P), I'd recommend sticking to explicit markers for filtering.

A final tip for filtering:

> [!TIP]
> If you only want see which tests would be collected after a filter, (without executing these tests at the moment), use the `--collect-only` or `--co` flag.
> ```
> python3 -m pytest -k 'init and not invalid or count or add' --co
> ```
> would only collect the tests and display them, without executing them.

This way, you can check your filter before you actually start running the tests.

Notice that when we mark or select a parametrized test, we're running all test cases on that case.
As I did, you may be wondering how to run only select test cases within the list of test cases.
It's not very elegant, but the simplest way I'm aware of, is to place a mark on a test case.
I usually call this mark `one`.
For quick check, I run only this `one` test.
Here's how we place the mark:
```python
test_count_cases = [
    ({}, 0),
    ({100:1}, 100),
    ({100:1, 50:2}, 200),
    ({100:1, 50:2, 20:4}, 280),
    pytest.param( # <=== This test case will be marked...
        {100:2, 50:3, 20:4, 10:1, 5:3, 2:2, 1:7},
        466,
        marks=pytest.mark.one  # <=== with this mark
    ),
]
```

Instead of a tuple, we create a `pytest.param` object, and pass to it a `marks` argument with the desired mark.

Of course, we should register in `pytest.ini`, like so:
```
[pytest]
markers =
    smoke: Quick sanity check
	one: Run just one test case
```

For every test, I mark one test case with `one`, and for a quick test of all features, I run
```
python3 -m pytest -m one
```

I hope you find this useful.

Next time we'll step up the game and start with the `CashDispenser` component.

---

- [Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post05.md)
- Next Post (coming soon)
- [List of All Posts](https://github.com/CodingComputing/gamified-testing/blob/main/README.md)
