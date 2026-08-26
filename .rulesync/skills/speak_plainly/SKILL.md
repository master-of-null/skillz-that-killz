---
name: speak-plainly
description: Apply clear, precise, plain-language writing rules to explanations, technical writing, documentation, and other written responses.
---

# Speak plainly

1. Noun consistency: Never refer to the same thing in different ways. Always use the same term, no matter how verbose, for the same thing. Repeat words.
2. Words have meaning. Never use a word poetically. Avoid coining original terms; instead prefer describing what is being referred to. Use boring words.
3. Never use a complex word when a simple word will do. eg. "subsumes", "axes of the entity type", "creation primitive", "creation-action descriptor". Using plain words.
4. Avoid vague words like: "connected", "relationship", "linked", "mounts", "feeds". Those words do not have a precise meaning. Words like those, used without clear meaning or relevant context, obscure your explanations. Use precise words.

    Exceptions exist, for example: "mount" could have a specific meaning in the context of React.
5. Avoid using abbreviations without first using the full term at least once, previously.

    Exceptions exist: You do not need to define very common abbreviations like HTTP, ADR, API, REST, ACL, etc. Use them without first defining them.
6. Before a novel technical term is used, make sure that it has been defined. Definitions should attempt to get down to the concrete domain role of a term. Definitions should include what type of thing the term refers to: Is it a state of a system or object, a database model, a React component, etc.

- Only define terms specific to the project domain. Do not define technical terms that are well-known.
- You should also skip terms that are very common in the context of the discussion.

7. Avoid referring to paths not taken. Focus on what was done, not what wasn't done.
8. Avoid referencing previous implementations. Prefer using declarative present-tense sentences. Examples: Code comments, responses, most technical documents, etc.

    Exception exist: Changelogs or commit messages
