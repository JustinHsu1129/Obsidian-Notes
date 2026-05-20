# Static Arrays

## Packed Arrays
A packed array refers to the dimensions declared before the data identifier name

```
<data type> [data width] name_of_array

bit [7:0] temp_var; //an 8 bit packed array of type bit called temp_var

bit [0:1023] abc; //a 1024 bit vector, packed array of type bit (MSB:LSB)
```

+ packed arrays are continuous in memory-they are held together in blocks without any gaps in between
+ often used for compact data representation

![[Pasted image 20260323134207.png]]
```
<data type> [number of elements][data width of each element] name_of_array

//the first [number of elements] is the number of arrays held within this array, so like array[2], array[1], array[0]

//the second [width of each element] represents how many bits are within each array, so [7:0] means there are 8 bits each within the array, so array[2] is 8 bits long, array[1] is 8 bits long, array[0] is 8 bits long
```

![[Pasted image 20260323134551.png]]

+ we can assign values to index the specific element of the array

`in this example, array[2]=2, array[1]=4, array[0]=12, and can be indexed with these values`

+ we can also directly access specific bits within the entire array

`access array location = array[element][bit within the element`

so for example `y=array[2][1]` means to access the 1st bit (with 0th bit being right-most bit) in the 2nd array, which returns `y=1` when we search for the array in the example

## Unpacked Array
An unpacked array refers to the dimensions declared after the data identifier name
+ unpacked arrays can be any data type
+ unpacked arrays may or may not be represented as a contiguous set of bits

An unpacked array stores everything with column form-you can have multiple sets of data, not just a long string of data as with the packed array:

![[Pasted image 20260323135349.png]]

can declare with:

`int <array name> [6]` or `int <array name> [0:5]`, this array has 6 individual `int` elements, meaning each element is 32 bits wide

We can also have 2d arrays-matrix:

![[Pasted image 20260323135555.png]]

Have more elements within our array-create a matrix, where each cell holds our data of specific type.

Create 2d array with `<data type> <array name> [number of rows] [number of columns]`

To access a specific cell within the array we can use `<array name> [row] [column]`

We can also have 3d arrays-tensor:

![[Pasted image 20260323135906.png]]

Create 3d array with `<data type> <array name> [layer] [row] [column]`

Conceptually, this is like a 3d matrix

# Dynamic Arrays

As the name suggests, dynamic arrays are dynamic in size, they can change in size depending on utilization.

To declare a dynamic array, we can provide empty brackets after the identifier:

`<data type> <array name> []`, the `[]` provides the dynamic array setting, as now it is dependent on the rest of the code.

can assign values to the dynamic array with: `d_array = '{0,1,2,3}`, or can do it loop-wise with:
`foreach (d_array[j]) d_array[j] = j //for each element within d_array, set it to j`

we can also change the length of the dynamic array in 2 ways:

change the length while deleting all information held within the array:

`d_array = new[10]; //set the array to 10 elements, deletes all information previously held within d_array`

or we can change the length while retaining all held information (concatenation):
`d_array = new[10](d_array); //set the array to 10 elements, with the first 4 elements being the data held in the original array, then add on 6 extra empty elements`

![[Pasted image 20260323141453.png]]

# Associative Arrays

An associative array stores entries in a sparse matrix, and storage is only allocated when they are used.

Associative arrays can be indexed with any data type, not just `int`, as in we can access the cells with other data types like `string, bit, etc.`

When the size of the collection is unknown or data space is sparse, an associative array is a better option

`<data type> <name of array> [index_type]`

+ can use `*` to denote any arbitrary indexing type

![[Pasted image 20260323142328.png]]

Ex 1.
for `int num_cars[string]`, it is assigned with `num_cars='{"hyundai":2, "suzuki":1, "tata":3};`, which means there are there are 2 hyundais, 1 suzukis, and 3 tatas

can access the specific elements of the array (2, 1, 3-for this case) with the index values `hyundai, suzuki, tata`

Ex 2.
since the index variables are enumerated, `array_enum` is indexed with those enumerated strings:

`bit [7:0] array_enum[tr_type] //array_enum indexed by tr_type, which is the enumerated list holding the index variables of {TRANS, RECIEVE, REPEATER}`

```
TRANS=enum 0, holds value 10
RECIEVE=enum 1, holds value 20
REPEATER=enum 2, holds value 30
```
