# Glossary

1. Partial task - child task.
2. Execution-context - ??
3. Interleaving - interrupting execution of a function, and run a separate function call on the same thread (breaks atomicity).
4. Task - an atomic unit of work.
5. Actor isolation - only a single thread accesses the data at any given time.
6. Synchronization context - actor.
7. Cooperative multitasking - a multitasking where OS never initiates context switch. Instead task yields its thread periodically.
8. Preemptive multitasking - OS decides which task to run and for how long.
9. Unstructured task - does not have a parent task. Start with Task or Task.detached. Task creates task that is part of the current actor, Task.detached creates task that is not the part of the current actor.