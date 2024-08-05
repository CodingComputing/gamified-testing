# Setting Up Missions for Cash Dispenser

Now let's get started with constructing tests for the `CashDispenser` class.

The responsibilities were:

> 1. Keep track of how much cash (and in what denominations) is present in the ATM
> 1. Accept deposited cash
> 1. When an amount is requested, try to dispense that amount with the available notes, preferably with a small number of notes. If successful, return a message to the Driver about the Success, and which notes to dispense.
> 1. If the amount cannot be dispensed, then return a message to the Driver about the Failure, with any other helpful information

From the requirements, it looks like withdrawals are slightly involved operations.
So, let's tackle the easy ones first: Keeping track of cash, and Deposits.

To keep track of cash, our `CashDispenser` needs to have an attribute that stores information about the cash present in the machine.
The right time to include this attribute is at the time of initialization of the `CashDispenser` object.
Thus, the test of "keeping track of money" would be equivalent to testing the initialization.

When we deposit some cash, the machine needs to update the information about the available cash.
That's part of the "keeping track of money".

Again, the drill is the same as earlier.
Just imagine that the `CashDispenser` component would work perfectly, and create test cases that showcase this behaviour.

Here I've whipped up a quick couple of tests, put in `tests/test_cash_dispenser.py`:
```python
from cash_dispenser.cash_dispenser import CashDispenser
from cash_dispenser.money import Money
import pytest

test_init_cases = [
    {},
    {100:1, 10:2},
    {100:1, 50:2, 20:3}
]
@pytest.mark.parametrize("initial_cash", test_init_cases)
def test_init(initial_cash):
    cd = CashDispenser(initial_cash)
    assert cd.money.cash == Money(initial_cash).cash

test_deposit_cases = [
    ({100:1}, {50:2}, {100:1, 50:2}),
    ({100:1, 50:2}, {50:1}, {100:1, 50:3}),
    ({100:1, 50:2, 20:3}, {20:1, 10:4}, {100:1, 50:2, 20:4, 10:4})
]
@pytest.mark.parametrize("initial_cash, deposit_cash, total_cash", test_deposit_cases)
def test_deposit(initial_cash, deposit_cash, total_cash):
    cd = CashDispenser(initial_cash)
    cd.deposit(deposit_cash)
    assert cd.money.cash == Money(expected_cash).cash
```

Let's focus on passing these before moving on to `withdraw`.

We now should run only the cases in `tests/test_cash_dispenser.py`.
There's no point running the `Money` tests again at the moment, because we aren't tweaking that class at all.

So, we run the tests with
```
python3 -m pytest tests/test_cash_dispenser.py
```

And sure enough, we get an `ImportError`, because the file we're trying to import doesn't yet exist.

Let's get to the game of fixing things one by one.

First, we'll create a file `cash_dispnenser/cash_dispnenser.py`, and a class `CashDispenser` within it:
```python
class CashDispenser:
    pass
```

Now the test re-run shows that 6 tests were collected and Failed.
That makes sense.

Now that the tests are being collected, it's better use the `-x --lf` options to make sure that only the failing tests are being re-run each time.

Let's try to pass the first test, `test_init`, by writing the following code in `cash_dispnenser/cash_dispnenser.py`:
```python
from cash_dispenser.money import Money

class CashDispenser:
    def __init__(self, initial_cash):
        self.money = Money(initial_cash)
```

Re-run the failing tests
```
python3 -m pytest tests/tests_cash_dispenser.py -x --lf
```

And there we go, the `test_init` test have passed.
So we can go on to working on `test_deposit`.
The reason for it's failure, as given in the `pytest` output, is that `CashDispenser` has no attribute `deposit`.
To fix, we need to define the `deposit` method, by adding these 3 lines to the `CashDispenser` class:
```python
    def deposit(self, deposit_cash):
        deposit_money = Money(deposit_cash)
        self.money = self.money + deposit_money
```

Re-run the failing tests, and they all pass.

Let's take a moment to think why this went so smoothly.

That's because we worked hard on the `Money` class, which forms the backbone of the calculations here.

Now let's go over to the `withdraw` method.

The purpose of this method is to try and dispense cash for a given amount.

There are 3 scenarios to consider:

1. Successful dispensing: The machine has enough cash money and change to give the requested amount.
	Eg. the machine has the following cash `{100:1, 50:2, 20:3, 10:5}`, the Driver requests and amount of say `80`.
	The machine can provide the requested amount with the following cash: `{50:1, 20:1, 10:1}`, which totals to `80`.
	In this scenario, the method should return a dictionary with 2 keys:
	```python
	{
		'status': "SUCCESS",
		'cash': {50:1, 20:1, 10:1}
	}
	```
	Note that we'd prefer to dispense the money with the minimum possible number of notes.
	Eg, we dispense `{50:1, 20:1, 10:1}` instead of `{50:1, 10:3}`.
	**Important: In this scenario, we need to update the money information, to reflect that the above cash has been dispensed.**

2. Insufficient Funds: In this scenario, the amount requested is larger than the total amount of money present in the machine.
	Eg. The machine has `{100:1, 50:1, 20:2, 5:1}`, totalling 195.
	If the Driver requests an amount of 196 or higher, obviously, the machine cannot dispense it.
	In such a scenario, we want the method to return a dictionary with a status and message, like so:
	```python
	{
		'status': "INSUFFICIENT FUNDS",
		'message': "Only 195 available"
	}
	```
	As you can see, we're giving back a helpful message that the Driver can display to the User.

3. Inadequate Change: In this scenario, the amount requested isn't larger than the total amount of money present in the machine.
	However, the cash available cannot be used to tender the exact amount requested.
	Eg, The machine has `{100:2, 50:1}`, ie, a total of `250`, but the Driver requests an amount of `190`.
	`190 <= 250`, so, the funds are sufficient, however, there is no way the machine can provide the exact amount.
	In such a scenario, we want the method to return a dictionary with a status and message, like so:
	```python
	{
		'status': "INADEQUATE CHANGE",
		'message': "Can dispense 150"
	}
	```
	The message tells the closest tenderable amount *below* the requested amount.

Note that Insufficient Funds and Inadequate Change are essentially failed transactions, so, no cash is dispensed, and thus no need to update the money information.

As you can see, the implementation of `withdraw` is the most complex part of this project, considering the multiple scenarios involved.
Also, the task of figuring out how to dispense a given amount with the available cash is non-trivial, and will require some algorithmic thinking.

That said, between the 3 scenarios above, the Insufficient Funds case seems fairly straightforward (we just need to compare the amounts).
After that, arguably the Successful Dispensing is easier, and then Insufficient Change the trickiest.

Let's construct some tests for the 3 scenarios.
Since these will be 3 related tests, it would be good to put them in a class, `TestWithdraw`.
It makes sense to list them in order of ease, so we go from Easy to Difficult missions.

Here are the tests I went with (feel free to devise your own).
It's a biggish block of code, but the bulk of it is test-case tuples.
Logic-wise, it is just 3 test functions that check for 3 scenarios of `test_withdraw`:
```python
class TestWithdraw:
    test_withdraw_insufficient_funds_cases = [
        # testcases are tuples of 3
        # (available cash, available amount, requested amount)
        ({}, 0, 1),
        ({50:1}, 50, 100),
        ({100:1, 50:2, 20:5, 10:10, 5:20, 2:50, 1:100}, 700, 701)
    ]
    @pytest.mark.parametrize(
        "available_cash, available_amt, requested_amt",
        test_withdraw_insufficient_funds_cases
    )
    def test_withdraw_insufficient_funds(
        self, available_cash, available_amt, requested_amt
        ):
        cd = CashDispenser(available_cash)
        withdraw_result = cd.withdraw(requested_amt)
        assert withdraw_result['status'] == "INSUFFICIENT FUNDS"
        assert withdraw_result['message'] == f"Only {available_amt} available"


    test_withdraw_success_cases = [
        # testcases are tuples of 4
        # (available cash, requested amount, expected dispensed cash, balance cash)
        (
            {100:1, 50:2, 20:3, 10:4, 5:5, 2:6, 1:7},
            10, {10:1},
            {100:1, 50:2, 20:3, 10:3, 5:5, 2:6, 1:7}
        ),
        (
            {100:1, 50:2, 20:0, 10:4, 5:5, 2:6, 1:7},
            20, {10:2},
            {100:1, 50:2, 20:0, 10:2, 5:5, 2:6, 1:7},
        ),
        (
            {100:1, 50:2, 20:3, 10:4, 5:5, 2:6, 1:7},
            100, {100:1},
            {50:2, 20:3, 10:4, 5:5, 2:6, 1:7},
        ),
        (
            {100:1, 50:2, 20:3, 10:4, 5:5, 2:6, 1:7},
            150, {100:1, 50:1},
            {50:1, 20:3, 10:4, 5:5, 2:6, 1:7},
        ),
        (
            {100:1, 50:2, 20:3, 10:4, 5:5, 2:6, 1:7},
            170, {100:1, 50:1, 20:1},
            {50:1, 20:2, 10:4, 5:5, 2:6, 1:7},
        ),
        (
            {100:1, 50:2, 20:1, 10:4, 5:5, 2:6, 1:7},
            198, {100:1, 50:1, 20:1, 10:2, 5:1, 2:1, 1:1},
            {50:1, 10:2, 5:4, 2:5, 1:6},
        ),
    ]
    @pytest.mark.parametrize(
        "available_cash, requested_amt, exp_dispensed_cash, balance_cash",
        test_withdraw_success_cases
    )
    def test_withdraw_success(
        self, available_cash, requested_amt, exp_dispensed_cash, balance_cash
    ):
        cd = CashDispenser(available_cash)
        withdraw_result = cd.withdraw(requested_amt)
        assert withdraw_result['status'] == "SUCCESS"
        assert withdraw_result['cash'] == exp_dispensed_cash
        assert cd.money.cash == Money(balance_cash).cash

    test_withdraw_inadequate_change_cases = [
        # testcases are tuples of 3
        # (available cash, requested amount, lower tenderable amount)
        ({100:1}, 10, 0),
        ({100:1, 50:1}, 70, 50),
        ({100:1, 50:1, 10:1}, 70, 60),
    ]
    @pytest.mark.parametrize(
        "available_cash, req_amt, lower_possible_amt",
        test_withdraw_inadequate_change_cases
    )
    def test_withdraw_inadequate_change(
        self, available_cash, req_amt, lower_possible_amt
    ):
        cd = CashDispenser(available_cash)
        withdraw_result = cd.withdraw(req_amt)
        assert withdraw_result['status'] == "INADEQUATE CHANGE"
        assert withdraw_result['message'] == f"Can dispense {lower_possible_amt}"
```

Let's try running the tests:
```
python3 -m pytest tests/test_cash_dispenser.py
```

12 failing tests, no less.

Let's roll our sleeves and play the game of passing these one by one.
We'll open up `cash_dispenser/cash_dispenser.py`.

First we'll deal with the Insufficient Funds scenario.

This is actually easy.
We need to count the `CashDispenser`'s `money`, by running the `count` method on it.
That gives the total amount of money available.
We simply compare it to the requested amount, and return if funds are insufficient.

```python
# In file `cash_dispenser/cash_dispenser.py`, inside the `CashDispenser` class
    def withdraw(self, amt):
        # Handle the INSUFFICIENT FUNDS scenario
        total_available_money = self.money.count()
        if total_available_money < amt:
            return {
                'status': "INSUFFICIENT FUNDS",
                'message': f"Only {total_available_money} available"
            }
        # TODO: SUCCESS and INADEQUATE CHANGE scenarios
```

That should do it.

We run the tests, and Lo, the 3 tests for Insufficient Funds pass already!

That leaves us to implement Success and Inadequate Change.

This is going to be a bit more challenging.

It should be, after all, these are the "final bosses" in the game of this project.

Not to worry.
We'll tackle them in the next part.
Stay tuned!


---

- [Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post06.md)
- [Next Post](https://github.com/CodingComputing/gamified-testing/blob/main/post08.md)
- [List of All Posts](https://github.com/CodingComputing/gamified-testing/blob/main/README.md)
