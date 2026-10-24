In TCL, everything follows the following format: `command arg1 arg2 ...` separated by spaces

Setting a variable: `set var_name value`
Reading a variable: `$var_name`
Arithmetic requires the `expr` command

```
set clk_period 2.5
set clk_frequency [expr{1000 / $clk_period}]
```

Double quotes (`"..."`) group everything together and are evaluated with variables, substitutions, command evaluations, etc.
Curly braces `{}` group everything together and are evaluated literally

```
set item "book"

puts "I am reading a $item"
# outputs: I am reading a book

puts {I am reading a $item}
# outputs: I am reading a $item

puts "2 + 2 = [expr{2+2}]"
# outputs: 2 + 2 = 4

puts {2 + 2 = [expr{2+2}]}
# outputs: 2 + 2 = [expr{2+2}]
```

`\n` is evaluated as a newline with `""` and literally `\n` with `{}`

`{}` is best used for control structures like `if, while, proc, expr` because these blocks expect expressions to be evaluated later:

```
set i 0

# BROKEN: Using double quotes
while "$i < 3" {
    puts "Iteration: $i"
    incr i
}
```

Since `""` are used here, that means `$i < 3` is immediately evaluated. Since `i = 0` right now, that means the condition for `while` is `0 < 3`, which is always true. This results in an infinite loop where `i` will be infinitely incremented.

Instead we need to use `{}` here:

```
while {$i < 3} {
    puts "Iteration: $i"
    incr i
}
```

Since `{}` evaluates the expression literally, that means the `while` loop will re-evaluate the expression `$i < 3` on each loop, so `i` will eventually stop incrementing at `i = 3`.

`""` is best used for assembling formatted strings:

```
set filename "output"
puts "Log written to: /tmp/$filename.log"
```

