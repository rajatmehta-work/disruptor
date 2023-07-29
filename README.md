## Description
[LMAX-Disruptor](https://lmax-exchange.github.io/disruptor) is a ulta low latency inter thread communication as an alternative to bounded queueu, which makes use of padding to avoid memory false sharing, pre assign things to be cache friendly. 
This is the implemention of LMAX-Disruptor in Golang. 

It send data to consumers/threads at constant rate, it does this using below primary techniq:
1. It avoids locks/mutex at OS level.
2. It produces no garbage at runtime, by preallocating the ring buffer/array, By avoiding garbage, the need for a garbage collection and the stop-the-world application pauses  introduced can be almost entirely avoided.
3. Cache line padding

and many more.


## Features
On a MacBook Pro (Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz) using Go 1.20.5, it can pass many hundreds of millions of messages per second (yes, its true) from one goroutine to another goroutine, which is far fastest then go channels.

In Go, the current implementation of a channel (`chan`) maintains a lock around send, receive, and `len` operations and it maxes out around 25 million messages per second for uncontended access&mdash;more than an orders of magnitude slower when compared to the Disruptor.  The same channel, when contended between OS threads only pushes about 7 million messages per second.

## Prerequisites
1. Install latest go lang binary (1.18.3 and above) 
2. Install latest vs code
## Perfomance
Running all unit test cases in benchmarks
```Shell
    go test -v -bench=Benchmark -benchtime=1000000000x -count 3 ./benchmarks
```
Benchmarks
----------------------------
Each of the following benchmark tests sends an incrementing sequence message from one goroutine to another. The receiving goroutine asserts that the message is received is the expected incrementing sequence value. Any failures cause a panic.

* CPU: `Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz`
* Go Runtime: `Go 1.20.5`

Scenario | Per Operation Time | Bytes per Operation | Allocation per operation
-------- | ------------------ | ------------------- | ------------------------
Disruptor: Reserve One & One Consumer | 13.11 ns/op | 0 B/op | 0 allocs/op
Disruptor: Reserve Many & One Consumer | 1.567 ns/op | 0 B/op | 0 allocs/op
Disruptor: Reserve One Multiple Consumer | 13.98 ns/op | 0 B/op | 0 allocs/op
Disruptor: Reserve Many & Multiple Consumer | 1.615 ns/op | 0 B/op | 0 allocs/op

No new memory/bytes was assigned in each operation, this is the power of preallocated ring buffer, and considering cache line and Mechanical Sympathy.

## Example
Example is implemented in example/main.go