# rust-priority-queue

This crate implements a Priority Queue with a function to change the priority of an object.
Priority and items are stored in an `IndexMap` and the queue is implemented as a Heap of indexes.


Please read the [API documentation here](https://docs.rs/priority-queue/)

## Usage

To use this crate, simply add the following string to your `Cargo.toml`:
```
priority-queue = "1.3.2"
```

Version numbers follow the [semver](https://semver.org/) convention.

Then use the data structure inside your Rust source code as in the following Example.

Remember that, if you need serde support, you should compile using `--features serde`.

## Example

```rust
extern crate priority_queue; // not necessary in Rust edition 2018

use priority_queue::rust-priority-queue;

fn main() {
    let mut pq = PriorityQueue::new();

    assert!(pq.is_empty());
    pq.push("Apples", 5);
    pq.push("Bananas", 8);
    pq.push("Strawberries", 23);

    assert_eq!(pq.peek(), Some((&"Strawberries", &23)));

    for (item, _) in pq.into_sorted_iter() {
        println!("{}", item);
    }
}
```

Note: in recent versions of Rust (edition 2018) the `extern crate priority_queue` is not necessary anymore!

## Speeding up

You can use custom BuildHasher for the underlying IndexMap and therefore achieve better performance.
For example you can create the queue with the speedy [FxHash](https://github.com/Amanieu/hashbrown) hasher:

```rust
use hashbrown::hash_map::DefaultHashBuilder;

let mut pq = PriorityQueue::<_, _, DefaultHashBuilder>::with_default_hasher();
```

Attention: FxHash does not offer any protection for dos attacks. This means that some pathological inputs can make the operations on the hashmap O(n^2). Use the standard hasher if you cannot control the inputs.

## Benchmarks

Some benchmarks have been run to compare the performances of this priority queue to the standard BinaryHeap, also using the FxHash hasher.
On a Ryzen 9 3900X, the benchmarks produced the following results:
```
test benchmarks::priority_change_on_large_double_queue     ... bench:          25 ns/iter (+/- 1)
test benchmarks::priority_change_on_large_double_queue_fx  ... bench:          21 ns/iter (+/- 1)
test benchmarks::priority_change_on_large_queue            ... bench:          15 ns/iter (+/- 0)
test benchmarks::priority_change_on_large_queue_fx         ... bench:          11 ns/iter (+/- 0)
test benchmarks::priority_change_on_large_queue_std        ... bench:     190,345 ns/iter (+/- 4,976)
test benchmarks::priority_change_on_small_double_queue     ... bench:          26 ns/iter (+/- 0)
test benchmarks::priority_change_on_small_double_queue_fx  ... bench:          20 ns/iter (+/- 0)
test benchmarks::priority_change_on_small_queue            ... bench:          15 ns/iter (+/- 0)
test benchmarks::priority_change_on_small_queue_fx         ... bench:          10 ns/iter (+/- 0)
test benchmarks::priority_change_on_small_queue_std        ... bench:       1,694 ns/iter (+/- 21)
test benchmarks::push_and_pop                              ... bench:          31 ns/iter (+/- 0)
test benchmarks::push_and_pop_double                       ... bench:          31 ns/iter (+/- 0)
test benchmarks::push_and_pop_double_fx                    ... bench:          24 ns/iter (+/- 1)
test benchmarks::push_and_pop_fx                           ... bench:          26 ns/iter (+/- 0)
test benchmarks::push_and_pop_min_on_large_double_queue    ... bench:         101 ns/iter (+/- 2)
test benchmarks::push_and_pop_min_on_large_double_queue_fx ... bench:          98 ns/iter (+/- 0)
test benchmarks::push_and_pop_on_large_double_queue        ... bench:         107 ns/iter (+/- 2)
test benchmarks::push_and_pop_on_large_double_queue_fx     ... bench:         106 ns/iter (+/- 2)
test benchmarks::push_and_pop_on_large_queue               ... bench:          84 ns/iter (+/- 1)
test benchmarks::push_and_pop_on_large_queue_fx            ... bench:          78 ns/iter (+/- 2)
test benchmarks::push_and_pop_on_large_queue_std           ... bench:          71 ns/iter (+/- 1)
test benchmarks::push_and_pop_std                          ... bench:           4 ns/iter (+/- 0)
```

The priority change on the standard queue was obtained with the following:

```rust
pq = pq.drain().map(|Entry(i, p)| {
    if i == 50_000 {
        Entry(i, p/2)
    } else {
        Entry(i, p)
    }
}).collect()
```

The interpretation of the benchmarks is that the data structures provided by this crate is generally slightly slower than the standard Binary Heap.

On small queues (<10000 elements), the change_priority function, obtained on the standard Binary Heap with the code above, is way slower than the one provided by `PriorityQueue` and `DoublePriorityQueue`.
With the queue becoming bigger, the operation takes almost the same amount of time on `PriorityQueue` and `DoublePriorityQueue`, while it takes more and more time for the standard queue.

It also emerges that the ability to arbitrarily pop the minimum or maximum element comes with a cost, that is visible in all the operations on `DoublePriorityQueue`, that are slower then the corresponding operations executed on the `PriorityQueue`.