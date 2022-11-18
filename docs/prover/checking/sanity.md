Rule Sanity Checks
==================

```{todo}
Move this to the user guide, move CLI option description here
```

It is possible to make a mistake when writing a spec that makes a rule pass
when it shouldn't.  These buggy rules are dangerous because they may fail to
catch contract bugs that they seem to rule out.

The {ref}`--rule_sanity` option enables some automatic checks that can warn you
about certain classes of mistakes in specifications.  We refer to these mistakes
as "sanity violations".

We recommend running rule sanity checks once you have written a rule that you
think is correct and that passes on your code.  Changes to contract code can
also cause sanity violations (although this is rare), so it is useful to
occasionally recheck sanity as contracts evolve as well.

Reachability checks
-------------------

Consider the following rule:

```cvl
/**
 * Depositing `amount` ETH and then transferring `amount` tokens to
 * `receiver` must cause `receiver`'s balance to be at least `amount`.
 */
rule deposit_transfer_balance {
    env e; uint amount;
    require amount > 0;

    deposit(e, amount);

    address receiver;
    transfer(e, receiver, amount);

    assert balanceOf(receiver) >= amount,
        "deposit followed by transfer must increase recipient's balance";
}
```

Although this seems like a reasonable rule, it will actually pass for almost
any contract implementation, even one that is buggy.  The rule contains a
subtle mistake: it reuses the same environment for the calls to `deposit` and
`transfer`.

To see why this is a problem, note that the `deposit` method will only
successfully execute if `e.msg.value >= amount > 0`.  On the other hand, the
`transfer` method should not be payable, so it will revert whenever the message
value is nonzero.  Therefore we have the following conditions for any
non-reverting path:

 - `amount > 0` from the require statement
 - `e.msg.value >= amount` for `deposit` to succeed
 - `e.msg.value == 0` for `transfer` to succeed

Since it is impossible to satisfy all three of these conditions, every possible
example will cause a revert, which means none of them will even reach the
`assert` statement.


The reachability sanity check detects errors like these.

```{todo}
Finish
```

Assert vacuity
--------------

```{todo}
This section should be expanded.
```

Require redundancy
------------------

```{todo}
This section should be expanded.
```

