# Hash Hash Hash
In this lab, we need to make a hash table implementation safe to use concurrently given a serial hash table implementation and two additional hash table implementations to modify. We need to implement two locking strategies and compare them with the base implementation by only adding mutex locks.

## Building
```shell
make
```

## Running
```shell
./hash_table_tester -t [num_threads] -s [num_element]
```
Example:
```shell
./hash_table_tester -t 4 -s 100000
```

## First Implementation
In the `hash_table_v1_add_entry` function, I added one mutex lock for the program. Before insertion, a thread needs to acqurie a lock before continuing its operation.

### Performance
```shell
./hash-table-tester -t 4 -s 100000
Generation: 158,056 usec
Hash table base: 1,964,421 usec
  - 0 missing
Hash table v1: 3,583,050 usec
  - 0 missing
Hash table v2: 1,245,258 usec
  - 0 missing
```
The thread needs to pass mutex before entering the critical section, which helps prevent race condition.

Version 1 is a little slower/faster than the base version because of overheads. As there is only one lock, thread must acquire the lock before insertion, deletion, and search. And when one thread holds the lock, other threads cannot operate.

## Second Implementation
In the `hash_table_v2_add_entry` function, I add one lock per bucket for the hashtable. This implementation enables multiple threads to insert elements into different buckets, preventing threads fighting for one lock.

### Performance
```shell
./hash-table-tester -t 4 -s 100000
Generation: 146,019 usec
Hash table base: 2,035,268 usec
  - 0 missing
Hash table v1: 3,516,874 usec
  - 0 missing
Hash table v2: 1,212,150 usec
  - 0 missing
```
Analysis:
The v2 implementation enables a much better performance than v1 because it has one lock per bucket instead of a global lock.
By implementing a lock for each individual bucket in the hash table, I enable the possibility for multiple threads to work concurrently, as long as they target different buckets. This approach considerably reduces contention in comparison to utilizing a single global lock, where access by one thread to any part of the hash table would prevent other threads from executing operations on distinct buckets. Also, this method shows improved scalability with an increasing number of threads. As the workload grows, the chance of multiple threads simultaneously needing to access the same bucket decreases, compared to the likelihood of them requiring access to any section of the hash table. Therefore, the strategy of assigning a lock to each bucket demonstrates effective scalability with the growth of both the number of threads and the size of the hash table.

More tests:
```
./hash-table-tester -t 4 -s 50000
Generation: 75,155 usec
Hash table base: 375,908 usec
  - 0 missing
Hash table v1: 784,610 usec
  - 0 missing
Hash table v2: 292,436 usec
  - 0 missing

./hash-table-tester -t 8 -s 50000
Generation: 132,546 usec
Hash table base: 1,901,019 usec
  - 0 missing
Hash table v1: 2,800,461 usec
  - 0 missing
Hash table v2: 1,517,435 usec
  - 0 missing

python -m unittest
.Running tester code 1...
.Running tester code 2...
.Running tester code 3...
.
----------------------------------------------------------------------
Ran 3 tests in 27.429s

OK
```

## Cleaning up
```shell
make clean
```