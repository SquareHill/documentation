# Documentation Review Checklist

Use this checklist to review and streamline documentation files.

## Content Quality

- [ ] **Direct & Concise**: Gets to the point without unnecessary preamble
- [ ] **No List Overload**: Maximum 1-2 list sections per page; use prose where possible
- [ ] **User-Focused**: Focuses on "how to use" not "how it works internally"
- [ ] **Match UI**: Button names, workflows, and field names match actual application
- [ ] **Scannable**: Headers and structure allow quick navigation
- [ ] **Complete**: Covers essential workflows without over-explaining

## Lists & Formatting

- [ ] **Minimal Lists**: Avoid consecutive sections of lists (steps → bullets → numbered)
- [ ] **Short Lists**: Keep lists to 3-5 items maximum
- [ ] **No Nested Lists**: Avoid bullets within bullets within numbered steps
- [ ] **Prose Over Lists**: Use flowing paragraphs for explanations, lists only for actual steps or options
- [ ] **Appropriate Use**: Use lists only for: sequential steps, discrete options, or key points

## Structure Issues

- [ ] **No Redundancy**: Remove sections that repeat information
- [ ] **Clear Headers**: Each section has clear purpose, no vague titles
- [ ] **Logical Flow**: Sections follow natural user journey
- [ ] **No Fluff**: Remove "Key Benefits", "Why This Matters" sections unless critical
- [ ] **Right Level**: Not too basic, not too technical for audience

## Examples & Code

- [ ] **Relevant Examples**: Examples match common use cases
- [ ] **Concise Code**: Code blocks show essentials, not entire files
- [ ] **Good vs Avoid**: Use comparison only when clarifying confusion
- [ ] **Real Values**: Use realistic examples, not "test123" or "example1"

## Common Issues to Fix

- [ ] **Remove**: "In this section" / "Let's explore" / "Now let's" introductions
- [ ] **Remove**: Obvious statements (e.g., "You can edit tools after creating them")
- [ ] **Condense**: Multi-step processes with sub-bullets (flatten or prose)
- [ ] **Condense**: "What happens when..." explanation sections (assume smart users)
- [ ] **Condense**: Troubleshooting sections with 5+ bullet solutions per error

## Files to Review

### Concepts

- [ ] `concepts/agents.mdx`
- [ ] `concepts/datasets.mdx`
- [ ] `concepts/pages.mdx`
- [ ] `concepts/record-detail-page.mdx`
- [ ] `concepts/tools.mdx`
- [ ] `concepts/workspaces.mdx`

### Agents Guides

- [ ] `guides/agents/chatting-with-agents.mdx`
- [ ] `guides/agents/configuring-models.mdx`
- [ ] `guides/agents/creating-agents.mdx`
- [ ] `guides/agents/managing-tools.mdx`
- [ ] `guides/agents/testing-agents.mdx`

### Datasets Guides

- [ ] `guides/datasets/creating-datasets.mdx`
- [ ] `guides/datasets/importing-data.mdx`
- [ ] `guides/datasets/managing-columns.mdx`

### Tools Guides

- [ ] `guides/tools/rest-api.mdx`
- [ ] `guides/tools/mcp-tools.mdx` ✅ **COMPLETED**

### Pages Guides

- [ ] Review all pages guides (check file_search for exact list)

### Variables Guides

- [ ] Review all variables guides

### Workspaces Guides

- [ ] Review all workspaces guides

## Review Process

For each file:

1. **Read full file** - Understand content and flow
2. **Identify issues** - Mark list-heavy sections, redundancy, fluff
3. **Rewrite** - Convert lists to prose, remove unnecessary content
4. **Verify** - Ensure completeness while being concise
5. **Check UI** - Confirm button names and workflows match application

## Target Metrics

- **Page Length**: Aim for 80-150 lines (exceptions for comprehensive guides)
- **List Sections**: Maximum 2 per page
- **List Items**: 3-5 items per list
- **Intro Paragraphs**: 2-3 sentences maximum
- **Code Examples**: 1-3 per page, focused on essentials
