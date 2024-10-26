# Structured Concurrency Anti-Patterns

## Priority Inversion

TBS

## Inadvertently sharing a mutable state:

```swift

struct NonSendableType {
    var int: Int
}

final class ClassWithAsyncFunctions: Sendable {
    var type: [NonSendableType]

    func doStuff() async -> NonSendableType {
        // mutate state concurrently

        return type.first
    }
}


```

## Out-of-order operations, data races without corrupting the data. 
```swift
try await withThrowingTaskGroup(of: QuakeLocation.self) { group in
    let count = Counter() // Create a counter object.
    var locations: [QuakeLocation] = []
    for quake in quakes {
        group.addTask {
            let location = try await provider.location(for: quake.url)
            // Increment the counter after fetch completes.
            count.increment() 
        }
    }
    while let location = try await group.next() {
        locations.append(location)
    }
    // This will sometimes fail.
    assert(counter.count == locations.count) 
}

```
Counter may read the same value twice, and add to it, resulting in incorrect count. Unprotected mutable state counter in shared
between multiple threads.

Also, Sendable class will allow you to return-nonsendable type, but actor will not, what gives?

_@Sendable and actor-isolation. @Sendable closure cannot escape the syncronization context where it was formed.
When can an actor create a task,
which runs concurrently with itself, breaking actor isolation
How are actors and critical sections related?_

_Where is async function executed, when it does not have an actor? Does async create its own new task?_

_Implement executor to avoid blocking_

### Depending on the state, which maybe changed across suspension point. Depending on the order on which the actor was awaited.

Actors act similarly to serial DispatchQueue, but there is a difference - (partial) Tasks awaiting an actor are not guaranteed to run in the same order they are received.

Two calls are made to the actor. In both I expect state to look in a certain way. But, these calls are not ordered.
See AyncQueue git hub repo.  

_Inadvertent race condition, actors can run tasks concurrently, they do not guarantee the order of execution. 
The only runtime guarantee is that async function will return to the same actor it was called from_

```swift
actor MyActor {
    var data: Data?
    
    func appleData() async throws -> Data {
        guard let data = self.data else {
            newData = try await httpData(from: URL(string: "https://www.apple.com")!)
            self.data = newData
            return newData
        }
        return data
    }
}
```
A second request to network will happen, if it's done while the first one is in-flight.

### Two instances of the same actor.
Every actor instance is unique. Different isolations? basically a path way to modify the same shared data.

Accessing some shared state, again, data race.

Notes:
all async functions are executed as a part of some higher level task. Task is run as a part of a job, every actor has a job executor, which executes job on a single pool of threads.

## Blocking main thread with IO bound or CPU-bound work - result of cooperative model.

Is network parallelization really worth it?

## Blocking other threads with a CPU-intensive work

_Actors interleave code, run it in a separate context (CPU-intensive work). Async should avoid calling functions that block the thread_

_Should yield really_

[Apple Dev Tutorials - Caching Network Data](https://developer.apple.com/tutorials/app-dev-training/caching-network-data)

## Thread-explosion from using GCD under the async interface. Deadlock.
[Sample post 1](https://forums.swift.org/t/cooperative-pool-deadlock-when-calling-into-an-opaque-subsystem/70685/8?page=2)
[Sample post 2](https://forums.swift.org/t/deadlock-when-using-dispatchqueue-from-swift-task/66058)

When thread pool is exhausted, DispatchQueue will not overcommit, seeing that Tasks promise not to lock on future work.

>The specific implementation reason this blows up today is that both Swift concurrency and Dispatch’s queues are serviced by same underlying pool of threads — Swift concurrency’s jobs just promise not to block on future work. So when Dispatch is deciding whether to over-commit, i.e. to create an extra thread in order to process the barrier block, it sees that all the threads are tied up by Swift concurrency jobs, which promise to not be blocked on future work and therefore can be assumed to eventually terminate without extra threads being created. Therefore, it doesn’t make an extra thread, and since you are blocking on future work, you’re deadlocking.

## Debugging

State of all threads.
```bash
lldb> bt all
```

## TODO
Isolated vs non-isolated keyword.