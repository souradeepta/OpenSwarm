# Repository Handoff

**Repository:** OpenSwarm
**Branch:** main
**Date:** 2026-09-12

## Overview

OpenSwarm is a fully open-source, production-ready multi-agent system built on Agency Swarm that coordinates 8 specialized AI agents to produce complete deliverables (slide decks, research reports, documents, visualizations, images, and videos) from a single terminal prompt.

## Current Status

No known work-in-progress as of 2026-09-12.

## Key Entry Points

- `swarm.py` — Agency definition, communication flows, TUI entry point
- `server.py` — FastAPI REST API wrapper for deployment
- `DEVELOPER_GUIDE.md` — Comprehensive guide for developers covering installation, deployment patterns, customization, and use cases
- `.cursor/rules/agency-swarm-workflow.mdc` — Workflow rules for AI-assisted agency building

## Next Steps

1. Review the DEVELOPER_GUIDE.md for setup and customization options
2. Choose a deployment pattern: TUI, API server, or Docker
3. Configure API keys in `.env` file
4. Test with sample prompts to verify agent coordination

## Communication Rules

For any additions or modifications to this repository, update this handoff file and reference it in commit messages.
