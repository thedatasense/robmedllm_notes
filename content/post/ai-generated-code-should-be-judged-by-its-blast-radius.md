---
title: "AI-Generated Code Should Be Judged by Its Blast Radius"
date: 2026-09-11T03:00:00+00:00
lastmod: 2026-09-12
slug: ai-generated-code-should-be-judged-by-its-blast-radius
tags: ["ai-assisted-coding", "software-engineering", "agents", "code-review", "production-systems"]
description: "When AI generates code faster than teams can review it, quality depends on risk, clear ownership, independent tests, and a review queue that has limits."
---

Imagine a team of six receiving 30 artificial intelligence (AI) generated pull requests (PRs) a day. Some touch thousands of lines. One changes authorization behavior and then replaces the relevant dependency with a mock in its tests. The test suite passes. The reviewers are exhausted.

What should the team do next?

I would start by questioning what the organization is calling progress. If management rewards code volume while leaving review capacity unchanged, it is encouraging work to accumulate at the point where someone must establish whether the changes are correct. Asking reviewers to work faster does little to resolve that imbalance.

My view is that generation should move at the pace the team can responsibly verify. Producing a change and establishing that the change belongs in a system require different kinds of work. A tool that accelerates the first can still leave the second more expensive.

The review queue makes that difference visible.

## The standard should follow the blast radius

I can write a script, run it once, inspect the output, and delete it before lunch. I can also write a service that handles customer data through an application programming interface (API) and remains in production for five years.

The verification these programs deserve depends on what can happen when they fail.

I find **blast radius** useful here. How far can a failure travel, and what sits in its path? A local data-cleaning mistake might waste an afternoon. A migration can corrupt records that other systems have already consumed. Restoring the database may leave those downstream effects untouched.

| Dimension | Question |
| --- | --- |
| Impact | What happens to users, data, money, or safety if the code is wrong? |
| Exposure | How many people or systems can encounter the failure? |
| Reversibility | How quickly can we detect the effect and undo it? |
| Lifetime | How long will the code remain in use, and how often will it change? |

A temporary experiment with isolated data may justify inspecting its result and moving on. A production service needs evidence about its interfaces and failure behavior, along with a maintenance path. Changes involving sensitive data or irreversible actions deserve closer examination and explicit ownership.

Repository labels do not settle this. An internal script connected to payroll can have a larger blast radius than a public page. And code expected to live for years will encounter conditions that its original tests did not anticipate.

The difficult part is applying that standard when submissions exceed the team's capacity. I would treat reviewability as a condition of submission, with the required evidence proportional to the consequences of getting the change wrong.

## Thirty PRs a day is a capacity decision

Consider an illustrative calculation. If each of 30 PRs requires 30 minutes of review, the queue consumes 15 engineer-hours a day. Split evenly across six people, that is two and a half hours each, before revisions, interruptions, or their own engineering work. Those assumed review times could be far too low for changes involving unfamiliar domains.

As arrivals exceed completions, unfinished work accumulates. Faster generation then increases waiting time unless the team reduces unnecessary submissions or expands its ability to verify them.

Human review was a constraint long before large language models (LLMs). Teams have always had to reconcile local changes with shared requirements. Management has also long rewarded visible feature work while underfunding maintenance. Agents can amplify those incentives by making a large patch cheap to produce.

So I would stop treating the review queue as an individual productivity problem. The team needs an explicit agreement about how much work it can accept and what must happen before that work arrives.

The objective should be accepted, maintainable behavior. Counting generated lines or opened PRs measures activity without establishing that the system improved.

## The expensive failure is losing understanding

Technical debt is only part of the cost. A team can lose the ability to explain what its software does.

An agent may generate 1,000 lines where an engineer familiar with the system would reuse an existing function and add 50. The larger implementation can satisfy every supplied test while introducing dependencies and assumptions nobody needed.

Specifications rarely include all the reasons a codebase looks the way it does. Some constraints live in documentation. Others live in incident history or in an engineer's memory of why a simpler-looking approach failed.

I would direct review effort toward places where missing context can conceal a consequential mistake:

| Pattern to examine | What the reviewer has to establish |
| --- | --- |
| Duplicated helpers and unused code | Whether an existing implementation already solves the problem |
| New services or endpoints | Whether the dependency exists and its documented behavior matches the call |
| Early exits through middleware | Whether authorization and other required checks still execute |
| Tests that replace security behavior with mocks | Whether any test exercises the actual boundary |
| Files scattered across unrelated modules | Whether the change respects ownership and dependency direction |
| Broad data retrieval and repeated fetching | Whether the implementation has acceptable cost under realistic use |

These checks follow from the behavior the system needs to preserve. Their priority should depend on the change's blast radius and the team's own defect history.

A reviewer who cannot explain the intended behavior cannot reliably assess the implementation. The submitting developer should supply that explanation before asking someone else to reconstruct it.

## Make the author responsible for review readiness

Submitting a PR should mean that its author has inspected the diff, run the relevant checks, and can explain why each substantive change is present. Prompting an agent does not satisfy those obligations by itself.

The same requirement should apply to every contributor. Singling out juniors misses the mechanism: anyone can transfer unfinished reasoning to a reviewer. Less experienced engineers often need closer pairing to learn how to recognize a plausible but incorrect solution. A blanket prohibition teaches less than a bounded task followed by a careful walkthrough.

For the overloaded team, I would establish the following admission rules.

| Rule | Evidence required before domain review |
| --- | --- |
| One coherent purpose | A linked issue and a plain-language account of the user-visible problem |
| Author self-review | Confirmation that the author examined the diff and removed unnecessary changes |
| Explainable implementation | The cause of the problem, the chosen approach, and the reason each affected component changes |
| Visible verification | Relevant test results, including explanations for modified or removed tests |
| Real dependencies | Documentation links for new external services, endpoints, or packages |
| Reviewable scope | A change small enough to assess, or an agreed plan for reviewing a larger one |
| Clear ownership | The submitting team's review of its business logic before another team reviews domain integration |

A template helps collect this evidence. It cannot establish that an author understands it, especially when a model can generate the description too. A short walkthrough often reveals more: what existing behavior had to remain true, and where does the change demonstrate that?

Enforce mechanical rules through continuous integration (CI) and repository settings. Local commit hooks can help contributors catch problems early, but contributors can bypass them. Required checks and protected merge rules belong at the shared boundary.

Reviewers should also be allowed to stop. After finding several substantial problems, they can identify the pattern, state where review ended, and return the change for revision. They should not have to enumerate every defect in a submission that was never ready.

Keep that response specific and professional. Public shaming and hundreds of machine-generated complaints create another queue of work.

## Small changes help when they preserve meaning

Size limits are useful signals. They are poor substitutes for judgment.

A 200-line authorization change can require more scrutiny than a 2,000-line mechanical rename. Generated files can dominate a diff without dominating its risk. Conversely, a short patch can quietly remove an important check.

I would use a size threshold to trigger a conversation, with exceptions for migrations or coordinated changes that have a documented review plan. The author should separate mechanical edits from behavioral changes when that separation is safe.

Splitting one inseparable change into five patches that must merge in an exact sequence can make deployment harder to reason about. Stacked PRs help when reviewers can understand the dependencies and the intermediate states are safe.

The useful unit is a coherent decision a reviewer can assess. Small commits also make fault isolation and rollback easier, provided the team has considered data changes and external effects that reverting code will not reverse.

## Tests need an expectation independent of the implementation

Tests describe selected behavior. A passing suite tells us little about a requirement it never exercised.

Before generating production code, I would establish acceptance criteria and tests for the important constraints. Humans do not need to type every assertion. They do need to decide what counts as correct.

Suppose a request without permission must never reach a protected operation. If the implementation bypasses authorization, and its generated tests mock authorization away, code and tests can agree while the requirement is violated.

That is why letting the same generation pass freely rewrite implementation and expected behavior deserves caution. Sometimes a legitimate requirement change needs changes to each. Make that decision visible, and review the changed expectations separately.

| Verification layer | What it contributes |
| --- | --- |
| Unit tests | Local behavior, edge cases, and useful fault isolation |
| Integration and contract tests | Evidence that real component boundaries match their assumptions |
| End-to-end tests | Selected user journeys through the assembled system |
| Security tests | Checks on access control, trust boundaries, and unsafe inputs |
| Static analysis and dependency checks | Detection of certain defects before execution |
| Recovery exercises | Evidence that the team can restore service and recover data |

Use mocks to isolate behavior deliberately. Pair them with tests of the real boundaries they replace. Otherwise, a suite can become very precise about an imaginary system.

Testable design matters too. Clear interfaces and limited side effects tend to make failures easier to locate. Breaking a function into dozens of tiny wrappers, however, can add indirection without improving the evidence.

The test suite should grow with the system's risks. An isolated script does not need a disaster recovery exercise. A service whose failure would interrupt the business may well need one.

## AI review can direct attention

I would use a second model to challenge a change. I would also remain responsible for judging its findings.

A useful review request includes the ticket, the diff, and the relevant architectural constraints. Ask where the implementation exceeds scope, which assumptions lack tests, and whether a required check can be bypassed. Require concrete references to the changed code.

A different model may catch mistakes the generator missed. It can also share the same mistaken assumption, particularly when each receives the same incomplete description. Agreement between models does not establish independence.

| Useful model assistance | Human judgment still required |
| --- | --- |
| Summarize a diff and identify affected paths | Check the summary against the actual changes |
| List possible defects with locations | Determine which findings are real and consequential |
| Review security or performance assumptions | Check those assumptions against the deployed system |
| Suggest simpler implementations | Decide whether the simplification preserves required behavior |
| Identify missing test cases | Establish whether the proposed expectations are correct |

Builder and critic loops need a stopping rule. Repeated passes may uncover defects, but they can also produce speculative objections and unnecessary rewrites. Stop when the relevant findings are resolved and the required evidence is present. An indefinitely growing critique is not a quality metric.

Specialized reviewers can be useful when their responsibilities are explicit. A security reviewer and a performance reviewer should receive the context each needs, with a human assessing unresolved risks. Adding agents without defining their task often multiplies commentary.

I would judge an automated reviewer by whether it improves defect detection or reduces review effort on the team's own changes. Count the consequential mistakes it finds, but also the time spent dismissing false alarms. A long review comment is easy to produce.

## Git views, explanations, and documentation reduce setup time

Review often starts with reconstructing context. Good Git tooling can reduce that cost.

A view of the complete diff helps reveal unrelated edits that disappear when reviewing files individually. Ignoring whitespace is useful for formatting changes, although it deserves care in languages where whitespace affects behavior. A model-generated explanation can give the reviewer a starting map, provided the reviewer checks that map against the patch.

Documentation should answer questions at the level where decisions occur.

| Level | Questions the documentation should answer |
| --- | --- |
| System | Who uses this, what problem does it solve, and where does data travel? |
| Component | What does this part own, and what promises does it make to its callers? |
| Local implementation | Which state, assumptions, or side effects would surprise a maintainer? |

Automatically restating every method signature can make documentation longer without making review faster. Spend the effort on intent and constraints that code does not explain on its own.

Architecture checks can remove another class of repetitive review. Dependency rules can flag forbidden coupling or circular references before a human opens the PR. Compilers and type checkers also catch particular errors, but neither compilation nor a language choice establishes correct business behavior.

## Verification continues after deployment

Some failures require real workloads to appear. Structured logs and useful error reports help connect an incident to the internal state that preceded it. That evidence should inform a regression test when possible.

Keep development and staging separate from production. Coding agents should work in controlled environments with access appropriate to their task. Production changes deserve a defined release process, observable results, and a recovery plan.

Practice recovery in non-production environments. A team that has never restored its backups does not yet know whether its recovery plan works under pressure.

And a rollback plan needs to account for more than source code. Messages already sent, records already exported, and downstream decisions may persist after a deployment is reversed. Poor reversibility raises the verification required before release.

## What I would change for the six-person team

I would begin with the queue itself. Pause admission of changes that lack an owner, a coherent purpose, or the required evidence. Have the submitting teams perform their own review first, then define exactly which parts require the domain team's approval.

Next, reserve explicit review capacity. If incoming work exceeds it, leadership must choose which work waits or which other commitments move. The cost exists whether management acknowledges it or asks engineers to absorb it after hours.

| Immediate action | What it changes |
| --- | --- |
| Require author self-review and test evidence | Returns basic preparation to the submitting team |
| Agree on scope and size exceptions | Prevents surprise submissions that consume days |
| Protect important regression tests | Makes changes to established expectations visible |
| Add automated checks for recurring defects | Removes repeated mechanical work from human review |
| Pair contributors on rejected changes | Builds the understanding needed for better submissions |
| Limit work awaiting review | Connects generation to available verification capacity |

Then measure the result. Track waiting time and hands-on review time separately. Record rework, recurring defects, and incidents associated with merged changes. Code volume alone cannot tell management whether the process is improving.

I would make productivity claims only after measuring the full path from task definition to accepted change, including review and rework. Faster generation can save time, but the gain depends on how much additional verification the output requires. A universal multiplier tells me little about that trade.

When leadership demands faster delivery, the discussion should include what is being displaced and what uncertainty remains. Document the decision and its owner. Deliberately allowing a preventable failure to prove a point transfers the cost to users and colleagues.

The engineering skill I most want to preserve is knowing when to delete code, reuse something that already exists, or avoid writing the feature at all. An experienced developer can make that decision before an agent creates a large implementation. Training should help newer engineers learn the same judgment.

For a disposable experiment, moving quickly and inspecting the result may be enough. For a system that people depend on, the team needs to understand its important behavior and explain the evidence supporting a change.

If an agent can produce more code than we can responsibly accept, I would reduce the amount it produces. The review queue is already telling us where the next engineering decision belongs.
