# Fighting the Final Boss


Last time, we started tackling the `withdraw` method of the Cash Dispenser.
This is the final and the most challenging method in this project.
In other words, this is the Final Boss of our coding game.

We'd already handled the INSUFFICIENT FUNDS scenario.

Now, let's move on the SUCCESS scenario.

Here, we assume that we have enough cash in change, to tender the requested amount.

Now, how do we do this?

> [!TIP]
> If you're stuck about how to code a problem, try solving the problem on paper manually to get ideas.

Let's say you need to take out exactly 277 bucks from your wallet.

Your wallet has
```
500 note (x2)
100 note (x1)
 50 note (x4)
 20 note (x7)
 10 note (x8)
  5 note (x1)
  2 note (x1)
  1 note (x9)
```

Here's one way:

You think, 
I need to take out 277 bucks more.
Which is the largest note that I can give out?
You can't give out a 500, that wouuld be too much.
The next available denomination I have is 100.
That's less than 277, so I'll give that out.
And, I take out the 100 note.
I don't have 100 notes any more.
So, I need to give 177 bucks more.

Now how do I give out 177 bucks more.
Which is the largest note that I can give out?
That would be a 50, so I take that out.
I am left with 3 notes of 50, and have to pay 127 bucks more.

How do I give out 127?
Again, with a 50. That leaves 77 to pay, and I have 2 notes of 50.

How do I give out 77?
Once more, a 50. That leaves 27 to pay, and I have 1 note of 50 left.

How do I give out 27?
I give out the next available note that's less than 27.
That would be a 20. I give out the 20, leaving me with 6 20-notes.
I still have to pay 7 more bucks.

How do I give out 7?
Again I check which is the biggest note less than 7 I have.
That would be a 5. I give out my only 5-note.
I still need to pay 2 bucks.

I pay it off with a single note of 2.

So, in the net, I gave out 277 bucks by:
```
100 note (x1)
 50 note (x3)
 20 note (x1)
 10 note (x0)
  5 note (x1)
  2 note (x1)
  1 note (x0)
```

What is the learning from this exercise?

It is that, to pay any amount x, I need to find the denomination that I have, which is less than x.

I keep doing that until I have given out the entire amount I need to.

If I have the required change, it is guaranteed that we find the solution.

Now let's start filling in the code.

First, we'll create an empty dictionary representing the cash to be dispensed.

Then we'll keep track of the remaining amount to be dispensed.

At each step, we'll determine the note to hand out, as discussed before.

And we'll continue till the entire amount has been dispensed.

(There is a better way to do this, but let's keep it very simple in the beginning. We'll improve later.)

The withdraw method looks like this after this step:
```python
    def withdraw(self, amt):
        # Handle the INSUFFICIENT FUNDS scenario
        total_available_money = self.money.count()
        if total_available_money < amt:
            return {
                'status': "INSUFFICIENT FUNDS",
                'message': f"Only {total_available_money} available"
            }
        # NEW PART: Handling the Success scenario
        cash_to_give = {}
        remaining_amt = amt
        while remaining_amt>0:
            note_to_give = max(
                note
                for note,qty in self.money.cash.items()
                if qty>0 and note<=remaining_amt
            )
            if note_to_give in cash_to_give:
                cash_to_give[note_to_give] += 1
            else:
                cash_to_give[note_to_give] = 1
            remaining_amt -= note_to_give
            self.money.cash[note_to_give] -= 1
        return {"status": "SUCCESS", "cash":cash_to_give}
        # TODO: Inadequate Change scenario
```

And we'll run the tests.

That does pass the 3 test for the Success scenario.

Now, we only have the Inadequate Change scenario to deal with.

The first question we need to ask is: How do we determine whether there is adequate change or not?

Let's think back to how we would do this manually.

When do we realize that we don't have proper change to pay an amount?

The thing is, we don't know in advance.

Once we start collecting the cash to pay out, then at some point we get to a situation when:
1. Remaining amount to be given > 0
2. Remaining amount to be given < All available denomination

Keep in mind that this has to be checked in the dispensing loop.
The first condition here is being checked in the `while` loop itself, so that's taken care of.

So, we just need to check the second condition.
We can simply make a list of availabe denominations, that are less than the remaning amount.
If that list happens to be empty, we have inadequate change.

While we are at it, can we figure out what is the lower dispensable amount?

It's simply the amount of `cash_to_give`!

So, let's go ahead:
```python
# In cash_dispenser/cash_dispenser.py, CashDispenser class, withdraw method
    def withdraw(self, amt):
        # Handle the INSUFFICIENT FUNDS scenario
        total_available_money = self.money.count()
        if total_available_money < amt:
            return {
                'status': "INSUFFICIENT FUNDS",
                'message': f"Only {total_available_money} available"
            }
        cash_to_give = {}
        remaining_amt = amt
        while remaining_amt>0:
            # Detect whether change adequate
            available_denoms_less_than_remaining_amt = [
                note for note,qty in self.money.cash.items()
                if qty>0 and note<=remaining_amt
            ]
			is_change_inadequate = available_denoms_less_than_remaining_amt == []
            if is_change_inadequate:
			    # Restore `money` that was taken out in the earlier calculation
				self.money = self.money + Money(cash_to_give)
                lower_dispensable_value = Money(cash_to_give).count()
                return {
                    "status": "INADEQUATE CHANGE",
                    "message": f"Can dispense {lower_dispensable_value}"
                }
            note_to_give = max(
                note
                for note,qty in self.money.cash.items()
                if qty>0 and note<=remaining_amt
            )
            if note_to_give in cash_to_give:
                cash_to_give[note_to_give] += 1
            else:
                cash_to_give[note_to_give] = 1
            remaining_amt -= note_to_give
            self.money.cash[note_to_give] -= 1
        return {"status": "SUCCESS", "cash":cash_to_give}
```

And it passes the tests of `test_cash_dispenser`!

Just to make sure, we'll run the entire test suite:
```
python3 -m pytest
```

And sure enough, the code passes all the 43 total tests.

So, we have defeated the "final boss", the final scenario of the final method in our component.

But as you would notice, there is some duplication in the code.

We have accomplished the functionality, but we still haven't done it with finesse.

Next time we'll fix it up and improve the code.

---

- [Previous Post](https://github.com/CodingComputing/gamified-testing/blob/main/post06.md)
- Next Post (Coming Soon)
- [List of All Posts](https://github.com/CodingComputing/gamified-testing/blob/main/README.md)

