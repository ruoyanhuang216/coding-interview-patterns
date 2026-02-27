# Coding Interview Patterns

A personal reference guide for coding interview preparation. Each pattern includes high-level strategy, a methodology for how to talk through problems in interviews, and a checklist of practiced LeetCode problems grouped by archetype — all with commented Python 3 solutions.

## Patterns

| # | Pattern | Key Idea | Groups |
|---|---------|----------|--------|
| 1 | [Two Pointers](patterns/two-pointers.md) | Search space reduction on linear data using two traversal indices | 5 archetypes, 10 problems |
| 2 | [Fast & Slow Pointers](patterns/fast-and-slow-pointers.md) | Different-speed traversal for midpoints, cycles, and implicit graphs | 3 archetypes, 8 problems |

## How This Guide Is Organized

Each pattern file follows a consistent structure:

1. **What is it?** — One-paragraph definition and core philosophy
2. **When to use it** — Recognition checklist for interviews
3. **Real-life analogies** — Intuitive mental models
4. **Archetypes** — Problems grouped by sub-technique, with interview logistics for each
5. **Problem checklist** — Each problem includes:
   - An interview "spiel" (what you'd say out loud to the interviewer)
   - Time & space complexity
   - Fully commented Python 3 solution

## How to Use This During Prep

- **Before a session**: Read the pattern overview and archetype descriptions to prime recognition
- **During practice**: After solving a problem, compare your approach with the solution here
- **Before interviews**: Review the "spiel" sections — practice saying them out loud

## Adding a New Pattern

1. Create `patterns/<pattern-name>.md` following the template in any existing pattern
2. Add a row to the table above
