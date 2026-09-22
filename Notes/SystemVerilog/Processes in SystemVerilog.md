# Fork-join

![[Pasted image 20260323153216.png]]

Within a `fork join` all processes will execute in parallel, `fork join` ends only after all processes within the construct are done executing-wait for everything within the `fork join` to finish before continuing to a different part of the code

start the construct with `fork`, and end with `join`, with individual processes within using `begin` and `end`

# Fork-join_any

![[Pasted image 20260323153534.png]]

Every process starts in parallel, but waits for any process within the `fork join_any` to finish, then jumps to the outside (even if there are other processes within the `fork join_any` still executing)

# Fork join_none

![[Pasted image 20260323154107.png]]

Every process starts in parallel, but will not wait for any processes to be complete and jumps ahead immediately

+ mainly used only for starting a process simultaneously

## Difference between `begin end` and `fork join`

`begin end` executes sequentially-each subsequent process is dependent on the completion time of the previous process

`fork join` executes processes in parallel, but their completion time is dependent on the individual processes themselves

