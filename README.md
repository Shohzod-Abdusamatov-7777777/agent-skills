# Claude Skills

A collection of specialized skills for Claude and AI agents. Skills provide domain-specific expertise and context that AI assistants can load on-demand to deliver more accurate, consistent, and high-quality responses.

## What are Skills?

Skills are structured knowledge modules that extend AI capabilities in specific domains. Each skill contains:

- **Role definition** - Expert persona and background
- **Triggers** - Keywords/contexts that activate the skill
- **Constraints** - Must-do and must-not-do rules
- **Reference guides** - Detailed documentation loaded on-demand
- **Output templates** - Consistent response formats

## Available Skills

| Skill | Description |
|-------|-------------|
| [vue-expert](./vue-expert/SKILL.md) | Vue 3, Composition API, Nuxt 3, Pinia, Quasar, Capacitor, PWA, Vite |
| [vue-fsd](./vue-fsd/SKILL.md) | Feature-Sliced Design architecture for scalable Vue 3 applications |
| [tailwind-v4](./tailwind-v4/SKILL.md) | Tailwind CSS v4 with @theme directive, OKLCH colors, Vite integration |

## Usage

### With Claude Code

Skills can be automatically activated based on triggers or explicitly invoked when working on relevant projects.

### Skill Structure

Each skill follows this structure:

```
skill-name/
├── SKILL.md           # Main skill definition
└── references/        # Detailed reference guides
    ├── topic-1.md
    ├── topic-2.md
    └── ...
```

## Creating New Skills

1. Create a new directory with your skill name
2. Add a `SKILL.md` file with frontmatter and skill definition
3. Add reference files for detailed knowledge areas
4. Update this README with the new skill

### Skill Template

```yaml
---
name: skill-name
description: Brief description of when to use this skill
triggers:
  - keyword1
  - keyword2
role: specialist
scope: implementation
output-format: code
---

# Skill Name

[Role definition and expertise areas]

## When to Use This Skill

[List of use cases]

## Core Workflow

[Step-by-step approach]

## Reference Guide

[Table of reference files and when to load them]

## Constraints

### MUST DO
- [Required behaviors]

### MUST NOT DO
- [Prohibited behaviors]
```

## Contributing

Contributions are welcome! Please ensure your skills follow the established structure and include comprehensive reference documentation.

## License

MIT
