---
title: "Documentation"
excerpt: "Notes on writing clean documentation."
categories:
  - Notes
---

# Introduction

My quick reference notes on writing documentation.

# Words

- Define new or unfamiliar terms.
- Use terms consistently.
  - Don't have multiple names for the same thing.
- Define an acronym on its first use.
  - Don't cycle between the full term and the acronym.
  - Don't define an acronym if it's only going to be used a few times.
- Disambiguate pronouns.
  - Only use a pronoun after introducing the noun.
  - If the pronoun is more than a few words away from the noun then repeat the noun.
  - If you introduce a second noun between your noun and your pronoun, reuse your noun instead of using a pronoun.

# Active Voice

- Active voice: actor + verb + target.
- Passive voice: target + verb + (optional) actor.
- Prefer using active voice.
  - Bad: The c++ source code is compiled by clang.
  - Good: Clang compiles the c++ source code.

# Clear Sentences

- Reduce imprecise, weak and generic verbs.
  - Bad: This error _happens_ when ...
  - Good: The system _generates_ this error when ...
- Omit phrases '_there is_' and '_there are_'.
  - Bad: There is a variable called '`last_dwell`' that stores the last dwell which was calculated.
  - Good: The variable '`last_dwell`' stores the last dwell which was calculated.
- Put conditional clauses before instructions (not after).
  - Bad: Call `updateConfluence()` if you want to update the component's confluence page.
  - Good: To update the component's confluence page, call `updateConfluence()`.
- Use second person narrative ("_you_" rather than "_we_").
  - Bad: If we're using pub/sub ...
  - Good: If you're using pub/sub ...

# Short Sentences

- Focus each sentence on a single idea.
- Convert long sentences to lists.
- Eliminate extraneous words.
  - Bad: Calling the `registerSelf` method causes the triggering of the RegisterTaskTopic.
  - Good: The `registerSelf` method triggers the RegisterTaskTopic.
- Subordinate clauses should extend the sentence and not branch off into a separate idea.
  - 'That' introduces an essential subordinate clause.
  - 'Which' introduces a non-essential subordinate clause.

# Lists

- Use bulleted lists for unordered items.
- Use numbered lists for ordered items.
- Reduce embedded lists (also known as run-in lists).
- Keep list items parallel (same grammar, logical category, capitalization & punctuation).
- Start numbered lists with imperative verbs.

# Tables

- Label each column with a meaningful header.
- Avoid putting too much text into a table cell.
- Strive for parallelism within individual columns (same grammar, logical category, capitalization & punctuation).

# Paragraphs

- The opening sentence should establish the paragraph's central point.
- Focus each paragraph on a single topic.
- Paragraphs should contain around 3 - 7 sentences.

# Audience

- Define your audience.
- Determine what your audience needs to learn.
- Good documentation `=` "knowledge needed to do a task" `-` "the audience's current knowledge".

# Longer Documents

- State your document's scope.
- You can also define its non-scope.
- Write an executive summary (a TL;DR).

# Formatting

- Put code-related text in code font.
- Prefer SVG files and PNG files.
- Use descriptive link text.
  - Bad: Want more? <a href="/moreInfo">Click here!</a>
  - Good: For more information, visit <a href="/moreInfo">the Volume Search landing page</a>.

# Editing

- Think like your audience.
- Read the documentation aloud.
- Come back to it later.
- Ask a peer to review it.
- Run a spell checker on the document.

# Illustrations

- Write the caption before creating the illustration to ensure the illustration matches the goal.
- Constrain the amount of information in a single drawing.
- Don't put more than 1 paragraph's worth of information in a single diagram.
- Focus the reader's attention.
  - Use arrows or circles to highlight the important part in screenshots.

# Sample Code

- Sample code should set the best way to interact with your API.
- Maintain the sample code.
  - Over time the system may change and so will the sample code.
  - Test the sample code actually works.
- Example the anti-example.
  - In addition to showing readers what to do, it is sometimes beneficial to show readers what not to do.
