# CORDIC Engine
Can use CORDIC as an area efficient way to implement the different ways of generating the many amount of possible frequencies needed for the different protocols.

# Programmability
+ Decide on length of instruction (should probably be 32 bits tbh)
	+ Decide if instruction should have built in fields to specify an already existing protocol
	+ Decide on how to split up instruction fields
		+ What bits should be used to program the CORDIC engine, what fields used for control
			+ Can look at ML quantization formats to program frequency
			+ Can use BF16/fp24/etc. to represent large frequencies (no need for sign bit)
				+ https://en.wikipedia.org/wiki/Bfloat16_floating-point_format
		+ Use multiple instructions to program multiple pins

# Physical Implementation
+ Timing is a very big concern-timing needs to be done on the highest frequency to support the most amount of protocols
+ Physical signal propagation is a concern because of IR drop on pins