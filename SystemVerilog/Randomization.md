SystemVerilog has a built in randomize function `randomize()` as well as randomize keywords to provide a random input as a stimulus for our testbenches

`rand` is the keyword for randomizing an object-provides uniformly distributed random variables which can repeat

`randc` is the keyword for randomizing an object-but it ensures that all values within its range are randomly generated before any value is repeat

Ex. if `data` is an 8 bit variable:

`randc bit [3:0] data //data will have randomly generated values for 0-15 before any order can repeat again`

![[Pasted image 20260324191618.png]]

## Constraints in randomization

`constrain` blocks represent the constraints for random variables:

`constrain <constraint name> {expression/condition}`

Ex.

```
constrain range_constraint { value >= 10; value <= 50;}
constrain set_constraint {value inside{10, 20, 30, 40, 50};}
constraint relational_constraint {value1 > value2;}
```

![[Pasted image 20260324192231.png]]

## Inline constraints

Might need to modify constraints during randomization, so we can use inline constraints

Inline constraints should not conflict with constraints written for the class so as to avoid randomization failure

```
item.randomize with {val1 > 150; val1 <160;};
item.randomize with {val2 inside {[10:15};};
```

## Soft constraint

![[Pasted image 20260324194800.png]]

Allows for adjustment of inline constraints so the inline constraint and the general class constraint do not conflict and cause randomization failures
+ any constraint with the `soft` keyword will be discarded, so in the example `soft val inside {5, [10:15];}` will be discarded and the inline constraint `value inside {[20:30];}` will be used instead

## Implication constraints

![[Screenshot 2026-03-24 at 19.54.36.png]]

Implication can be used to declare a conditional relation between two variables

`expression -> constraint`

Ex. the `addr_range` is a string and set to `small`, with the condition (the implication) that `addr < 8`

can also directly use `if else` for this as well:

![[Pasted image 20260324201032.png]]

`addr_range` is small if `addr < 8` otherwise `addr_range` is not small if `addr > 8`

## Array constraints

![[Pasted image 20260324201405.png]]

we can constrain all the entries to an array, called iterative constraints
+ the constraints to every foreach loop are called iterative constraints

## Enabling and disabling constraints

We can enable and disable constraints
+ by default all constraints will be enabled

`constraint_mode(1)` **enables** the constraint block

`constraint_mode(0)` **disables** the constraint block

+ the default value to `constraint_mode()` is 1
+ once the block is disabled, in order to re-enable `constraint_mode(1)` must be used

Ex.

![[Pasted image 20260324211716.png]]

in the example, after `obj.c_x.constraint_mode(0)` disables the constraint on `x`, in order to re-enable the constraint on `x` the command `obj.c_x.constraint_mode(1)` must be used

Ex. 

![[Pasted image 20260324212039.png]]

in the class `my_class`, variables `x` and `y` are both being randomized, but what if we wanted to disable randomization for only `x`, then we can use `obj.x.rand_mode(0)` to disable randomization on only `x`, and to re-enable randomization we can use `obj.x.rand_mode(1)` to re-enable randomization

## Static constraints

![[Pasted image 20260324212415.png]]

if a constraint is labeled as `static`, this means if the constraint applies to multiple variables then if the constraint is disabled on one of the variables it is disabled on all of the variables

## Functions in constraints

![[Pasted image 20260324212658.png]]

a function written outside of the constraint block can be called within a constraint and be used as the constraint

## Unique constraint

![[Pasted image 20260324212913.png]]

the random generation of the random variables `var_1`, `var_2` and `var_3` (from the example) will be different values due to the `unique` modifier

## Mailbox

![[Screenshot 2026-03-24 at 21.31.01.png]]

a mailbox is a way to communicate between different processes/threads and is commonly used in testbenches to transfer data in a synchronization-safe way

ordering of the mailbox follows FIFO principle

## Semaphore

![[Pasted image 20260324213529.png]]

semaphore acts like a key based lock-preventing race conditions when multiple processes/threads are executing at once
+ only once execution finishes and a key is sent to the semaphore is access unlocked for another process/thread