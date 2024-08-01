# Adding Variety to Missions

So far, we've done a good job on one component of the project, which is `Money`.

We wrote 4 tests to verify the basic functionality, and passed all of them.


## Parametrized Tests

But, some of you may be wondering:

"Fine, it works on those specific examples, but what about some others?"

In short, passing a test with one example isn't sufficient to convince people.

The solution? Test with more examples. The examples are called "*test cases*".

Now, I could just duplicate the same tests over and over again, and change some values to make new test cases.

But, that doesn't sound like a great approach.

> [!TIP]
> As a dev, always be wary of duplication!

For example, our `tests/test_money.py` currently stands at 25 lines.

If I were to repeat each test say, just 4 more times, the size of the file would go above a 100 lines.
That is to say, the copy-paste approach to increasing test cases isn't easy to manage.

But worry not, pytest has this covered.

Pytest gives a feature of *parameterizing* tests, ie, testing multiple test cases with a single test function.

The basic idea is this: We represent each test case as a tuple,
```
(test_case_data, test_case_expected_result)
```

We maintain a list of such test cases for each test function.

The test function is then run repeatedly for each case in the list.

Let's see this in action, for `test_count`. We'll make a list of tuples. The first item of each tuple would be a `cash` dictionary, and the second would be the counted money:
```python
test_count_cases = [
    ({}, 0),
    ({100:1}, 100),
    ({100:1, 50:2}, 200),
    ({100:1, 50:2, 20:4}, 280),
    ({100:2, 50:3, 20:4, 10:1, 5:3, 2:2, 1:7}, 466),
]
```

Now that we have the test cases ready, we can now begin to create our parameterized test.
To be able to use this feature, we'll need to import `pytest` into our test file, with
```python
import pytest
```

Parametrization is best presented through an example.
Don't get overwhelmed, we'll understand it in detail later, but it helps to see what it looks like:
```python
# tests/test_money.py; only showing test_count and it's parametrization

import pytest

test_count_cases = [
    ({}, 0),
    ({100:1}, 100),
    ({100:1, 50:2}, 200),
    ({100:1, 50:2, 20:4}, 280),
    ({100:2, 50:3, 20:4, 10:1, 5:3, 2:2, 1:7}, 466),
]

@pytest.mark.parametrize("cash, expected", test_count_cases)
def test_count(cash, expected):
    money = Money(cash)
    assert money.count() == expected

```

Let's break this down.

In the list `test_count_cases`, we're simply storing the data for each test case.
A bunch of tuples, each having one `cash` dictionary, and one excpected answer.

Then, we want to tell `pytest` that we want to parametrize `test_count`.
We do this by placing `@pytest.mark.parametrize` on top of `test_count`.
This `@` syntax is called a decorator.
If you don't know about decorators, it doesn't matter at the moment.
Just note that this is how you tell pytest that you want the coming test to be parametrized.

Now, what are the arguments that go to `pytest.mark.parametrized`.
Let's start with the second argument.
This is just the list of test cases we created before.
That's all there is.

Now the first argument.
This is where we unpack each test case tuple into variables.
For example, the first test case tuple is `({}, 0)`.
How do we convey to pytest that the first item of the tuple is a `cash` dictionary, and the second item is the expected count?

That is the job of the first argument.
It is unpack this test case tuple into variables, which will be used to conduct the test.
The string "cash, expected" tells pytest that each test case tuple is to be unpacked into 2 variables: `cash` and `expected`.
If you're familiar with tuple unpacking, this should feel right at home.
Otherwise, it's the simple concept of breaking down the test tuple.
The first item should go to `cash`, and the second item should go to `expected`.
Now, once we have these values unpacked, what next?

Next comes the step of passing these values to the test function.

Recall that earlier our test functions didn't have any parameters.
Now, we pass these unpacked values to this function. So, now it's:
```python
def test_count(cash, expected):
    # test function
```

Lastly, the easy part of conducting the test.
Earlier we had hardcoded values into the test.
Now, we've hardcoded values into the test case list, and are extracting them into variables.
All we have to do is conduct the test with these variables.
In this case, it means to use the `cash` dictionary to create a `Money` object, run its `count` method, and compare the result with `expected`, like this

```python
def test_count(cash, expected):
    money = Money(cash)
    assert money.count() == expected
```

> [!NOTE]
> If you didn't understand all of this, *it's okay*.
> Just go through these motions a couple of times, and it will come naturally to you.

Let's now parametrize the `test_add`.

This test essentially involves 3 `cash` dictionaries.
1. `cash` for the first `Money` object
1. `cash` for the second `Money` object
1. `cash` for the result of the addition

A test case would look like:
```python
({100:1, 50:2}, {100:2, 50:4}, {100:3, 50:6})
```

I'll go ahead and create a list of test cases.
Again, note that each of the calculations is done by hand:
```python
test_add_cases = [
    ({}, {}, {}),
    ({100:1}, {50:1}, {100:1, 50:1}),
    ({1:9}, {1:5}, {1:14}),
    ({100:1, 50:2}, {100:2, 50:4}, {100:3, 50:6}),
    (
        {100:1, 50:1, 20:3, 2:8},
        {50:2, 20:2, 10:3, 5:2, 1:9},
        {100:1, 50:3, 20:5, 10:3, 5:2, 2:8, 1:9},
    )
]
```

Finally, we'll parametrize and conduct the test like so:
```python
@pytest.mark.parametrize("cash1, cash2, cash_expected", test_add_cases)
def test_add(cash1, cash2, cash_expected):
    money1 = Money(cash1)
    money2 = Money(cash2)
    added_money = money1 + money2
    expected_money = Money(cash_expected)
    assert added_money.cash == expected_money.cash
```

This is just like what we did for `test_count`.
We prepared a list of test cases; here each test case is a 3-tuple.
We unpacked it into 3 variables: `cash1`, `cash2`, and `cash_expected`

And then we conducted the *Arrange*, *Act*, and *Assert* parts of the test using these variables.

And just like that, we have a test with 5 different test cases.

Let's take a moment to run this.
```
python3 -m pytest
```

It should have collected 12 tests, all passing.

Try to repeat the parametrization exercise for `test_init` by yourself, before you proceed. Just whip up a few more test cases, and fit it into the syntax we discussed earlier. Do it now if you can.

All right, here is what I did:
```python
test_init_cases = [
    ({}, {100:0, 50:0, 20:0, 10:0, 5:0, 2:0, 1:0}),
    ({100:1}, {100:1, 50:0, 20:0, 10:0, 5:0, 2:0, 1:0}),
    ({50:2, 100:1}, {100:1, 50:2, 20:0, 10:0, 5:0, 2:0, 1:0}),
    ({50:2, 10:3, 100:1, 1:8}, {100:1, 50:2, 20:0, 10:3, 5:0, 2:0, 1:8}),
    (
        {20:7, 50:2, 10:3, 100:1, 1:8, 5:4, 2:9},
        {100:1, 50:2, 20:7, 10:3, 5:4, 2:9, 1:8},
    ),
]

@pytest.mark.parametrize("cash, expected", test_init_cases)
def test_init(cash, expected):
    money = Money(cash)
    assert money.cash == expected
```

Confirm that this indeed gets detected as 5 passing tests.

Now, for `test_sub`.
There is a neat thing we can do here.

You see, a test case for subtraction would be very similar to a test case for `add`, because
```
IF
(money1 + money2).cash == money3.cash
THEN
(money3 - money1).cash == money2.cash
```
So, we can re-use the `add` test cases, like this:
```python
@pytest.mark.parametrize("cash1, cash2, cash_total", test_add_cases)
def test_subtract(cash1, cash2, cash_total):
    money_total = Money(cash_total)
    money1 = Money(cash1)
    subtracted_money = money_total - money1
    expected_money = Money(cash2)
    assert subtracted_money.cash == expected_money.cash
```

And, when we try running the tests, we get 20 tests, all green!


## What if the ideal behaviour is to throw an Error?

Having done this, let's take a moment to think if there is something that could go wrong.

One thing is that, if we try to initialize a `Money` object with an invalid denomination or quantity, things could get messed up.
Think of a cash dictionary like `{100: -0.5, 40:3}`

We don't have checks against this yet.
Ideally, the `Money` class should throw a `ValueError` at such an invalid initializer.
Let's fix that in the `Money` class initializer.
To begin, we need to write a failing test that describes this behaviour.

We know how to construct tests for specific equality assertions.
But, how to test when the ideal behavior is to thrown an error?

Pytest has this taken care of.
Let's see how.

We'll be writing a new test for catching invalid initialization, with new test cases.

We'll start by creating test cases of invalid inputs, and a parametrized test:
```python
test_init_invalid_cases = [
    {45: -1},
    {35: 10},
    {100:"Hello"},
    {100: -1},
    {100:10, 50: 0.3},
]
@pytest.mark.parametrize("cash", test_init_invalid_cases)
def test_init(cash):
    # TODO
```

Note that if each test case has just a single item (like we have here), then you don't need to wrap it in a tuple.

Now, we need to tell pytest that `Money(cash)` should raise a `ValueError`.
We do this by:
```python
def test_init_invalid(cash):
    with pytest.raises(ValueError):
        Money(cash)
```

That's it. Using the `with pytest.raises` block, we've told `pytest`, "the code within this block must raise a `ValueError`, else it's a fail."

We'll try running the tests with
```
python3 -m pytest
```

And sure enough we get 5 failing tests.
Each of them says that `DID NOT RAISE <class 'ValueError'>`

Let's get back into the game and make these tests pass.

Going back to `cash_dispenser/money.py`, let's first figure out if the keys in `cash` are present in `DENOMS`

```python
    def __init__(self, cash):
        for key in cash.keys():
            if key not in DENOMS:
                raise ValueError(f'Invalid denomination: {key}')
        self.cash = {d:cash.get(d, 0) for d in DENOMS}
```

And we re-run the failing tests with
```
python3 -m pytest -x --lf
```

And... boom!
We passed the first 2! Not bad!

Which one did we fail this time? `{100: "Hello"}`.
That's because we checked only for invalid keys so far, not for invalid values.
Let's do that now, and allow only values of type `int`.
```python
    def __init__(self, cash):
        for key,value in cash.items():
            if key not in DENOMS:
                raise ValueError(f'Invalid denomination: {key}')
            if type(value)!=int:
                raise ValueError(f'Invalid quantity: {value}')
        self.cash = {d:cash.get(d, 0) for d in DENOMS}
```

Rerun the tests and... Bingo! We passed the one we were stuck on!

Now the one we're failing is `{100: -1}`.
That's because we checked `-1` is an `int`, but an invalid value.
Easy fix for that!
We'll simply add an `or` to the earlier value condition:
```python
    def __init__(self, cash):
        for key,value in cash.items():
            if key not in DENOMS:
                raise ValueError(f'Invalid denomination: {key}')
            if type(value)!=int or value<0:
                raise ValueError(f'Invalid quantity: {value}')
        self.cash = {d:cash.get(d, 0) for d in DENOMS}
```

And, fingers crossed, let's rerun the last failing test.

And there's that.
We passed that one, and all test after that one.

Let's run all tests again, to make sure we haven't broken any existing functionality:
```
python3 -m pytest
```

Nope, all good, 25 passing tests.
That's something to be proud of!

At this point, if you wish, you could go refactoring your newly written code, but I won't do it here.

This was probably a heavy post.
But, putting in some time and learning these techniques will take your testing game to the next level.

Next time, we'll talk about organizing our tests, stay tuned.

---

- [Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post04.md)
- Next Post (coming soon)
- [List of All Posts](https://github.com/CodingComputing/gamified-testing/blob/main/README.md)

