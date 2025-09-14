# Code Review Interview

## Requirements

Begin with assessing requirements:

First of all under what context am I reviewing this code? So that I can have an idea of what I should be looking for, has as a new feature been added to an established code base? Is this a greenfield project I should be reviewing as if everything is new code? This will give me an idea of what I should be looking out for more carefully in the review and if I should focus on certain areas.

As the reviewer do I have an understanding of what I'm reviewing? Can you give me a brief rundown of the requirements of the system I am reviewing, or would you like me to evaluate that and find that out for myself to the best of my ability?

Do we have any performance constraints (latency, throughput, compliance)? Service level objectives?

## Review

If I haven't seen the code base before I like to begin by taking a high level look at the codebase. I'm going to determine the entry point for interaction with the code I am reviewing, and walk through the steps the code is proceeding through.

Doing this I'm going to discover the different components, and build a mental map of the code base. Along the way I will note down any red flags I noticed in my review.

Once I'm done my preliminary discovery of the code, I will look again for bugs & things that are incorrect. This is my primary purpose in the review.

1. Bugs
2. Design
3. Look for "Clever" Code
4. Check for Code Duplication
5. Descriptive naming
6. Performance Improvements
7. Tests
8. Request changes and explain them
9. Code Documentation


I'm going to think about the design decisions made, and see if there is a large downside to the way the code has been designed. If so I will make recommendations on design changes. I'm concerned with testability, potential edge cases. How are errors being handled

Deep dive on:

* Input validation, invariants, edge cases, pre/post-conditions
* Error handling, propogation, clean death, logging
* Concurrency: (granularity, deadlock risk, lock order), atomicity
* Runtime (complexity)
* Logging, metrics
* Code smell: Cyclomatic complexity, naming, cohesion, coupling, Consistent style
* Tests: Unit tests, Integration tests, Concurrency/race
* Security
