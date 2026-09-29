---
name: to-spec
description: Turn the current conversation (usually a finished grill-me session) into a spec, saved as SPEC.md in the plan directory. No interview, just synthesis of what has already been discussed. Use after grilling, or when the user says "write the spec", "turn this into a spec", "to-spec".
---

# to-spec

Take the current conversation context and codebase understanding and produce a spec. Do
NOT interview the user; just synthesize what you already know. If the plan has not been
grilled yet, say so and offer `/grill-me` first.

## Process

1. **Explore the repo** the plan targets, if you haven't already, to understand the
   current state of the code. Use the project's glossary vocabulary (`context/glossary.md`)
   throughout the spec, and respect any ADRs in the area you're touching.

2. **Sketch the seams** at which the feature will be tested. Prefer existing seams to new
   ones, and use the highest seam possible. The fewer seams across the codebase, the
   better — the ideal number is one. Check with the user that these seams match their
   expectations.

3. **Write the spec** using the template below and save it as `<plan-dir>/SPEC.md` — always
   that exact filename, never a slug-named file. One plan, one directory, one `SPEC.md`.
   If the plan came from an issue, link it in a bold `**Epic:**` line directly under the
   title, so `/to-issues` can find it later:

   ```md
   # <Plan title>

   **Epic:** [<owner>/<repo>#<n>](<issue-url>)
   ```

Do not pin a library's or component's current version from memory. Verify it against a
primary source or write it version-agnostically, and don't launder a version stated
earlier in the conversation into the spec unless it was verified there.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this plan.

## Further Notes

Any further notes about the feature.

</spec-template>
