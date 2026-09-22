### Assigning variables

item = 'Banana'

Item = 'Apple'

(python is case sensitive-item =/= Item)

### Data types

integer = any number
string = 'text' (must have the single quotations for string)
boolean = True/False
list =['Rahh', 'Deez', 'nuts'] (data can be changed)
N-tuple = (1.5,2.5,3.5) (cannot change once created)
set = {1,2,3,4,4,5} (cannot contain duplicates-will remove any duplicates)
dictionary = {'Bob': 1. 'Mark': 2} (each element contains a key and value which are associated with each other)

to concatenate different datatypes together they must be converted into the same type (eg. an integer must be converted to string to concatenate a string and an integer together)

### Math

addition: a+b
subtraction: a-b
multiplication: a*(b)
division: a/b
power: a**(b)

### Logic

if something:

elif something:

else:

for i in range(3): (if i < 3)

while i < something:

while True: (infinite loop)
break: cancel the loop

### Functions

def function_name(input_variable):
	function function
	return (to get the final answer out of the function, otherwise don't need this if only code is being executed)

def function_name(input_variable = 'default value'):
### Try and Accept Block

try:
	some function
except:
	some other function

It will try to execute whatever is in the try function (like getting a specific input) and if it does not work it will execute the except function

### F-Strings

print(f'Text: {variable}, Text: {variable2}')

F stands for format, makes it much more convenient to type out complex strings

### Import

import module_name

import math as m (lets us shortcut the imported module names as just m, so in this case math.function() would turn into m.function())

### Classes

```
class ClassName (eg. car):
	def __init__(self, [object definition variable eg. color, power]):
		self.color = color
		self.power = power
		etc.
		
Volvo: car = car('red', 200)
		
```

This creates an object of class `car` called Volvo, and we can now refer to the different definitions that create a Volvo (eg. the color, power, etc.)

### Methods

A method is a function within a class

```
class ClassName (eg. car):
	def __init__(self, [object definition variable eg. color, power]):
		self.color = color
		self.power = power
		etc

	def drive(self)
		print(f'{self.brand} is driving')
		
Volvo: car = car('red', 200
Volvo.drive()
		
```


