# Inside the CU

Key ISA difference is scalar + vector
+ In AMD GPUs scalar work gets its own processor and reg file
	+ Scalar unit job: control flow and addr generation
	+ 4 scalar inst maps to only 4 registers instead of 4 * 32 registers in NVIDIA
	+ SGPRs can be also used for handling branching and data dependencies
		+ Divergence is not branching-it is masking
+ Compiler works with the microarchitecture
	+ Schedules independent work between dependent inst