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

# Procedures (`proc`)

Procedures are defined using the `proc` command. Every procedure runs in its own local scope by default, meaning variables instantiated within that `proc` are not accessible outside of that `proc`, and variables instantiated on the outside of that `proc` are not usable within that `proc`.

Defining a proc:
```
proc name {arg1 arg2 ...} {
	# body of proc
}
```

Procedures automatically return the last value computed in the last command of the `proc`, but you can also explicitly use `return $value`

Default parameter values can be defined using nested lists:

```
proc greet {name {prefix "Hello"}} {
    return "$prefix, $name!"
}

puts [greet "Alice"]          ;# Outputs: Hello, Alice!
puts [greet "Bob" "Welcome"]  ;# Outputs: Welcome, Bob!
```

Variable arguments can be collected using `args`:

```
proc sum_all {args} {
    set total 0
    foreach n $args {
        set total [expr {$total + $n}]
    }
    return $total
}

puts [sum_all 1 2 3 4 5] ;# Outputs: 15
```

---
`foreach` is the primary command to iterate over elements in a list:

`foreach var_name list body`

`var_name`: the variable that takes the value of the current element on each iteration
`list`: the list to iterate through
`body`: the script enclosed with `{}` to be executed on each element

```
set clk_list [list "clk_core" "clk_mem" "clk_pcie"]

foreach clk $clk_list {
    puts "Applying clock uncertainty to: $clk"
    # set_clock_uncertainty 0.1 [get_clocks $clk]
}
```

Makes a list called `clk_list` with following entries: `clk_core, clk_mem, clk_pcie`

The `foreach` then iterates over `clk_list` where variable `clk` takes the value of the current element, and the body is executed using that variable `clk` instead, which will then be applied back to the entries in `clk_list`.

Pairs of variables can also be iterated upon:

```
# Pin and its corresponding delay constraint
set pin_delays {
    in_a 1.2
    in_b 0.8
    in_c 1.5
}

foreach {pin delay} $pin_delays {
    puts "Pin: $pin has a max delay of $delay ns"
}
```

In the `pin_delays` list, each entry has a corresponding value, as such in `foreach` we can use 2 `var_names` to represent each entry and its corresponding value from the `pin_delays` list, and operate on both variables at one in the body.

Multiple lists can also be passed in parallel:

```
set modules [list "alu" "regfile" "decoder"]
set areas   [list 1200  4500      320]

foreach mod $modules area $areas {
    puts "Module $mod occupies $area um^2"
}
```

Control commands `break` and `continue` can be used to control execution:

```
set corners [list "ss_0.72v_125c" "tt_0.80v_25c" "ff_0.88v_0c"]

foreach corner $corners {
    if {$corner eq "tt_0.80v_25c"} {
        continue ;# Skip typical corner
    }
    puts "Analyzing extreme corner: $corner"
}
```

---

Variables defined outside of `proc` are global. Inside a `proc`, only local variables can be used:

```
set status "IDLE"

proc print_status {} {
    # Error: can't read "status": no such variable
    puts "Status: $status"
}
```

To use global variables within a `proc`, the commands `global` and `upvar` can be used.

`global` links one or more variable names in the current `proc` to the top level global namespace:

```
set counter 0

proc increment {} {
    global counter
    incr counter
}

increment
increment
puts $counter ;# Outputs: 2
```

+ any changes to `counter` inside `increment` modify the real global value directly
+ `global` only targets the top level scope

`upvar` passes arrays into `proc` because using `global` will typically cause the array to be turned into a list or cause errors:

```
proc print_user_info {arrayName} {
    upvar 1 $arrayName user
    puts "Name: $user(name)"
    puts "Role: $user(role)"
}

set account(name) "Dana"
set account(role) "Admin"

print_user_info account
# Outputs:
# Name: Dana
# Role: Admin
```

# Lists vs Strings

A list is a string with specific whitespace and quoting rules, you should never construct lists with manual string concatenation, always use dedicated list commands:

```
# Bad: vulnerable to spaces or quotes breaking structure
set items "apple orange"
set items "$items banana with space"

# Good: constructs safe elements
set fruits [list "apple" "orange"]
lappend fruits "banana with space"

puts [lindex $fruits 2]   ;# Outputs: banana with space
puts [llength $fruits]    ;# Outputs: 3
```

Key list commands to know:

- `lindex $list $i` — get element at index `$i`.
- `lappend varName elem ...` — append elements in-place to a variable.
- `lsearch $list $pattern` — find matching elements.
- `lsort $list` — sort elements alphabetically, numerically, or by custom key.
- `split $str $delim` and `join $list $delim` — convert between strings and lists.

# Arrays vs Dictionaries

Arrays in Tcl are collections of variables tied to a namespace. They cannot be returned from or passed to a `proc` (must use `upvar` or `array get`):

```
set user(name) "Alice"
set user(role) "Developer"

# Inspecting keys and values
puts [array names user] ;# role name
```

Dicts are true values that can be passed as normal arguments, returned from `proc`, and nested:

```
# Creating and updating a dict
set profile [dict create name "Alice" role "Developer"]
dict set profile location "Atlanta"

# Reading values
puts [dict get $profile name] ;# Alice

# Iterating
dict for {key val} $profile {
    puts "$key -> $val"
}
```

# File System Operations

Tcl has built in collection of commands that handle path resolution and file management:

```
set path [file join "" "var" "log" "app.log"] ;# /var/log/app.log
if {[file exists $path]} {
    puts "Size: [file size $path] bytes"
    puts "Directory: [file dirname $path]"
    puts "Extension: [file extension $path]"
}
```

All I/O, file operations, etc., is managed through channels. When you open a channel, a channel identifier is returned that allows you to read and write:

```
set fp [open "data.txt" r]
# ... read or process ...
close $fp
```

`open` and `close` are used to open and close channels, respectively