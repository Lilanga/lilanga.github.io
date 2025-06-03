---
title: Avoiding Build Breakages — Essential Practices for Continuous Integration
date: 2018-01-01 20:00:00 +0530
categories: [Continuous Integration, Build Automation]
tags: [CI, DevOps]
render_with_liquid: true
image:
  path: /assets/img/posts/2018-01-01/cover.png
---

Build breakages are expected as part of our day-to-day development tasks when the code transforms and evolves rapidly. Build breaks are inevitable when many people work on a project, and the code base is highly distributed.

![Breaking builds](/assets/img/posts/2018-01-01/breaking-builds.png)
_Image is copyrighted to Geek & Poke under a CC License_

## Avoiding build breaks is the priority

We need to plan to mitigate build errors and have a proper approach to finding and eliminating them as soon as possible. With our strategy, we should be able to block build error propagation to other development and deployment environments and address them as quickly as possible. When the build passes, we can guarantee the code can build on any environment that meets the required setup.

## Benefits of frequent Continouos Integrations

The basic idea of continuous integration is to integrate code changes more rapidly. Each integration is verified by an automated build with a test suite to identify possible build or functionality errors as well as the quality of the code. Each developer is expected to integrate their changes frequently — the duration may vary on the team setup and development phase of the team.

If you’re the only one contributing to a code base, then a deferred commit won’t be that crucial. However, if you’re in a team with several developers, and everyone is thinking, “Oh well, I’ll wait to integrate my changes until completing my feature fully,” then you’ll often end up handling a lot of unwanted conflicts and losing time. Late aggregated code merges will lead to more rework, refactoring, and underutilizing of shared code segments(No DRY).

The general rule would be:

>
- Commit as often as possible.
- If you think “it’s not ready yet” (because it’ll break the integration build or simply shell code), create a branch and commit without pushing to the main branch.
- Make sure you practice proper commits with incremental changes.
{: .prompt-info }

At the end of each task, developers are expected to fetch the latest changes from upstream. This practice significantly helps teams minimize integration conflicts and logic deviations, among the other benefits.

With proper continuous integration in place, developers are always starting their feature developments from a known stable code base, guaranteeing the expected functionality achieved using an automated test suite and desired code quality achieved using static code analysis.

If a continuous integration build is broken, it is essential to fix it fast as possible to guarantee the reliability of the main branch.

## Planning to prevent builds breaks

![Alt text](/assets/img/posts/2018-01-01/preventing-build-breaks.png)
_planning for preventing build breaks_

There are some golden rules to avoid breaking builds in agile, dev-ops-friendly environments.

### Commit smallest changes possible

It is crucial to commit your changes frequently. When you are done with a workable unit in your feature or functionality, commit it and push it to the integration branch. When doing so, firstly, you have to update your working copy to match the integration branch and resolve any conflicts locally.

 You can continue your work if the build passes and the commit test suite is healthy. Do not wait till the whole story gets completed. Developers must check in their changes as soon as possible when working with a team to maintain a conflict-free, healthy integrations environment.

### Avoid end of the day bulk check-ins

Research revealed that usually, integration build issues happen late evening hours. That can be because many developers accumulate all the changes they are doing throughout the day and try to check in at the end of the day, leading to many conflicts and build breaks.

### Before check-in, make sure the build is healthy

Always check the build status before doing a check-in. If your local build fails, you must fix it before pushing changes to the source branch. More importantly, if the integration build fails, you must wait until it gets resolved to push your changes. Do not check in over broken builds. It may complicate fixing the integration build, and your change set may be altered with undesired effects.

### Run commit test suites locally before check-in changes

Define a commit test suite to align with your feature work. Run that suite and verify the correct functionality before pushing your changes upstream. Prevent intentionally polluting the integration branch with faulty/half-baked code since that change may propagate and be used by other developers before you can apply a fix.

### Complete all the steps of the commit cycle

Committing your changes upstream does not finish your development cycle. You must wait until the Continuous Integration build is completed to see how your changes are integrated with others' changes. Also, look for unit test results to identify any functional issues and static code analyzer reports for any code quality issues. If build issues are identified, you need to attend to them immediately.

### If you break it, you make it

Suppose the integration build is broken due to your change sets. In that case, you must take full responsibility to fix it as soon as possible.

The owner of the change set is the best person to address the issue effectively. Consider build breaks the highest priority items since they can spoil the other developer environments and will also interrupt the team’s development workflow significantly. If fixing the integration build take considerable time, revert changes and bring the integration branch to the previous state. You can push your changes back after the fix is applied locally and tested correctly. Take complete ownership of build breaks happening due to your code changes. You are the best person to address them.

### Do not tweak build failures to make it work

When fixing build breaks, apply a proper fix. Do not tweak it by bypassing validations or commenting out code segments hoping to patch it later. Take your time for a proper fix. You can always revert the branch to the known healthy state if it takes time.

## Enhance Continuous Integration workflows to mitigate build breaks

These are some essential steps that need to happen in the integration workflow to ensure we are identifying any build breaks early.

### Following proper branching strategy

A proper branching strategy is crucial in avoiding unnecessary conflicts and building breaks. With a proper branching strategy, developers can easily sync with the main integration branch and fix possible conflicts and issues locally before check-in the code. Git workflow is a popular branching strategy for continuous integration enabled git repositories.

### Build notification system

The continuous integration build system should be configured with the proper notification system. If the build is failing, a notification (e.g., email alerts, hipchat notifications) should be triggered to notify the team and responsible developers immediately.

### Pull requrest build system with prior merging

The pull request stage is one of the best places to avoid build breakages. We can configure the pull-request build system with prior merging. In that case, a pull request will merge virtually before the build, and the merged code will be input to the build job. We need to run the commit test suite with the build job. So we will be able to identify build and functional breaks before we push changes to the main integration branch.
