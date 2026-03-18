# Phase 2 Implementation Checklist

**Date:** 2026-03-18
**Status:** Ready to Start
**Estimated Time:** 10-15 hours

## Progress Tracking

- [ ] **Stage 1:** Rename Extension Reference Files (15 minutes)
- [ ] **Stage 2:** Create Package-Specific Source Files (4-5 hours)
- [ ] **Stage 3:** Create Extension/Source Guides (4-5 hours)
- [ ] **Stage 4:** Update Main SKILL.md Files (3-4 hours)
- [ ] **Stage 5:** Update Shared References (1-2 hours)
- [ ] **Stage 6:** Minor Updates to Existing References (1-2 hours)

---

## Stage 1: Rename Extension Reference Files

### 1.1 Rename existing files in shared-references/
- [ ] Rename `testing-patterns.md` → `testing-patterns-extension.md`
- [ ] Rename `best-practices.md` → `best-practices-extension.md`
- [ ] Rename `troubleshooting.md` → `troubleshooting-extension.md`

### 1.2 Update content in renamed files
- [ ] Add note at top: "This guide is for extension development"
- [ ] Explicitly state "never use internal functions"
- [ ] Keep all existing content (already focused on avoiding internals)

### 1.3 Update cross-references in other shared-references files
- [ ] Update references in r-package-setup.md
- [ ] Update references in development-workflow.md
- [ ] Update references in roxygen-documentation.md
- [ ] Update references in package-imports.md
- [ ] Update references in repository-access.md

---

## Stage 2: Create Package-Specific Source Files

### 2.1 Create Yardstick Source Files

#### Create add-yardstick-metric/testing-patterns-source.md
- [ ] Add front matter and title
- [ ] Add section: "When to Use Internal Test Helpers"
- [ ] Add section: "Yardstick Internal Test Helpers"
  - [ ] `data_altman()` - Binary classification
  - [ ] `data_three_class()` - Multiclass
  - [ ] `data_hpc_cv1()` - Cross-validation
  - [ ] Other internal helpers
- [ ] Add section: "Snapshot Testing in Yardstick"
  - [ ] When to use `expect_snapshot()`
  - [ ] Example snapshot tests
- [ ] Add section: "File Naming Conventions"
  - [ ] `test-num-*.R` for numeric metrics
  - [ ] `test-class-*.R` for class metrics
  - [ ] `test-prob-*.R` for probability metrics
- [ ] Add section: "Test Organization"
  - [ ] Matching existing test structure
  - [ ] Standard test categories
- [ ] Add examples from actual yardstick tests
- [ ] Link to testing-patterns-extension.md for common patterns

#### Create add-yardstick-metric/best-practices-source.md
- [ ] Add front matter and title
- [ ] Add section: "Using Internal Functions in Yardstick"
  - [ ] `yardstick_mean()` - Weighted mean helper
  - [ ] `finalize_estimator_internal()` - Estimator helper
  - [ ] `check_metric()` - Validation helper
  - [ ] When to create new internals
- [ ] Add section: "File Naming Conventions"
  - [ ] `R/num-[name].R` for numeric metrics
  - [ ] `R/class-[name].R` for class metrics
  - [ ] `R/prob-[name].R` for probability metrics
- [ ] Add section: "Documentation Patterns"
  - [ ] Using `@template`
  - [ ] Using `@templateVar`
  - [ ] Matching existing metric docs
- [ ] Add section: "Code Style Specifics"
  - [ ] Yardstick-specific conventions
  - [ ] Consistency with existing metrics
- [ ] Link to best-practices-extension.md for common patterns

#### Create add-yardstick-metric/troubleshooting-source.md
- [ ] Add front matter and title
- [ ] Add section: "Working with Yardstick Internals"
  - [ ] Finding internal functions
  - [ ] When internals change
- [ ] Add section: "Common Yardstick Issues"
  - [ ] Estimator-related errors
  - [ ] metric_set() integration
  - [ ] Case weight handling
- [ ] Add section: "Package Check Issues"
  - [ ] Yardstick-specific checks
  - [ ] Integration testing
- [ ] Add section: "Git/PR Issues" (minimal)
  - [ ] Branch conflicts
  - [ ] Review process
- [ ] Link to troubleshooting-extension.md for common issues

### 2.2 Create Recipes Source Files

#### Create add-recipe-step/testing-patterns-source.md
- [ ] Add front matter and title
- [ ] Add section: "Using Internal Test Helpers in Recipes"
  - [ ] Internal test recipes
  - [ ] Internal test data
- [ ] Add section: "Testing prep/bake Workflow"
  - [ ] Testing prep() behavior
  - [ ] Testing bake() behavior
  - [ ] Integration testing
- [ ] Add section: "File Naming Conventions"
  - [ ] `test-*.R` for steps
- [ ] Add section: "Test Organization in Recipes"
  - [ ] Matching existing structure
  - [ ] Standard test categories for steps
- [ ] Add examples from actual recipes tests
- [ ] Link to testing-patterns-extension.md for common patterns

#### Create add-recipe-step/best-practices-source.md
- [ ] Add front matter and title
- [ ] Add section: "Using Internal Functions in Recipes"
  - [ ] `recipes_eval_select()` - Variable selection
  - [ ] `get_case_weights()` - Case weights
  - [ ] `check_type()` - Type validation
  - [ ] `check_new_data()` - Data validation
  - [ ] When to create new internals
- [ ] Add section: "File Naming Conventions"
  - [ ] `R/[step_name].R` for steps
- [ ] Add section: "Documentation Patterns"
  - [ ] Heavy use of `@inheritParams`
  - [ ] Cross-referencing steps
  - [ ] Matching existing step docs
- [ ] Add section: "Code Style Specifics"
  - [ ] Recipes-specific conventions
  - [ ] Consistency with existing steps
- [ ] Link to best-practices-extension.md for common patterns

#### Create add-recipe-step/troubleshooting-source.md
- [ ] Add front matter and title
- [ ] Add section: "Working with Recipes Internals"
  - [ ] Finding internal functions
  - [ ] When internals change
- [ ] Add section: "Common Recipes Issues"
  - [ ] Variable selection errors
  - [ ] prep/bake debugging
  - [ ] Case weight handling
- [ ] Add section: "Package Check Issues"
  - [ ] Recipes-specific checks
  - [ ] Integration testing
- [ ] Add section: "Git/PR Issues" (minimal)
  - [ ] Branch conflicts
  - [ ] Review process
- [ ] Link to troubleshooting-extension.md for common issues

### 2.3 Verify Package-Specific Files
- [ ] Check all yardstick files are complete
- [ ] Check all recipes files are complete
- [ ] Verify cross-references work
- [ ] Ensure package-specific nuances are captured
- [ ] Test navigation between files

---

## Stage 3: Create Extension/Source Guides

### 3.1 Create add-yardstick-metric/extension-guide.md

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

### 3.2 Create add-yardstick-metric/source-guide.md

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

### 3.3 Create add-recipe-step/extension-guide.md

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

### 3.4 Create add-recipe-step/source-guide.md

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

## Stage 4: Update Main SKILL.md Files

### 4.1 Update add-yardstick-metric/SKILL.md

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

### 4.2 Update add-recipe-step/SKILL.md

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

## Stage 5: Update Shared References

### 5.1 Update development-workflow.md

- [ ] Add section: "Git Workflow (Source Development)"
  - [ ] Keep it minimal
  - [ ] Create branch
  - [ ] Commit changes
  - [ ] Push and create PR
  - [ ] Note: Tidymodels developers know this
- [ ] Update cross-references if needed
- [ ] Verify navigation

### 5.2 Update Cross-References Across Files

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

## Stage 6: Minor Updates to Existing References

### 6.1 Update Metric Type References

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

### 6.2 Update Step Type References

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
