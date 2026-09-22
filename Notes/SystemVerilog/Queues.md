![[Pasted image 20260323152447.png]]
A queue operates on the same principle as a FIFO. 2 types of queues: unbounded and bounded

+ unbounded queue-unlimited depth to the queue
+ bounded queue-set depth to the queue

A queue is constructed by `<data type> <queue name> [$] //the $ within the brackets is what makes it a queue`

A bounded queue can be constructed with `[$:n] //where n is the desired depth, giving you n+1 depth to the queue`

`push` allows you to put data into the queue

`pop` allows you to pull data from the queue

Ex. 

![[Pasted image 20260323152825.png]]

declare the queue with `int q[$] //creates a queue called q that stores ints, with infinite depth`

add elements to the back of the queue with `q.push_back()`

remove elements from the front of the queue with `q.pop_front` or from the back with `q.pop_back`