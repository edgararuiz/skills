# Phase 1 Implementation Checklist (Revised)

**Goal**: Create standalone cloning scripts and centralized documentation for repository access functionality.

## 1. Create Scripts Directory and Files

### 1.1 Setup Scripts Directory
- [x] Create `shared-scripts/` directory in skills-personal root
- [x] Create README.md in shared-scripts/ explaining script purpose and usage

### 1.2 Write Bash Script: clone-tidymodels-repos.sh
- [x] Create file: `shared-scripts/clone-tidymodels-repos.sh`
- [x] Add shebang and script header documentation
- [x] Implement git installation check
- [x] Implement argument parsing (yardstick, recipes, all)
- [x] Implement repos/ directory creation
- [x] Implement repository existence check
- [x] Implement shallow clone (`git clone --depth 1`)
- [x] Implement .gitignore update logic (Unix line endings)
- [x] Implement .Rbuildignore update logic (Unix line endings)
- [x] Add error handling with appropriate exit codes:
  - 0 = success
  - 1 = git not found
  - 2 = clone failed (network/disk space)
  - 3 = permission error
- [x] Add clear progress messages
- [x] Make script executable (`chmod +x`)
- [x] Fix bash 3.2 compatibility (macOS default - no associative arrays)

### 1.3 Write PowerShell Script: clone-tidymodels-repos.ps1
- [x] Create file: `shared-scripts/clone-tidymodels-repos.ps1`
- [x] Add script header documentation with .SYNOPSIS, .DESCRIPTION, .PARAMETER
- [x] Implement git installation check (Test-Path for git.exe)
- [x] Implement parameter parsing (yardstick, recipes, all)
- [x] Implement repos\ directory creation (New-Item if needed)
- [x] Implement repository existence check (Test-Path)
- [x] Implement shallow clone using git.exe
- [x] Implement .gitignore update logic (handle CRLF line endings)
- [x] Implement .Rbuildignore update logic (handle CRLF line endings)
- [x] Add error handling with appropriate exit codes
- [x] Use Write-Host for colored progress messages
- [x] Set execution policy recommendation in documentation

### 1.4 Write Python Script: clone-tidymodels-repos.py
- [x] Create file: `shared-scripts/clone-tidymodels-repos.py`
- [x] Add shebang and script header documentation
- [x] Implement git installation check (shutil.which('git'))
- [x] Implement argument parsing using argparse (yardstick, recipes, all)
- [x] Implement repos/ directory creation (os.makedirs)
- [x] Implement repository existence check (os.path.exists)
- [x] Implement shallow clone using subprocess
- [x] Implement .gitignore update logic (automatic line ending handling)
- [x] Implement .Rbuildignore update logic (automatic line ending handling)
- [x] Add error handling with appropriate exit codes
- [x] Add colored progress messages (ANSI with Windows fallback)
- [x] Test cross-platform compatibility - tested on macOS
- [x] Fix write_text() newline parameter issue (use open() instead)

## 2. Create Centralized Documentation

### 2.1 Create Repository Access Reference
- [x] Create file: `tidymodels/skills/shared-references/repository-access.md` (11.5 KB)
- [x] Write "Overview" section - Benefits of repository access
- [x] Write "Prerequisites" section - Git installation check and links
- [x] Write "Quick Start" section - Running the clone script
- [x] Write "Step-by-Step Workflow" section - Detailed process
- [x] Write "Using the Scripts" section:
  - [x] Bash script usage examples (macOS/Linux)
  - [x] PowerShell script usage examples (Windows)
  - [x] Python script usage examples (universal fallback)
  - [x] Script selection guide by platform
  - [x] Script output explanation
  - [x] Exit codes reference
- [x] Write "Manual Setup" section - For users who prefer manual cloning
- [x] Write "Troubleshooting" section:
  - [x] Git not installed
  - [x] Permission errors
  - [x] Network issues
  - [x] Disk space issues
  - [x] Repository already exists
  - [x] Python version issues
- [x] Write "What Gets Modified" section:
  - [x] repos/ directory structure
  - [x] .gitignore changes
  - [x] .Rbuildignore changes
- [x] Write "FAQ" section (13 questions):
  - [x] Disk space requirements
  - [x] Update frequency
  - [x] Removing cloned repositories
  - [x] Why not use git submodules?
  - [x] And 9 more questions

## 3. Update Skills (Simplify)

### 3.1 Update add-yardstick-metric Skill
- [x] Remove long "Repository Access Setup" section from SKILL.md (156 lines removed)
- [x] Add brief "Repository Access (Optional but Recommended)" section (25 lines)
- [x] Link to shared-references/repository-access.md
- [x] Show script usage for all platforms (bash, PowerShell, Python)

### 3.2 Update add-recipe-step Skill
- [x] Remove long "Repository Access Setup" section from SKILL.md (156 lines removed)
- [x] Add brief "Repository Access (Optional but Recommended)" section (25 lines)
- [x] Link to shared-references/repository-access.md
- [x] Show script usage for all platforms (bash, PowerShell, Python)

## 4. Testing Scripts

### 4.1 Test Bash Script on macOS
- [ ] Test with git installed - verify detection
- [ ] Test clone single package: `./clone-tidymodels-repos.sh yardstick`
- [ ] Test clone multiple packages: `./clone-tidymodels-repos.sh yardstick recipes`
- [ ] Test clone all: `./clone-tidymodels-repos.sh all`
- [ ] Test with existing repository - verify skip behavior
- [ ] Verify .gitignore created/updated correctly (Unix line endings)
- [ ] Verify .Rbuildignore created/updated correctly (Unix line endings)
- [ ] Verify no duplicates added to ignore files
- [ ] Verify shallow clone (check .git/shallow file)
- [ ] Verify appropriate exit codes
- [ ] Test error messages are clear

### 4.2 Test PowerShell Script on Windows (if available)
- [ ] Test with git installed - verify detection (git.exe in PATH)
- [ ] Test clone single package: `.\clone-tidymodels-repos.ps1 yardstick`
- [ ] Test clone multiple packages: `.\clone-tidymodels-repos.ps1 yardstick recipes`
- [ ] Test clone all: `.\clone-tidymodels-repos.ps1 all`
- [ ] Test with existing repository - verify skip behavior
- [ ] Verify .gitignore created/updated correctly (CRLF line endings)
- [ ] Verify .Rbuildignore created/updated correctly (CRLF line endings)
- [ ] Verify no duplicates added to ignore files
- [ ] Verify shallow clone (check .git/shallow file)
- [ ] Verify appropriate exit codes
- [ ] Test error messages are clear and colored
- [ ] Test execution policy handling

### 4.3 Test Python Script (Universal)
- [ ] Test on macOS: `python3 clone-tidymodels-repos.py yardstick`
- [ ] Test on Linux (if available)
- [ ] Test on Windows (if available)
- [ ] Test clone single package
- [ ] Test clone multiple packages
- [ ] Test clone all packages
- [ ] Test with existing repository
- [ ] Verify ignore file modifications (correct line endings per platform)
- [ ] Verify appropriate exit codes
- [ ] Test error messages are clear

### 4.4 Cross-Platform Testing Summary
- [ ] Bash script verified on macOS
- [ ] Bash script verified on Linux (if available)
- [ ] PowerShell script verified on Windows (if available)
- [ ] Python script verified on at least 2 platforms
- [ ] Document any platform-specific issues or workarounds

### 4.5 Test Without Git (All Scripts)
- [ ] Bash: Simulate git not installed
- [ ] PowerShell: Simulate git.exe not in PATH
- [ ] Python: Simulate git not found by shutil.which
- [ ] Verify all scripts exit with code 1
- [ ] Verify clear error messages with installation instructions

## 5. Edge Cases & Script Error Handling

### 5.1 Git Not Installed
- [ ] Script detects missing git binary
- [ ] Exit code 1
- [ ] Clear error message with platform-specific installation links:
  - macOS: Xcode Command Line Tools or https://git-scm.com/downloads
  - Linux: Package manager instructions (apt-get, yum, etc.)
  - Windows: https://git-scm.com/downloads

### 5.2 No Write Permissions
- [ ] Script detects permission errors when creating repos/
- [ ] Exit code 3
- [ ] Clear error message: "Unable to create repos/ directory. Please check write permissions."

### 5.3 Network Issues
- [ ] Script detects git clone failures
- [ ] Exit code 2
- [ ] Clear error message: "Unable to clone repository. Please check internet connection and try again."

### 5.4 Disk Space Issues
- [ ] Script detects disk space errors (if possible)
- [ ] Exit code 2
- [ ] Error message mentions space requirements (~5 MB yardstick, ~8 MB recipes)

### 5.5 Repository Already Exists
- [ ] Script detects existing repos/{package}/ directory
- [ ] Skip cloning with informative message
- [ ] Optionally offer to update (git pull) - future enhancement
- [ ] Exit code 0 (success, using existing)

## 6. Documentation

### 6.1 Update Main README
- [ ] Add "Repository Access" section
- [ ] Link to shared-references/repository-access.md
- [ ] Briefly explain benefits (enhanced guidance with real examples)
- [ ] Mention scripts location

### 6.2 Scripts README
- [ ] Create shared-scripts/README.md
- [ ] Explain purpose of clone scripts
- [ ] Show usage examples
- [ ] Document exit codes
- [ ] Link to full documentation in shared-references/

### 6.3 Update .github Files
- [ ] Update .Rbuildignore pattern if needed (shared-scripts shouldn't be in package builds)

## 7. Validation

### 7.1 Script Validation
- [ ] Verify scripts are executable
- [ ] Verify scripts work in fresh directory
- [ ] Verify scripts handle all error cases gracefully
- [ ] Verify output messages are clear and actionable

### 7.2 Documentation Validation
- [ ] Verify shared-references/repository-access.md is comprehensive
- [ ] Verify all links work correctly
- [ ] Verify instructions are clear for different user types:
  - Users comfortable with command line
  - Users who prefer manual setup
  - Users who want to skip repository access

### 7.3 End-to-End Validation
- [ ] Create test R package in temp directory
- [ ] Run script: `./shared-scripts/clone-tidymodels-repos.sh yardstick`
- [ ] Verify repos/yardstick/ exists
- [ ] Verify .gitignore includes repos/
- [ ] Verify .Rbuildignore includes ^repos$
- [ ] Invoke skill and verify it can reference cloned files
- [ ] Clean up test package

### 7.4 User Experience Review
- [ ] Skills are concise (long details moved to centralized doc)
- [ ] Scripts provide clear feedback at each step
- [ ] Error messages are actionable
- [ ] Non-blocking: users can skip repository access and still use skills

---

## Progress Tracking

**Started**: 2026-03-17 (initial approach)
**Revised**: 2026-03-17 (script-based approach)
**Completed**: 2026-03-17
**Status**: ✅ **PHASE 1 COMPLETE**

## Revised Approach Summary

### Changes from Initial Implementation
- ❌ **Removed**: Long "Repository Access Setup" sections in each skill
- ✅ **Added**: Standalone scripts in shared-scripts/ directory
- ✅ **Added**: Centralized documentation in shared-references/repository-access.md
- ✅ **Simplified**: Skills now have brief links to centralized docs
- ✅ **Improved**: Reproducible setup with executable scripts
- ✅ **Better**: Clear separation of concerns

### What Will Be Created
1. **Scripts** (platform-native approach):
   - `shared-scripts/clone-tidymodels-repos.sh` (Bash for macOS/Linux/WSL)
   - `shared-scripts/clone-tidymodels-repos.ps1` (PowerShell for Windows)
   - `shared-scripts/clone-tidymodels-repos.py` (Python universal fallback)
   - `shared-scripts/README.md` (usage guide with platform selection)

2. **Documentation**:
   - `tidymodels/skills/shared-references/repository-access.md` (comprehensive guide)

3. **Skill Updates**:
   - Simplified section in `add-yardstick-metric/SKILL.md`
   - Simplified section in `add-recipe-step/SKILL.md`

### Benefits of This Approach
- **Reproducible**: Users can run scripts independently
- **Maintainable**: Single source of truth for setup logic
- **Concise**: Skills stay focused on their core purpose
- **Flexible**: Scripts can be run manually or by Claude
- **Platform-native**: Bash for Unix-like, PowerShell for Windows, Python as universal fallback
- **No dependencies**: Uses tools native to each platform (no Python required on Windows)

### Next Steps (Awaiting Approval)
1. Review this revised checklist
2. Review updated plan in .github/REPOSITORY_ACCESS_PLAN.md
3. If approved, execute Phase 1 tasks in order
4. Test scripts thoroughly
5. Validate with end-to-end test
