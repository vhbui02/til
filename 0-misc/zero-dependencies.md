# Zero Dependencies

<!-- tl;dr starts -->

Think twice before adding a dependency.

<!-- tl;dr ends -->

## Dependency is bad?

- Use dependency on a small application.
- Dependencies depend on each other, forms a dependency chain. Updating one dependency affect the dependencies before and after it. Same project have different direct dependencies relying on different versions of a transitive dependency.
- More bug-prone as the price of running somebody else's code along side your code. It's hard to debug because dependencies are black-boxed, you don't know what they're doing.
- The responsibility of security auditing a dependency lies with that dependency's developer/maintainer. If they're lazy, a lot of unpatched vulnerabilities might exist for a long time. If they're hijacked, it's worse. That's why direct dependencies and transitive dependencies can introduce supply-chain attack surface.
- Senior developers' experiences show that using dependencies can increase development but drastically reduce maintenance time and cost.
- When reaching the edge of a dependency, you will need to reach out to the package developer/maintainer. If you're lucky, they will add the features that you need. If not, you will have to fork it, implement it yourself and then maintain this separate fork for the rest of your life.

## When dependency is NOT bad?

- Use dependency on a large application.

## The benefits of zero dependencies

- You control anything. There is no redundant code. Code can be changed at any point of time to reflect the business requirements.
- Evaluate the logic of the dependency. If it's small and reproducible, don't add dependency. That level of reinventing the wheel is acceptable.
- Switch on writing microservices, break your app into smaller pieces, each of those pieces use much fewer dependency.

> Unix philosophy.