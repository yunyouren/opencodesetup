---
name: demo-skill
description: Demonstrate a Claude Code skill that should be mapped to an OpenCode skill.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob
---

# Demo Skill

Read the requested file and summarize its structure. This fixture intentionally uses Claude-only frontmatter fields so the recommender can flag their OpenCode equivalents or gaps.
