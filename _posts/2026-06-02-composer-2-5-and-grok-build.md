---
layout: post
title: "composer 2.5 and grok build"
date: 2026-06-02 09:00:00 -0500
categories: update
---

Two notable agentic coding tools landed in beta in mid-May: Cursor's Composer 2.5 and xAI's Grok Build.

Cursor released [Composer 2.5](https://cursor.com/blog/composer-2-5) on May 18. It is their most capable model yet, with gains in sustained performance on long-running tasks and reliability following complex instructions. The model starts from Moonshot's open Kimi K2.5 checkpoint and receives further continued pretraining plus reinforcement learning. Key techniques include 25x more synthetic tasks grounded in real codebases and targeted textual feedback during RL to improve credit assignment and behavioral traits such as communication style and effort calibration.

Benchmarks show it closing in on or matching top frontier models on several coding agent evaluations while carrying a dramatically lower price: $0.50 per million input tokens and $2.50 per million output for the standard tier (with a faster variant at the same intelligence level). The launch included doubled included usage for the first week.

The response on X and forums has been enthusiastic. Developers point to strong iteration speed, particularly for UI and design-heavy work, and a feedback loop that feels meaningfully tighter than previous versions or some alternatives. Several people who had stepped away from Cursor are testing it again seriously during the promotional window.

xAI shipped [Grok Build](https://x.ai/cli) around the same period as a terminal-native coding agent. It offers an interactive TUI with mouse support and fullscreen experience, plus headless operation for scripts. Standout workflow features include explicit plan mode (with structured plans, step approvals, and clean diffs), parallel subagents that can research or build in separate contexts or worktrees, skills (including AGENTS.md conventions, auto-captured session skills via /skillify, and plugins), hooks, MCP server integrations for external tools and services, git operations, terminal execution with streaming, code search, and sandboxed runs.

The underlying grok-build-0.1 model is tuned for agentic software engineering and tool use; the same model is now exposed via the xAI API in early access. Early hands-on reports praise the TUI as best-in-class for this category of tool. Model capability and long unattended execution have drawn more measured feedback in the initial beta, with the team acknowledging the early state and signaling fast iteration ahead. Elon Musk replied directly to one detailed review noting "rapid improvement coming."

A small but telling detail: Composer 2.5 is already selectable as a model inside the Grok Build TUI.

These releases sit in a broader surge of specialized coding agents. One emphasizes deep integration inside an editor with its own model line. The other prioritizes a powerful, extensible harness that lives in the terminal and plays well with existing workflows and external services. Both invest heavily in the "agentic" layer beyond raw model intelligence: planning, decomposition, verification loops, extensibility, and memory of project conventions.

The practical effect is that certain classes of work—large refactors, language migrations, exploratory implementation, and maintaining momentum across multi-step changes—become more tractable for smaller teams or solo builders. The cost curve is also bending sharply downward for high-quality agent time.

It is still early. Session length, consistency on production-grade changes, and integration into real team processes all have room to improve. But the velocity of releases, the specific engineering focus on long-horizon reliability and workflow primitives, and the quick sharing of capabilities (models crossing tools) suggest the gap between current agents and reliable daily drivers is narrowing quickly.

The tooling is moving faster than most organizations can update their habits. That is the real story behind the buzz.