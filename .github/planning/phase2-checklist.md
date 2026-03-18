# Phase 2 Implementation Checklist

**Date:** 2026-03-18
**Status:** Ready to Start
**Estimated Time:** 10-15 hours

## Progress Tracking

- [ ] **Stage 1:** Create Split Reference Files (3-4 hours)
- [ ] **Stage 2:** Create Extension/Source Guides (4-5 hours)
- [ ] **Stage 3:** Update Main SKILL.md Files (3-4 hours)
- [ ] **Stage 4:** Update Shared References (2-3 hours)
- [ ] **Stage 5:** Minor Updates to Existing References (1-2 hours)

---

## Stage 1: Create Split Reference Files

### 1.1 Split testing-patterns.md

#### Create testing-patterns-extension.md
- [ ] Copy relevant sections from testing-patterns.md
- [ ] Focus on creating test data from scratch
- [ ] Add section: "Never Use Internal Functions"
- [ ] Add section: "Creating Test Data"
  - [ ] Simple data frames
  - [ ] Binary classification data
  - [ ] Multiclass data
  - [ ] Standard datasets (mtcars, iris, modeldata)
- [ ] Add section: "Standard Test Categories"
  - [ ] Correctness tests
  - [ ] NA handling
  - [ ] Input validation
  - [ ] Case weights
  - [ ] Edge cases
- [ ] Add examples for each test type
- [ ] Remove any references to internal functions
- [ ] Update cross-references to other extension files

#### Create testing-patterns-source.md
- [ ] Copy relevant sections from testing-patterns.md
- [ ] Add section: "When to Use Internal Test Helpers"
- [ ] Add section: "Yardstick Test Helpers"
  - [ ] List: `data_altman()`, `data_three_class()`, etc.
  - [ ] When to use each
- [ ] Add section: "Recipes Test Helpers"
  - [ ] List internal test data/functions
- [ ] Add section: "Snapshot Testing"
  - [ ] `expect_snapshot()` patterns
  - [ ] When to use snapshots
- [ ] Add section: "Package-Specific Test Structure"
  - [ ] File naming conventions
  - [ ] Test organization
- [ ] Add examples matching actual package tests
- [ ] Update cross-references to other source files

### 1.2 Split best-practices.md

#### Create best-practices-extension.md
- [ ] Copy relevant sections from best-practices.md
- [ ] Add section: "Exported Functions Only"
  - [ ] Why internal functions are off-limits
  - [ ] How to find exported functions
  - [ ] Base R alternatives
- [ ] Add section: "Code Style"
  - [ ] Base pipe
  - [ ] For-loops over map
  - [ ] Minimal comments
- [ ] Add section: "Error Messages"
  - [ ] Using cli functions
  - [ ] Clear error messages
- [ ] Add section: "Self-Contained Implementations"
  - [ ] When to reimplement vs. depend
- [ ] Remove any internal function examples
- [ ] Update cross-references

#### Create best-practices-source.md
- [ ] Copy relevant sections from best-practices.md
- [ ] Add section: "Using Internal Functions"
  - [ ] When appropriate
  - [ ] When to create new internals
  - [ ] How to document internals
  - [ ] Checking for existing internals
- [ ] Add section: "Following Package Conventions"
  - [ ] Match existing code style
  - [ ] File organization
  - [ ] Naming conventions
- [ ] Add section: "Yardstick Conventions"
  - [ ] Metric file naming (num-, class-, prob-)
  - [ ] Documentation templates
  - [ ] Test organization
- [ ] Add section: "Recipes Conventions"
  - [ ] Step file naming
  - [ ] Helper function patterns
  - [ ] Documentation inheritance
- [ ] Add examples with internal functions
- [ ] Update cross-references

### 1.3 Split troubleshooting.md

#### Create troubleshooting-extension.md
- [ ] Copy relevant sections from troubleshooting.md
- [ ] Add section: "Package Setup Issues"
- [ ] Add section: "Namespace and Import Issues"
  - [ ] "No visible global function definition"
  - [ ] "No visible binding for global variable"
- [ ] Add section: "Dependency Management"
- [ ] Add section: "Common Errors with External Functions"
- [ ] Add section: "Testing Issues"
- [ ] Remove any source-specific troubleshooting
- [ ] Update cross-references

#### Create troubleshooting-source.md
- [ ] Copy relevant sections from troubleshooting.md
- [ ] Add section: "Working with Package Internals"
  - [ ] Finding internal functions
  - [ ] When internal functions change
- [ ] Add section: "Package Check Issues"
  - [ ] Package-specific checks
  - [ ] Integration testing
- [ ] Add section: "Git/Branch Issues" (minimal)
  - [ ] Merge conflicts
  - [ ] Branch synchronization
- [ ] Add section: "PR Submission Issues" (minimal)
  - [ ] Common review feedback
- [ ] Add section: "Yardstick-Specific Issues"
- [ ] Add section: "Recipes-Specific Issues"
- [ ] Update cross-references

### 1.4 Verify Split Files
- [ ] Check all cross-references work
- [ ] Ensure no duplication between extension/source
- [ ] Verify both files are complete
- [ ] Test navigation between files

---

## Stage 2: Create Extension/Source Guides

### 2.1 Create add-yardstick-metric/extension-guide.md

- [ ] Add front matter and title
- [ ] Add section: "When to Use This Guide"
  - [ ] Creating new package
  - [ ] Extending yardstick
  - [ ] Not submitting PR to yardstick
- [ ] Add section: "Prerequisites"
  - [ ] Package setup
  - [ ] Dependencies
  - [ ] Link to r-package-setup.md
- [ ] Add section: "Key Constraints"
  - [ ] Never use :::
  - [ ] Only exported functions
  - [ ] Self-contained implementations
- [ ] Add section: "Step-by-Step Implementation"
  - [ ] Choose metric type
  - [ ] Create implementation function
  - [ ] Create vector interface
  - [ ] Create data frame method
  - [ ] Document
  - [ ] Test
- [ ] Add section: "Complete Example: Custom MAE"
  - [ ] Full implementation
  - [ ] With explanations
  - [ ] Extension-specific patterns
- [ ] Add section: "Common Patterns"
  - [ ] Case weight handling (manual)
  - [ ] NA handling
  - [ ] Error messages
- [ ] Add navigation to:
  - [ ] testing-patterns-extension.md
  - [ ] best-practices-extension.md
  - [ ] troubleshooting-extension.md
- [ ] Add links to reference files

### 2.2 Create add-yardstick-metric/source-guide.md

- [ ] Add front matter and title
- [ ] Add section: "When to Use This Guide"
  - [ ] Contributing PR to yardstick
  - [ ] Cloned yardstick repository
  - [ ] Want metric in yardstick
- [ ] Add section: "Prerequisites"
  - [ ] Repository setup
  - [ ] Clone yardstick
  - [ ] Branch creation
  - [ ] Link to repository-access.md
- [ ] Add section: "Understanding Yardstick's Architecture"
  - [ ] Package organization
  - [ ] File naming conventions
  - [ ] Internal helper system
- [ ] Add section: "Working with Internal Functions"
  - [ ] When to use
  - [ ] How to find existing helpers
  - [ ] Common internal functions:
    - [ ] `yardstick_mean()`
    - [ ] `finalize_estimator_internal()`
    - [ ] `check_metric()`
  - [ ] When to create new internals
- [ ] Add section: "File Naming Conventions"
  - [ ] Numeric: `R/num-[name].R`
  - [ ] Class: `R/class-[name].R`
  - [ ] Probability: `R/prob-[name].R`
  - [ ] Survival: `R/surv-[name].R`
  - [ ] Tests: `tests/testthat/test-[type]-[name].R`
- [ ] Add section: "Documentation Requirements"
  - [ ] Using @template
  - [ ] Using @templateVar
  - [ ] Matching existing style
- [ ] Add section: "Testing with Internal Helpers"
  - [ ] Using `data_altman()`
  - [ ] Using `data_three_class()`
  - [ ] Snapshot testing
- [ ] Add section: "Step-by-Step Implementation"
  - [ ] Choose metric type
  - [ ] Check for existing helpers
  - [ ] Create implementation (may use internals)
  - [ ] Create vector interface
  - [ ] Create data frame method
  - [ ] Document (follow package style)
  - [ ] Test (use package patterns)
- [ ] Add section: "Complete Example: Adding MAE to Yardstick"
  - [ ] Full implementation
  - [ ] Using internal helpers
  - [ ] Matching yardstick style exactly
- [ ] Add section: "PR Submission" (minimal)
  - [ ] Branch naming
  - [ ] Commit messages
  - [ ] Creating PR
- [ ] Add navigation to:
  - [ ] testing-patterns-source.md
  - [ ] best-practices-source.md
  - [ ] troubleshooting-source.md
- [ ] Add links to reference files

### 2.3 Create add-recipe-step/extension-guide.md

- [ ] Add front matter and title
- [ ] Add section: "When to Use This Guide"
- [ ] Add section: "Prerequisites"
- [ ] Add section: "Key Constraints"
  - [ ] Only exported recipes functions
  - [ ] No internal helpers
- [ ] Add section: "Step-by-Step Implementation"
  - [ ] Choose step type
  - [ ] Create step constructor
  - [ ] Create step_new function
  - [ ] Create prep method
  - [ ] Create bake method
  - [ ] Create print/tidy methods
  - [ ] Document
  - [ ] Test
- [ ] Add section: "Complete Example: Custom Centering Step"
  - [ ] Full implementation
  - [ ] Extension-specific patterns
- [ ] Add section: "Common Patterns"
  - [ ] Variable selection (recipes_eval_select)
  - [ ] Case weight handling
  - [ ] Error messages
- [ ] Add navigation to extension references
- [ ] Add links to step type references

### 2.4 Create add-recipe-step/source-guide.md

- [ ] Add front matter and title
- [ ] Add section: "When to Use This Guide"
  - [ ] Contributing to recipes
  - [ ] Cloned recipes repository
- [ ] Add section: "Prerequisites"
  - [ ] Repository setup
  - [ ] Clone recipes
- [ ] Add section: "Understanding Recipes Architecture"
  - [ ] Package organization
  - [ ] Step type organization
  - [ ] Internal helper system
- [ ] Add section: "Working with Internal Functions"
  - [ ] Common internal helpers:
    - [ ] `get_case_weights()`
    - [ ] `recipes_eval_select()`
    - [ ] `check_type()`
    - [ ] `check_new_data()`
- [ ] Add section: "File Naming Conventions"
  - [ ] Steps: `R/[step_name].R`
  - [ ] Tests: `tests/testthat/test-[step_name].R`
- [ ] Add section: "Documentation Requirements"
  - [ ] Using @inheritParams
  - [ ] Matching existing style
  - [ ] Cross-referencing steps
- [ ] Add section: "Testing with Internal Helpers"
  - [ ] Using internal test data
  - [ ] Package test patterns
- [ ] Add section: "Step-by-Step Implementation"
  - [ ] Choose step type
  - [ ] Check for existing helpers
  - [ ] Create step (may use internals)
  - [ ] Document (follow package style)
  - [ ] Test (use package patterns)
- [ ] Add section: "Complete Example: Adding Step to Recipes"
  - [ ] Full implementation
  - [ ] Using internal helpers
  - [ ] Matching recipes style
- [ ] Add section: "PR Submission" (minimal)
- [ ] Add navigation to source references
- [ ] Add links to step type references

---

## Stage 3: Update Main SKILL.md Files

### 3.1 Update add-yardstick-metric/SKILL.md

#### Add Auto-Detection Section
- [ ] Add at top of file (after front matter)
- [ ] Add section: "Context Detection"
- [ ] Explain detection logic:
  - [ ] Check for yardstick DESCRIPTION
  - [ ] Check for other package
  - [ ] Check for non-package project
  - [ ] Empty directory handling
- [ ] Show detected context
- [ ] Add clear visual indicators

#### Add Quick Start Section
- [ ] Add after context detection
- [ ] Link to extension-guide.md
- [ ] Link to source-guide.md
- [ ] Clear call-to-action for each path

#### Update Overview Section
- [ ] Keep common overview content
- [ ] Add notes about context differences
- [ ] Link to appropriate guides

#### Update Prerequisites Section
- [ ] Make context-aware
- [ ] Extension: link to extension guide
- [ ] Source: link to source guide

#### Keep Common Sections
- [ ] Choosing Your Metric Type (unchanged)
- [ ] Complete Example (add context notes)
- [ ] Implementation Guide by Type (add context notes)
- [ ] Documentation (add context notes)
- [ ] Testing (link to split references)

#### Add Package-Specific Section
- [ ] Add section: "Yardstick Package Patterns"
- [ ] File naming conventions
- [ ] Documentation patterns
- [ ] Testing patterns
- [ ] Internal functions overview

#### Update Navigation
- [ ] Update all links to split references
- [ ] Add links to new guides
- [ ] Verify all cross-references

### 3.2 Update add-recipe-step/SKILL.md

#### Add Auto-Detection Section
- [ ] Add at top of file
- [ ] Add section: "Context Detection"
- [ ] Explain detection logic (same as yardstick)
- [ ] Show detected context

#### Add Quick Start Section
- [ ] Link to extension-guide.md
- [ ] Link to source-guide.md

#### Update Overview Section
- [ ] Make context-aware
- [ ] Add context notes

#### Update Prerequisites Section
- [ ] Make context-aware
- [ ] Link to appropriate guides

#### Keep Common Sections
- [ ] Understanding Recipe Steps (unchanged)
- [ ] Step Type Decision Tree (unchanged)
- [ ] Complete Example (add context notes)
- [ ] Implementation Guide by Type (add context notes)

#### Add Package-Specific Section
- [ ] Add section: "Recipes Package Patterns"
- [ ] File naming conventions
- [ ] Documentation patterns
- [ ] Testing patterns
- [ ] Internal functions overview

#### Update Navigation
- [ ] Update all links to split references
- [ ] Add links to new guides
- [ ] Verify all cross-references

---

## Stage 4: Update Shared References

### 4.1 Update development-workflow.md

- [ ] Add section: "Git Workflow (Source Development)"
  - [ ] Keep it minimal
  - [ ] Create branch
  - [ ] Commit changes
  - [ ] Push and create PR
  - [ ] Note: Tidymodels developers know this
- [ ] Update cross-references if needed
- [ ] Verify navigation

### 4.2 Update Cross-References Across Files

#### Files to check and update:
- [ ] r-package-setup.md
  - [ ] Links to testing-patterns → split versions
  - [ ] Links to best-practices → split versions
  - [ ] Links to troubleshooting → split versions
- [ ] roxygen-documentation.md
  - [ ] Update any cross-references
- [ ] package-imports.md
  - [ ] Update any cross-references
- [ ] repository-access.md
  - [ ] Update any cross-references

---

## Stage 5: Minor Updates to Existing References

### 5.1 Update Metric Type References

For each file, add minimal notes about internal functions:

#### metric-system.md
- [ ] Add note: "Source development can use internal helpers"
- [ ] Keep content otherwise unchanged

#### numeric-metrics.md
- [ ] Add callout: "Source can use `yardstick_mean()`"
- [ ] Keep patterns unchanged

#### class-metrics.md
- [ ] Add callout: "Source can use internal estimator functions"
- [ ] Keep patterns unchanged

#### probability-metrics.md
- [ ] Add minimal internal function notes
- [ ] Keep patterns unchanged

#### Other metric type files (8 remaining)
- [ ] Add 1-2 sentence notes where relevant
- [ ] Keep patterns unchanged

### 5.2 Update Step Type References

#### step-architecture.md
- [ ] Add note about internal helpers in source development
- [ ] Keep architecture unchanged

#### modify-in-place-steps.md
- [ ] Add minimal notes about recipes internals
- [ ] Keep patterns unchanged

#### create-new-columns-steps.md
- [ ] Add minimal notes about recipes internals
- [ ] Keep patterns unchanged

#### row-operation-steps.md
- [ ] Add minimal notes about recipes internals
- [ ] Keep patterns unchanged

#### helper-functions.md
- [ ] Add section: "Internal Helpers (Source Development)"
- [ ] List common internal functions
- [ ] Note they're only for source development

#### optional-methods.md
- [ ] No changes needed (already optional)

---

## Verification and Testing

### Pre-Release Checks
- [ ] All new files created
- [ ] All modified files updated
- [ ] All cross-references verified
- [ ] Navigation works throughout
- [ ] No broken links
- [ ] Consistent formatting

### Test Auto-Detection Logic
- [ ] Test in yardstick repository → detects source
- [ ] Test in recipes repository → detects source
- [ ] Test in different package → detects extension
- [ ] Test in non-package directory → detects extension
- [ ] Test in empty directory → asks user

### Test Extension Path
- [ ] Follow extension-guide.md for yardstick
- [ ] Create example metric avoiding internals
- [ ] Verify all links work
- [ ] Test with actual package creation

### Test Source Path
- [ ] Follow source-guide.md for yardstick
- [ ] Create example metric using internals
- [ ] Verify all links work
- [ ] Test in actual yardstick repository

### Test Recipes Skills
- [ ] Repeat tests for recipes skill
- [ ] Verify step creation works
- [ ] Test both extension and source paths

### Documentation Review
- [ ] Check for typos
- [ ] Verify examples are correct
- [ ] Ensure consistency across files
- [ ] Review package-specific sections

---

## Post-Implementation Tasks

### Documentation
- [ ] Update project README if needed
- [ ] Add migration notes for existing users
- [ ] Document the split structure

### Communication
- [ ] Notify stakeholders of changes
- [ ] Provide summary of new features
- [ ] Share examples of both paths

### Maintenance Planning
- [ ] Set up process for updating when packages change
- [ ] Document where to update for package-specific changes
- [ ] Create schedule for periodic review

---

## Notes and Issues

### Decisions Made
- Two-skill structure confirmed
- Auto-detection logic approved
- Split reference files (6 total) approved
- Package-specific guides needed

### Open Issues
(None currently)

### Questions to Resolve
(None currently)

---

## Sign-Off

- [ ] **Stage 1 Complete:** Split reference files created and verified
- [ ] **Stage 2 Complete:** Extension/source guides created for both skills
- [ ] **Stage 3 Complete:** Main SKILL.md files updated with auto-detection
- [ ] **Stage 4 Complete:** Shared references updated
- [ ] **Stage 5 Complete:** Minor updates to existing references
- [ ] **Verification Complete:** All tests passed
- [ ] **Documentation Complete:** All files reviewed and finalized
- [ ] **Ready for Use:** Skills ready for production

---

**Completion Date:** _________________

**Completed By:** _________________

**Notes:** _________________
