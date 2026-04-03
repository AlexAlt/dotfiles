---
name: workflows-brainstorm
description: "Interactive thought partner for exploring ideas and clarifying requirements. Use before implementation to understand what you're building and why. Invoked with /workflows:brainstorm [feature idea or problem to explore]."
---

# Brainstorming Ideas Into Requirements

Help turn ideas into clear product requirements through collaborative dialogue. You are a thought partner — focus on understanding the **what** and **why**, exploring product-level approaches, and leaving detailed implementation to later stages.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Explore solution approaches with the user, then present the requirements in small sections, checking after each whether it looks right.

## Scope Boundaries

**Does:**
- Clarify the problem being solved
- Explore user intent and goals
- Explore product-level solution approaches with light technical framing
- Define requirements and constraints
- Establish success criteria (user-facing, not technical)
- Identify what's explicitly out of scope

**Does NOT:**
- Design implementation architecture, data models, APIs, or components
- Make low-level technical decisions (framework choices, data structures, etc.)
- Cover error handling, testing strategies, or data flow

Those belong in a planning phase after codebase research.

## Initial Setup

When this skill is invoked:

1. **Check for project folder argument**
   - If argument provided (e.g., `/workflows:brainstorm 2025-01-30-ENG-123-feature`):
     - Verify folder exists at `~/.cursor/project-specs/[argument]/`
     - Read `spec.md` if exists (for resuming)
     - Inform user: "Resuming spec for project [folder-name]."
   - If no argument, proceed to step 2

2. **Auto-detect recent project folders** (if no argument):
   - Find folders from last 30 days in `~/.cursor/project-specs/`
   - Use **AskQuestion tool** to show options:
     - [folder-1] (spec: yes — resume)
     - [folder-2] (spec: no — add spec)
     - Start fresh (new project folder)

3. **If no folders or user wants fresh**: Proceed to Clarity Gate

## Clarity Gate

Before diving into the full Q&A process, assess the user's input.

**Clear requirements indicators:**
- Specific acceptance criteria provided
- Referenced existing patterns to follow
- Described exact expected behavior
- Constrained, well-defined scope

**If requirements are already clear:**
Use **AskQuestion tool** to offer: "Your requirements seem detailed enough to skip the exploration questions. Should I go straight to writing the spec, or would you like to explore the idea further?"

If user chooses to skip: run Phase 1 (Lightweight Repo Research) to ground the spec, then jump directly to Phase 4 (Present the Requirements).

**If requirements are unclear or ambiguous:** Proceed to The Process.

## The Process

### Phase 1: Lightweight Repo Research

If invoked inside a git repository, run a quick scan before asking questions:
- Check CLAUDE.md, AGENTS.md, or similar project guidance files
- Look for features/patterns related to the brainstorm topic
- Glance at recent relevant commits

Use findings to ask more informed, context-aware questions. If not in a repo (or the topic is greenfield), skip this step.

### Phase 2: Understand the Idea

Use the **AskQuestion tool** to ask questions **one at a time**.

**Guidelines:**
- Prefer multiple choice options when natural alternatives exist
- Start broad (purpose, users) then narrow (constraints, edge cases)
- Only one question per message
- Focus on: purpose, goals, constraints, success criteria
- Validate assumptions explicitly

**Questions to explore:**
- What problem are we solving? Why does it matter?
- Who is this for? What's their current experience?
- What does success look like from the user's perspective?
- What are the must-haves vs nice-to-haves?
- What's explicitly out of scope?
- Are there constraints (timeline, budget, technical limitations)?
- Are there existing patterns or conventions we should follow?

**Exit condition:** Continue until the idea is clear OR user says "proceed."

### Phase 3: Explore Approaches

Propose **2-3 concrete approaches** based on repo research and conversation.

For each approach, provide:
- Brief description (2-3 sentences)
- Pros and cons (including light technical framing like "would require a new API endpoint" or "could extend the existing system")
- When it's best suited

Lead with your recommendation and explain why. Apply YAGNI — prefer simpler solutions.

Use **AskQuestion tool** to ask which approach the user prefers.

### Phase 4: Present the Requirements

- Present requirements once the idea and approach are understood
- Break into sections of 200-300 words
- Use **AskQuestion tool** after each section to check whether it looks right
- Be ready to go back and clarify if something doesn't fit

## Saving the Spec

**If working with a project folder** (from Initial Setup):
- Save to `~/.cursor/project-specs/[folder]/spec.md`

**If no project folder**, use **AskQuestion tool** to ask:
- Create new folder (provide ticket + feature name)
- Use existing folder: [list recent without spec.md]
- Skip saving

**Creating new project folder:**
- Format: `YYYY-MM-DD-[ticket]-[feature-name]` (e.g., `2025-01-31-ENG-123-user-auth`)
- Path: `~/.cursor/project-specs/[folder]/spec.md`

**spec.md format:**

```markdown
# Spec: [Topic]

**Project:** [folder-name]
**Created:** YYYY-MM-DD

## Problem Statement
[What problem we're solving and why it matters]

## Goals
[What we're trying to achieve — user-facing outcomes]

## Approach
[Chosen approach and rationale — why this over the alternatives]

## Requirements

### Must Have
- [Critical requirements — the feature doesn't work without these]

### Should Have
- [Important but flexible — can adjust approach]

### Won't Have (Out of Scope)
- [Explicitly excluded to prevent scope creep]

## Success Criteria
[How we'll know we're done — from the user's perspective]

## Constraints
[Timeline, budget, technical limitations, dependencies, etc.]

## Open Questions
[Things we need to validate during codebase research]

## Next Steps
- [ ] Research codebase: `/workflows:research [folder-name]`
```

**Git commits:** No commit (project-specs directory is outside project repos).

**After saving**, suggest the user clear context with `/clear`, then use **AskQuestion tool** to present next steps:
- Proceed to research: `/workflows:research [folder-name]`
- Refine spec further
- Done for now

## Key Principles

- **One question at a time** — Don't overwhelm with multiple questions
- **Always use AskQuestion tool** — Every question to the user goes through the tool
- **Multiple choice preferred** — Easier to answer than open-ended when possible
- **Stay in your lane** — Product-level approaches yes, implementation architecture no
- **YAGNI ruthlessly** — Push back on unnecessary requirements, prefer simpler approaches
- **Incremental validation** — Present in sections, validate each
- **Be flexible** — Go back and clarify when something doesn't make sense
- **Capture uncertainty** — Open questions go in the spec for research to answer
