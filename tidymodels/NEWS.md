# Tidymodels Skills - News

- **Architecture: Single Source of Truth & Code Duplication Remediation** (2026-03-19)
  - **BREAKING**: Removed all code blocks from SKILL.md files to eliminate duplication
  - SKILL.md is now purely navigational (overview + links to references)
  - Removed duplicated package setup code from extension-guide.md files (both recipe and yardstick)
  - Setup code now exists only in r-package-setup.md (single source of truth)
  - Removed duplicate MAE implementation from add-yardstick-metric SKILL.md
  - Complete MAE example now lives only in numeric-metrics.md reference
  - Prevents users from following incomplete/stale setup instructions
  - Prevents inconsistency where users follow abbreviated "Quick setup" and miss critical steps
  - Documented as #1 anti-pattern in SKILL_IMPLEMENTATION_GUIDE.md
  - Makes skills maintainable: one place to update, no synchronization needed
  - Part of systematic review to eliminate all code duplication across tidymodels skills

- **Claude Code Integration** (2026-03-19)
  - Added support for `usethis::use_claude_code()` in extension setup workflow
  - Claude uses `AskUserQuestion` to prompt users to read `.claude/CLAUDE.md` after setup
  - Seamless integration: Claude reads CLAUDE.md and incorporates tidyverse patterns automatically
  - Skill composition: tidymodels-dev skills combine with tidy-* skills for complete R package guidance
  - Graceful fallback: skills work perfectly without usethis dev version
  - Clear instructions in r-package-setup.md, extension guides, and SKILL_IMPLEMENTATION_GUIDE.md

- **Extension vs Source Development Architecture** (2026-03-18)
  - Skills now support two distinct development contexts
  - Extension development: Creating new packages using only exported functions
  - Source development: Contributing to tidymodels repos with access to internal functions
  - Automatic context detection based on DESCRIPTION file
  - Dedicated guidance for each context to avoid overwhelming developers

- **Repository Access Enhancement** (2026-03-17)
  - Optional repository cloning for direct access to canonical implementations
  - Platform-agnostic scripts (bash, PowerShell, Python) for one-command cloning
  - File path references added to skill documentation
  - Entirely optional—skills work with built-in references if preferred
