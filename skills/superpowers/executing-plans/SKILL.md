---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Check the current Codex tool set. If native subagents are available
and the plan has independent tasks, use subagent-driven-development. Otherwise
execute locally. Do not create user-owned app tasks as a substitute for subagents.

## The Process

### Step 1: Load and Review Plan
1. Use using-git-worktrees to check the workspace and add isolation when needed; respect an existing choice to work in place
2. Read plan file
3. Review critically - identify any questions or concerns about the plan
4. Resolve routine concerns from the spec and repository; ask only for missing decisions that materially block safe progress
5. If no concerns: Create todos for the plan items and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use finishing-a-development-branch
- Follow that skill to verify tests and complete any already-authorized integration; otherwise report the completed work

## When to Stop and Ask for Help

**Investigate blockers and test failures before asking.** Fix issues within
scope and resolve routine implementation choices from the spec and repository.
Ask when a required decision is missing, the plan has critical gaps you cannot
resolve, or an action needs authorization not already given. Keep independent
authorized work moving. Do not guess at a material requirement.

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Do not hide unresolved blockers.** Report what remains and why when user input is necessary.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Investigate blockers, and ask only when necessary
- Respect the user's chosen checkout and existing authorization
