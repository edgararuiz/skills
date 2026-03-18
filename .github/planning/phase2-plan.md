# Phase 2 Plan: Implementation Strategy

**Date:** 2026-03-18
**Status:** Planning
**Previous Phase:** Phase 1 Assessment Complete

## Decisions from Phase 1 Review

### ✅ Confirmed Decisions

1. **Architecture:** Two-Skill Structure (not three)
2. **Auto-detection:** Don't ask user, detect context automatically
3. **File splitting:** Split shared-references into extension/source variants (6 files total)
4. **Side-by-side comparisons:** Not needed, keep seamless
5. **Git/PR workflow:** Minimal guidance, low priority
6. **Template PRs:** Not at this time
7. **Package-specific guides:** Absolutely needed for yardstick vs recipes differences

---

## Auto-Detection Logic

### Implementation in Main SKILL.md

```markdown
# Add Yardstick Metric

## Context Detection

This skill automatically detects your development context:

- ✅ **Source Development**: Working in `yardstick/` repository → Use internal functions, package test helpers
- ✅ **Extension Development**: Working in other package or project → Avoid internals, use exported functions only

**Detection logic:**
1. If current directory contains `yardstick` source (checked via `.git` + `DESCRIPTION` name) → SOURCE
2. If in a different R package (has `DESCRIPTION`) → EXTENSION
3. If in non-package project (has content but no `DESCRIPTION`) → EXTENSION
4. If empty directory → Ask user preference

Current context: [DETECTED_CONTEXT]
```

### Technical Implementation

The skill should check:
1. Is there a `.git` folder?
2. Is there a `DESCRIPTION` file?
3. Does `DESCRIPTION` say `Package: yardstick` (or `Package: recipes`)?

**Decision tree:**
```
Check DESCRIPTION file
  ├─ Exists?
  │   ├─ Package: yardstick → SOURCE
  │   ├─ Package: recipes → SOURCE
  │   └─ Other package name → EXTENSION
  └─ Not exists?
      ├─ Has files? → EXTENSION (assume new extension)
      └─ Empty? → ASK USER
```

---

## File Structure Changes

### New Structure

```
tidymodels/skills/
├── add-yardstick-metric/
│   ├── SKILL.md                              # Main entry point with auto-detection
│   ├── extension-guide.md                    # NEW: Extension-specific deep dive
│   ├── source-guide.md                       # NEW: Source-specific deep dive
│   └── references/                           # Mostly unchanged
│       ├── metric-system.md
│       ├── numeric-metrics.md
│       ├── ... (other metric types)
│       └── ... (12 more files, minor updates)
│
├── add-recipe-step/
│   ├── SKILL.md                              # Main entry point with auto-detection
│   ├── extension-guide.md                    # NEW: Extension-specific deep dive
│   ├── source-guide.md                       # NEW: Source-specific deep dive
│   └── references/                           # Mostly unchanged
│       ├── step-architecture.md
│       ├── ... (other step types)
│       └── ... (5 more files, minor updates)
│
└── shared-references/
    ├── r-package-setup.md                    # Unchanged
    ├── development-workflow.md               # Minor updates (minimal git guidance)
    ├── roxygen-documentation.md              # Unchanged
    ├── repository-access.md                  # Unchanged
    ├── testing-patterns-extension.md         # NEW: Split from testing-patterns.md
    ├── testing-patterns-source.md            # NEW: Split from testing-patterns.md
    ├── best-practices-extension.md           # NEW: Split from best-practices.md
    ├── best-practices-source.md              # NEW: Split from best-practices.md
    ├── troubleshooting-extension.md          # NEW: Split from troubleshooting.md
    └── troubleshooting-source.md             # NEW: Split from troubleshooting.md
```

### Files to Create (New)

1. `add-yardstick-metric/extension-guide.md`
2. `add-yardstick-metric/source-guide.md`
3. `add-recipe-step/extension-guide.md`
4. `add-recipe-step/source-guide.md`
5. `shared-references/testing-patterns-extension.md`
6. `shared-references/testing-patterns-source.md`
7. `shared-references/best-practices-extension.md`
8. `shared-references/best-practices-source.md`
9. `shared-references/troubleshooting-extension.md`
10. `shared-references/troubleshooting-source.md`

### Files to Modify

1. `add-yardstick-metric/SKILL.md` - Add auto-detection, split sections
2. `add-recipe-step/SKILL.md` - Add auto-detection, split sections
3. `shared-references/development-workflow.md` - Add minimal git guidance

### Files to Archive/Remove

1. `shared-references/testing-patterns.md` → Split into extension/source
2. `shared-references/best-practices.md` → Split into extension/source
3. `shared-references/troubleshooting.md` → Split into extension/source

---

## Package-Specific Considerations

### Yardstick-Specific Patterns

**Source Development Differences:**
- Internal test helpers: `yardstick:::data_altman()`, `yardstick:::data_three_class()`
- Internal functions: `yardstick:::yardstick_mean()`, `yardstick:::finalize_estimator_internal()`
- Snapshot testing: Heavy use of `testthat::expect_snapshot()`
- Documentation: Uses `@templateVar` and `@template` extensively
- File organization: Metrics grouped by type (num-, class-, prob-, surv-)

**Extension Development Differences:**
- Must use exported functions only
- Create own test data
- Standard testthat patterns
- Standard roxygen2 without templates
- Any file organization works

### Recipes-Specific Patterns

**Source Development Differences:**
- Internal helpers: `recipes:::get_case_weights()`, `recipes:::recipes_eval_select()`
- Internal test data: `recipes:::iris_rec`, example data in package
- Step naming: Must follow `step_*` convention exactly
- Documentation: Uses `@inheritParams` heavily, references other steps
- File organization: Steps loosely grouped by type

**Extension Development Differences:**
- Must use exported recipes functions
- Use standard datasets (mtcars, iris)
- Step naming flexible
- Standard roxygen2
- Any file organization works

---

## Detailed Implementation Plan

### Stage 1: Create Split Reference Files (3-4 hours)

#### 1.1 Split testing-patterns.md

**testing-patterns-extension.md:**
- Focus on creating test data from scratch
- Avoid any internal functions
- Use standard datasets (mtcars, iris, modeldata)
- Basic testthat patterns
- Standard test categories: correctness, NA handling, validation, case weights

**testing-patterns-source.md:**
- CAN use internal test helpers
- Reference existing test files in package
- Include snapshot testing patterns
- More comprehensive edge cases to match package standards
- Package-specific test structure

**Key Differences to Highlight:**
```markdown
## When to Use Internal Test Helpers

### Extension Development
❌ **Don't use:** `yardstick:::data_altman()` - Not exported
✅ **Do use:** Create your own test data

### Source Development
✅ **Can use:** `yardstick:::data_altman()` - You're developing yardstick
✅ **Can use:** Any internal function for testing
```

#### 1.2 Split best-practices.md

**best-practices-extension.md:**
- Focus on clean, self-contained code
- Emphasize exported function usage
- Document when functionality is missing
- Use base R alternatives

**best-practices-source.md:**
- Internal functions are acceptable when appropriate
- Check if internal helper already exists before reimplementing
- Follow package conventions exactly
- Match existing code style in package

**Key Differences to Highlight:**
```markdown
## Using Internal Functions

### Extension Development
**Never use internal functions.** They are:
- Not guaranteed to be stable
- Not documented
- May change without notice
- Will cause CRAN check failures

Use base R alternatives or exported functions.

### Source Development
**Internal functions are part of the package.** You can:
- Use existing internal helpers
- Create new internal helpers when needed
- Refactor internals (with care)
- Document internals with roxygen but don't export

**When to use:**
- ✅ Shared logic between multiple functions
- ✅ Complex calculations used repeatedly
- ✅ Helper functions for testing
- ❌ Don't overuse - sometimes duplication is clearer
```

#### 1.3 Split troubleshooting.md

**troubleshooting-extension.md:**
- Focus on namespace/import issues
- Package setup problems
- Dependency management
- External function usage

**troubleshooting-source.md:**
- Working with package internals
- Git/branch issues (minimal)
- Integration testing
- Package-specific checks
- PR submission issues (minimal)

### Stage 2: Create Extension/Source Guides (4-5 hours)

#### 2.1 Create extension-guide.md (yardstick)

**Content:**
```markdown
# Extension Development Guide: Yardstick Metrics

Complete guide for creating new packages that extend yardstick.

## When to Use This Guide

✅ You are creating a NEW package that adds metrics
✅ You want to build on yardstick's foundation
✅ You're NOT submitting a PR to yardstick itself

## Prerequisites

[Package setup section]

## Step-by-Step Implementation

[Detailed walkthrough with example]

## Key Constraints

- ❌ Never use `:::` to access internal functions
- ✅ Only use exported yardstick functions
- ✅ Create self-contained implementations
- ✅ Use base R alternatives when needed

## Complete Example: Custom MAE Variant

[Full working example]

## Testing

→ See [Testing Patterns (Extension)](../shared-references/testing-patterns-extension.md)

## Best Practices

→ See [Best Practices (Extension)](../shared-references/best-practices-extension.md)

## Troubleshooting

→ See [Troubleshooting (Extension)](../shared-references/troubleshooting-extension.md)
```

#### 2.2 Create source-guide.md (yardstick)

**Content:**
```markdown
# Source Development Guide: Contributing to Yardstick

Complete guide for contributing new metrics to the yardstick package itself.

## When to Use This Guide

✅ You are contributing a PR to yardstick
✅ You have cloned the yardstick repository
✅ You want your metric included in yardstick

## Prerequisites

[Repository setup section]

## Understanding Yardstick's Architecture

[Package-specific architecture notes]

## Step-by-Step Implementation

[Detailed walkthrough with example]

## Working with Internal Functions

- ✅ CAN use internal helpers (you're developing the package)
- ✅ CAN create new internal helpers when appropriate
- ✅ Check if helper already exists: `ls("package:yardstick", all.names = TRUE)`
- ⚠️ Internal functions can change - document why you're using them

### Common Internal Helpers

```r
# Mean calculation with case weights
yardstick:::yardstick_mean(values, case_weights)

# Finalize estimator for multiclass
yardstick:::finalize_estimator_internal(metric_class, call = rlang::caller_env())
```

## File Naming Conventions

- Numeric metrics: `R/num-[name].R`, `tests/testthat/test-num-[name].R`
- Class metrics: `R/class-[name].R`, `tests/testthat/test-class-[name].R`
- Probability metrics: `R/prob-[name].R`, `tests/testthat/test-prob-[name].R`

## Documentation Requirements

[Package-specific documentation standards]

## Testing with Internal Helpers

```r
# You CAN use internal test data
data <- yardstick:::data_altman()
data <- yardstick:::data_three_class()

# You CAN use internal test functions
check_result <- yardstick:::check_metric(...)
```

## Complete Example: Adding a Metric to Yardstick

[Full working example matching yardstick style]

## Testing

→ See [Testing Patterns (Source)](../shared-references/testing-patterns-source.md)

## Best Practices

→ See [Best Practices (Source)](../shared-references/best-practices-source.md)

## Troubleshooting

→ See [Troubleshooting (Source)](../shared-references/troubleshooting-source.md)

## PR Submission (Minimal)

1. Create branch: `git checkout -b feature/add-metric-name`
2. Commit changes: `git commit -m "Add [metric_name] metric"`
3. Push and create PR
4. Tidymodels team will review
```

#### 2.3 Create extension-guide.md (recipes)

Similar structure to yardstick extension guide but for recipe steps.

#### 2.4 Create source-guide.md (recipes)

Similar structure to yardstick source guide but with recipes-specific patterns:
- Internal helpers: `recipes:::get_case_weights()`, `recipes:::check_type()`
- File naming: `R/[step_name].R`
- Documentation: Heavy use of `@inheritParams`

### Stage 3: Update Main SKILL.md Files (3-4 hours)

#### 3.1 Update add-yardstick-metric/SKILL.md

**Changes:**

1. **Add context detection section** (top of file, ~50 lines)
2. **Split "Overview" section** into context-aware guidance
3. **Update "Prerequisites" section** to redirect based on context:
   - Extension → Points to extension-guide.md
   - Source → Points to source-guide.md
4. **Keep common sections** (Choosing Metric Type, Architecture, etc.)
5. **Add navigation** to extension/source guides
6. **Update examples** to note which context they apply to

**Structure:**
```markdown
# Add Yardstick Metric

## Context Detection

[Auto-detection logic and result]

## Quick Start

- **Extension Development:** See [Extension Guide](extension-guide.md)
- **Source Development:** See [Source Guide](source-guide.md)

## Overview

[Common overview content]

## Repository Access

[Keep existing section - works for both]

## Choosing Your Metric Type

[Keep existing decision tree - works for both]

## Implementation Patterns

[Common patterns across contexts]
- Link to extension-guide.md for extension details
- Link to source-guide.md for source details

## Reference Files

[All existing references - mostly unchanged]

## Package-Specific Considerations

### Yardstick Package Patterns

[Yardstick-specific notes]

## Next Steps

- Extension Development: [Complete Extension Guide](extension-guide.md)
- Source Development: [Complete Source Guide](source-guide.md)
```

#### 3.2 Update add-recipe-step/SKILL.md

Similar changes to yardstick SKILL.md but for recipe steps.

### Stage 4: Update Shared References (2-3 hours)

#### 4.1 Update development-workflow.md

**Add section:**
```markdown
## Git Workflow (Source Development)

When contributing to tidymodels packages:

1. Create feature branch
2. Make changes
3. Commit with clear messages
4. Push and create PR
5. Address review feedback

Keep git workflow minimal - tidymodels developers are familiar with this process.
```

#### 4.2 Update reference navigation

Update all files to point to split references:
- `testing-patterns.md` → `testing-patterns-extension.md` or `testing-patterns-source.md`
- `best-practices.md` → `best-practices-extension.md` or `best-practices-source.md`
- `troubleshooting.md` → `troubleshooting-extension.md` or `troubleshooting-source.md`

### Stage 5: Minor Updates to Existing References (1-2 hours)

#### 5.1 Add notes to metric/step references

Add small callouts where internal functions are mentioned:

```markdown
## Implementation

[Existing content]

**Note for Source Development:** The yardstick package uses `yardstick:::yardstick_mean()`
internally for this calculation. When contributing to yardstick, you can use this helper.
See [Source Guide](../source-guide.md) for details.
```

These are minimal additions - just 1-2 sentences per file where relevant.

---

## Example Implementations

### Example 1: MAE Metric in Both Contexts

#### Extension Version (avoid internals)

```r
# R/mae.R in extension package

mae_impl <- function(truth, estimate, case_weights = NULL) {
  errors <- abs(truth - estimate)

  if (is.null(case_weights)) {
    mean(errors)
  } else {
    # Convert hardhat weights to numeric
    wts <- if (inherits(case_weights, c("hardhat_importance_weights",
                                        "hardhat_frequency_weights"))) {
      as.double(case_weights)
    } else {
      case_weights
    }
    weighted.mean(errors, w = wts)
  }
}
```

#### Source Version (can use internals)

```r
# R/num-mae.R in yardstick package

mae_impl <- function(truth, estimate, case_weights = NULL) {
  errors <- abs(truth - estimate)

  # Use internal helper - we're developing yardstick
  yardstick_mean(errors, case_weights = case_weights)
}
```

**Key Difference:** Source version can use `yardstick_mean()` internal helper. Extension version must implement weight handling manually.

### Example 2: Testing in Both Contexts

#### Extension Testing (create own data)

```r
# tests/testthat/test-mae.R in extension package

test_that("mae works correctly", {
  # Create simple test data
  df <- data.frame(
    truth = c(1, 2, 3, 4, 5),
    estimate = c(1.5, 2.5, 2.5, 3.5, 4.5)
  )

  result <- mae(df, truth, estimate)
  expect_equal(result$.estimate, 0.5)
})
```

#### Source Testing (can use internal helpers)

```r
# tests/testthat/test-num-mae.R in yardstick package

test_that("mae works correctly", {
  # Can use internal test data
  df <- data_altman()

  result <- mae(df, truth = pathology, estimate = scan)

  # Can use snapshot testing like rest of package
  expect_snapshot(result)
})
```

---

## Implementation Checklist

See [phase2-checklist.md](phase2-checklist.md) for detailed task tracking.

---

## Success Criteria

### For Extension Development Path

✅ User can create metrics/steps in new packages without using internals
✅ Clear guidance on exported functions only
✅ Self-contained examples that work
✅ Testing patterns that avoid package internals

### For Source Development Path

✅ User can contribute to yardstick/recipes effectively
✅ Clear guidance on when to use internal functions
✅ Examples match package style exactly
✅ Testing uses package conventions and helpers

### For Both Paths

✅ Auto-detection works correctly
✅ User doesn't need to think about which path
✅ Seamless experience with appropriate guidance
✅ References are clear and context-appropriate

---

## Timeline

### Week 1: Core Infrastructure
- **Days 1-2:** Create split reference files (Stage 1)
- **Days 3-4:** Create extension/source guides for yardstick (Stage 2.1-2.2)

### Week 2: Completion
- **Days 1-2:** Create extension/source guides for recipes (Stage 2.3-2.4)
- **Days 3-4:** Update main SKILL.md files (Stage 3)
- **Day 5:** Final updates and testing (Stages 4-5)

### Total Time: 10-15 hours (2 weeks part-time)

---

## Risk Mitigation

### Risk: Auto-detection fails
**Mitigation:** Fallback to asking user, clear error messages

### Risk: Users confused about which guide to use
**Mitigation:** Auto-detection eliminates choice, guides are clearly labeled

### Risk: Duplication between extension/source guides
**Mitigation:** Keep common content in main SKILL.md, guides only cover differences

### Risk: Package-specific patterns diverge over time
**Mitigation:** Reference actual package files, easy to update

---

## Post-Implementation

### Documentation
- Update README if needed
- Add migration notes for existing users (minimal impact)

### Testing
- Test both contexts with real examples
- Verify auto-detection works
- Check all cross-references

### Maintenance
- Monitor for package changes
- Update when tidymodels conventions change
- Keep examples current

---

## Next Step

Create implementation checklist → [phase2-checklist.md](phase2-checklist.md)
