A `class` is a user defined data type that includes data, functions, and tasks that operate on data

The data in a `class` are called class properties, while the functions and tasks that operate on the data are called `methods`

Ex.

![[Pasted image 20260323154739.png]]

declare a class with `class <class name>` and end with `endclass`

within the `class`, you can define data with `<data members>`, functions with `function <function name>` and tasks with `task <task name>`

so for the `class` called packet, it uses two pieces of data: `address` that is 32 bits, and `data` that is also 32 bits

packet has a function called `new()`, which runs the `$display` command

## Class declaration/class instance and object creation

![[Pasted image 20260323155127.png]]

`class` can be instantiated within `module` but not vice versa

in the `module`, we can declare `class_1` as the type `sv_class`, which is what it will be known in the code

we need to now create an object and assign its handle to `class_1` with `class_1=new();`
 + this allocates memory so the `class` can actually be used and store the relevant information, and assigns its handle to `class_1` so you can actually call and use `class_1`

## Class constructor

![[Pasted image 20260323155904.png]]

We can define our own function called `new();` within a class to do what we want (ie. initialize certain data, set our own values as default values, etc.)

this `new();` function will be used in the `module` once it is called instead of the default SystemVerilog `new();` which creates an object and assigns it a handle

## Class assignment

![[Pasted image 20260323161325.png]]

In the example, we see that the `function` transaction gets two handles `tr1` and `tr2`, and once we initialize `tr1` with `new();`, now `tr1` and `tr2` are out of sync-if we wanted to use both `tr1` and `tr2` to represent the same instance of a class, we now need to assign `tr2=tr1` so they both represent the same instance of the `transaction` class
+ share the same memory space
## Class inheritance

![[Pasted image 20260323161904.png]]

Create a `child` class with `extends` command: `class <child class> extends <parent class>;`

There are two parts to inheritance: `parent` and `child` classes, a `parent` class is just a normal class.

A `child` class inherits all the properties and methods of the `parent` class it is inheriting from-the `child` class has access to the same data, functions, and tasks as a `parent` class while also adding on new data, function, and tasks
+ the data the `child` class inherits is independent however-the `child` has its own data (that just shares a name with the `parent` data)
+ this is for ease of writing code-you do not have to constantly change a class over and over again to account for each new thing, you can just use inheritance and write a new class that supports the use case while maintaining consistency with the old class

Still need to perform object creation with `new();`

With `c_tr.data=5`, we can now access the data called `data` from the parent class through the child class

## Super keyword

![[Pasted image 20260323233646.png]]

The `super` keyword allows a `child` class to directly access data held in a `parent` class-the `super` keyword allows the child data to directly modify data in the `parent` class

## This keyword

![[Pasted image 20260323234122.png]]

The keyword `this` allows you to pass data from the input/output of a function to the actual data of the class

Ex. `data` is used as an input variable to the function `new` within the class `transaction`, which also has a property called `data`, we can pass the input `data` to the class property `data` by using `this.data=data //this.<property name> = <input/output name>`

## Polymorphism

![[Pasted image 20260323234525.png]]

We can have `parent` and `child` classes be declared with the same functions and tasks. By using the `virtual` keyword in the `parent` class, when that function is called from a `child` class, since the function is defined as `virtual` in the `parent` class, execution priority will be given to the `child` class function
+ the `parent` class function will not be executed