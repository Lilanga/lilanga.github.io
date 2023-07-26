---
title: Active development on legacy codebases
date: 2018-04-30 16:30:00 +0530
categories: [Maintenance, Best practices]
tags: [Refactoring, Development, Tech-debts]
render_with_liquid: true
image:
  path: /assets/img/posts/2018-04-30/cover.png
---

Legacy usually refers to neglected codebases and not using proper standards anymore when evolving the codebase. These codebases are generally large and complex, hard to read and understand. Legacy codebases often need more appropriate unit test suits to cover business logic while exhibiting severe inter-component dependencies, which creates a miracle to do so.

Business logic is unclear to understand by following code lines, undocumented and forgotten; changing one logic can break many unseen workflows and may go unnoticed until production deployments.

![reading-legacy-code](/assets/img/posts/2018-04-30/reading-legacy-code.png)
_dilbert comic strip <https://dilbert.com/strip/2006-12-08>_

Working with legacy code is often stressful and may need dedicated effort to untangle complex logic. When the size of the new changeset of the codebase is increasing, the risk of breaking existing workflows is also growing exponentially.

Also, legacy codebases often come with inherited technical debts, work-around logic, and quick patches to temporarily address issues. Less documentation and less understandability of the original developer's intention make new development even harder by utilizing the existing code.

## Dealing with legacy codebases

Proper tools and methods are always needed to mitigate risks when changing legacy codebases. Refactoring tools, Reference finders, and Dependency graphs help understand code structure and apply minimal viable changes without breaking the business logic.

Guarding new logic with unit tests are proven technique to prevent breaking new business logic with future bug fixes. Adding unit tests for new code will also be helpful in long-term development.

## Approaches to maintaining legacy codebases

![cleaning up codebase](/assets/img/posts/2018-04-30/maintain-your-codebase.jpg)
_mainining the legacy codebase_

These are three main approaches when developing code on top of legacy code without rewriting the whole application from scratch.

### Plug-In additional logic as a fresh development

The safest approach to developing new features on top of legacy codebases is to change existing logic as little as possible.

Design new code with minimum interaction with the existing codebase and write all the possible logic as a separate module. In most cases, wiring up the new module to the current codebase is the only change done to the existing system. New business logic is developed as adapters and plugs into the legacy codebase with minimal change sets.

Although this is the safest and less effort-consuming approach, there are better options from an engineering perspective. There will be lots of repeated logic used with different customizations and even the exact code segment repetitions. Addressing dependencies will introduce additional complexities. There will be performance impacts due to complex code wire-ups. By following workaround approaches to mitigate scarier code changes, things will be even more complicated, and future maintenance will become even harder.

### Adapting to the existing design

The second approach is to follow the original developers' same practice and programming style and introduce changes incorporating existing architecture.

Developers need to study the impacted area thoroughly to understand the existing business logic implementations to incorporate new changes. Most of the time, 90 percent of the time was spent reading and understanding the existing workflow and 10 percent on the new logic implementation. The new code mimics old developers' programming styles and practices when possible. A peer must review This new development carefully to spot any anomalies. The first iteration of the solution can be error-prone due to unseen business logic impacts and need thorough regression cycles to surface errors. Usually, need lengthy bug fixing cycles after new feature development.

### Improve the codebase before feature development

The best approach to implementing new features on top of legacy codebases is to refactor the codebase. Refactoring must be done before the feature development effort to improve its quality by removing complexities and dependencies and making the codebase unit testable and reusable. Feature development consists of two tasks. Firstly refactor the module to a stable state, followed by a complete testing cycle to spot any bugs. Then the actual feature development is happening on the refactored module.

A dedicated effort is needed to identify impacting areas for refactoring. When all the related components are in a maintainable, less complex state and well-tested for error-free, new feature development starts on top of that. This approach uses less effort than rewriting the application but consumes significant effort compared to other development efforts. This option highly embraces the “boy scout rule,” which says the codebase should be in a better state after development when compared with the initial state.

In legacy codebases, Continuous improvements are expected with the development efforts. Usually, every new change is done to the system should be done by improving its overall design.
