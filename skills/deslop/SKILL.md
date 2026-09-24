---
name: deslop
description: >-
  Remove AI code slop from the current branch's diff. Use when the user says
  "deslop", "remove slop", "clean up AI slop", "remove AI comments", or wants to
  strip out unnecessary AI-generated code patterns from their changes.
---
# Remove AI code slop

Check the diff against main, and remove all AI generated slop introduced in this branch.

This includes:

- Extra comments that a human wouldn't add or is inconsistent with the rest of the file
- Extra defensive checks or try/catch blocks that are abnormal for that area of the codebase (especially if called by trusted / validated codepaths)
- Casts to any to get around type issues
- Redundant intermediate variables (`const result = foo(); return result;`) and unnecessary else after return/throw
- Duplicated logic blocks that should be consolidated into a loop or shared helper
- Overly verbose tests — duplicate setup that belongs in beforeEach, redundant assertions that test the same thing
- Any other style that is inconsistent with the file

Report at the end with only a 1-3 sentence summary of what you changed
