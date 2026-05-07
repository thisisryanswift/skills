---
name: ryan-blog-style
description: Draft and edit Ryan-style developer blog posts from long technical sessions, especially for DEV.to. Use when turning coding sessions, tool explorations, or messy implementation notes into concise public posts.
---

# Ryan Blog Style

Use this skill when helping Ryan turn a long technical session into a short, readable developer blog post.

## Core Goal

Do not summarize the whole session. Find the main lesson and write around that.

The ideal post feels like:

- a working developer sharing what they tried
- practical, honest, and mildly funny
- short enough that someone will actually finish it
- focused on the top learning, not every command or detour

## Voice

Write in first person.

Use a conversational but direct tone. It should sound like Ryan explaining the thing to another developer, not like corporate content marketing.

Good traits:

- plainspoken
- lightly self-deprecating
- pragmatic
- curious
- honest about rough edges
- not trying too hard to sound profound

Avoid:

- exhaustive recaps
- polished SaaS-blog phrasing
- hype language
- overexplaining the background
- long lists of every tool, command, or debugging step
- turning the post into documentation

## Structure

Prefer a compact structure:

1. Personal setup or problem
2. Why it became annoying
3. What was tried
4. What worked and what was rough
5. The main takeaway

Keep sections short. Two to four headings is usually enough.

## Hooks

Start with the human problem before the tool.

Good hook pattern:

```text
I use a lot of AI coding tools.

<short list>

That sounds fun until every tool wants its own config.
```

Avoid opening with:

- product background
- installation steps
- a complete timeline
- definitions unless absolutely necessary

## Sentence Style

Use short paragraphs.

It is okay to use sentence fragments for rhythm.

Example:

```text
Its own MCP setup.

Its own skills folder.

Its own auth story.

Its own idea of what “installed” means.
```

Use one memorable line if possible. Do not force several.

Examples of the right shape:

```text
AI tool sprawl is the new dotfile hell.
```

```text
ToolHive did not make the ecosystem simple. It made it governable.
```

## Humor

Use small, specific jokes.

Good:

```text
It started feeling like I was torturing a tiny fleet of confused robots.
```

Avoid jokes that derail the post or require too much setup.

## Technical Detail

Include only enough technical detail to make the learning credible.

Prefer:

- the key tool names
- the important abstraction
- one or two concrete examples
- screenshots if available

Avoid:

- every command run
- every failed attempt
- every MCP installed
- full architecture unless the post is specifically about architecture

If the session was long, compress aggressively.

## How To Use Source Material

When given a long session history:

1. Identify the actual problem Ryan cared about.
2. Identify the one reusable lesson.
3. Pick only the moments that support that lesson.
4. Drop everything else, even if it was technically interesting.

For the ToolHive post, the winning focus was:

```text
I use many AI coding tools for my job. Keeping MCPs, skills, auth, and config consistent across them became painful. ToolHive helped turn that chaos into something governable.
```

The losing focus was:

```text
Here is everything that happened in a 383-message coding session.
```

## Editing Guidance

When reviewing Ryan's drafts:

- keep the post short if it is already working
- suggest small grammar and rhythm fixes
- do not rewrite unless asked
- preserve Ryan's casual wording when it works
- flag sentences that sound too braggy, too vague, or too corporate

Prefer comments like:

```text
This is publishable. I’d only fix these three small things.
```

Do not respond with a full replacement draft unless the draft is structurally broken or Ryan asks for one.

## DEV.to Fit

DEV posts should be readable on a phone.

Favor:

- short paragraphs
- clear headings
- one simple takeaway
- one question or conversational close if it fits

Do not over-optimize for SEO or make the post sound like a launch announcement.

## Checklist

Before finalizing a draft, check:

- Is the main lesson obvious?
- Could the post be 20% shorter?
- Does it start with Ryan's problem, not the product?
- Is there one memorable line?
- Are technical details supporting the point rather than taking over?
- Does it sound like a real person tried something today?
