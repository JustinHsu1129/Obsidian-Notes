![[Pasted image 20260323131814.png]]

Use `bit` for more flexibility in width

Use `byte` when you specifically need an 8 bit signed number

2's complement: invert then add 1

# Enumeration

A set of constants is called an `enum` (enumerated) type
* defines a set of named values having an anonymous `int` type

```
the [enum] keyword is used to define an enumerated list type without an alias

enum  {red, yellow, green} traffic_signal;
//int type: red=0, yellow=1, green=2
```

```
define an [enum] with a specific underlying data type using the [enum] keyword

enum logic[0:1] {red, yellow, green} color;
//the colors are defined with binary logic values: red=00, yellow=01, green=10

***remember to have enough width to represent all variables***
```

```
you can set custom values within the [enum] list, and any subsequent variable in the list will have an index incremented by 1

enum {red, yellow=10, green} color;
//int type: red=0, yellow=10, green=11
```

+ you can use user defined types through the keyword `typedef`

```
[typedef] allows you to create an alias for an enumerated type, beneficial for code clarity and resuability

typedef enum {red, yellow, green} color_t;
color_t my_color;

//the set stored in color_t is now accessible in my_color
```

![[Pasted image 20260323133118.png]]

In this example, the enumeration list is `IDLE, RUNNING, STOPPED`, with `IDLE=0, RUNNING=1, STOPPED=2`, and it is defined as `current_state`, which can be used throughout the code. Since the `state_t` list is enumerated and defined as `current_state`, that means you can either use `0, 1, 2` or use the values held within the `state_t` list to index `current_state` (directly using `IDLE, RUNNING, STOPPED`), as is in this example in the `case` statement.