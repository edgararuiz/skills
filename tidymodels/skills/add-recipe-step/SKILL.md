---
name: add-recipe-step
description: Create a new preprocessing step for the recipes package following tidymodels conventions
---

# Add Recipe Step

Guide for developing new preprocessing steps that extend the recipes package. This skill provides best practices, complete code templates, and testing patterns for creating custom recipe steps.

## Overview

Creating a custom recipe step provides:
- Integration with the recipes preprocessing pipeline
- Automatic handling of variable selection and roles
- Support for case weights
- Consistent prep/bake workflow
- Integration with tune for hyperparameter optimization
- Proper handling of grouped data frames
- Sparse data support (when applicable)

## Prerequisites

### Check and initialize project structure

**CRITICAL: Do this FIRST before attempting to create recipe steps**

```r
# Check if this is a new package or existing package
if (!file.exists("DESCRIPTION")) {
  # New package - create full structure
  usethis::create_package(".", open = FALSE)
  usethis::use_mit_license()  # or use_gpl3_license()
  usethis::use_package("recipes")
  usethis::use_package("rlang")
  usethis::use_package("tibble")
  usethis::use_package("vctrs")
  usethis::use_package("cli")
  usethis::use_testthat()

  # Suggested packages for examples
  usethis::use_package("modeldata", type = "Suggests")
} else {
  # Existing package - ensure dependencies are present
  usethis::use_package("recipes")
  usethis::use_package("rlang")
  usethis::use_package("tibble")
  usethis::use_package("vctrs")
  usethis::use_package("cli")

  # Add testthat if not present
  if (!dir.exists("tests/testthat")) {
    usethis::use_testthat()
  }

  # Suggested packages
  usethis::use_package("modeldata", type = "Suggests")
}
```

**Set up .Rbuildignore:**

Add patterns to exclude from package builds:

```r
# Add common exclusions to .Rbuildignore
writeLines(c(
  "^\\.here$",
  "^\\.claude$",
  "^example.*\\.R$",
  "^.*\\.Rproj$",
  "^\\.Rproj\\.user$"
), ".Rbuildignore", useBytes = TRUE)
```

Or manually edit `.Rbuildignore` to include:
```
^\.here$
^\.claude$
^example.*\.R$
^.*\.Rproj$
^\.Rproj\.user$
```

This prevents `R CMD check` NOTEs about non-standard files.

## Development Workflow

**IMPORTANT:** Follow this workflow to develop your step efficiently.

### During Development (Fast Iteration Cycle)

Run these commands repeatedly while developing:

1. `devtools::document()` - Generate documentation
2. `devtools::load_all()` - Load your package
3. `devtools::test()` - Run tests (or `devtools::test_active_file('R/yourfile.R')` for single file)

This cycle is fast (seconds) and gives you immediate feedback.

### Final Validation (Run Once at End)

4. `devtools::check()` - Full R CMD check

**WARNING:** Do NOT run `check()` during iteration. It takes 1-2 minutes and is unnecessary until you're done. Only run it once at the very end for final validation before submitting/publishing.

## Understanding Recipe Steps

Before implementing, understand the recipe step architecture.

### The Three-Function Pattern

Every recipe step consists of three functions:

1. **Step constructor** (e.g., `step_center()`) - User-facing function
   - Captures user arguments
   - Uses `enquos(...)` to capture variable selections
   - Returns recipe with step added via `add_step()`

2. **Step initialization** (e.g., `step_center_new()`) - Internal constructor
   - Minimal function with no defaults
   - Calls `step(subclass = "name", ...)` to create S3 object

3. **S3 methods** - Required methods for every step:
   - `prep.step_*()` - Estimates parameters from training data
   - `bake.step_*()` - Applies transformation to new data
   - `print.step_*()` - Displays step in recipe summary
   - `tidy.step_*()` - Returns step information as tibble

### The prep/bake Workflow

**prep() - Training phase:**
- Resolves variable selections (e.g., `all_numeric()` → actual column names)
- Validates column types
- Computes statistics/parameters from training data
- Stores learned parameters in step object
- Returns updated step with `trained = TRUE`

**bake() - Application phase:**
- Takes trained step and new data
- Validates required columns exist
- Applies transformation using stored parameters
- Returns transformed data

**Example workflow:**
```r
# Define recipe with step
rec <- recipe(mpg ~ ., data = mtcars) |>
  step_center(all_numeric_predictors())

# prep() trains the step (calculates means)
trained_rec <- prep(rec, training = mtcars)

# bake() applies the step (subtracts means)
new_data <- bake(trained_rec, new_data = mtcars)
```

## Step Type Decision Tree

Choose the appropriate template based on what your step does:

### Type 1: Modify-in-Place Steps
**Use when:** Your step transforms existing columns without creating new ones

**Characteristics:**
- `role = NA` (preserves existing roles)
- No `keep_original_cols` parameter
- Returns tibble with same columns (but modified values)
- Examples: `step_center`, `step_scale`, `step_normalize`, `step_log`

**Template:** See "Creating a Modify-in-Place Step" section below

### Type 2: Create-New-Columns Steps
**Use when:** Your step creates new columns from existing ones

**Characteristics:**
- `role = "predictor"` (default, assigns role to new columns)
- Includes `keep_original_cols` parameter (default `FALSE`)
- Uses `remove_original_cols()` in bake()
- May need `.recipes_estimate_sparsity()` if creating sparse columns
- Examples: `step_dummy`, `step_pca`, `step_interact`, `step_poly`

**Template:** See "Creating a Create-New-Columns Step" section below

### Type 3: Row-Operation Steps
**Use when:** Your step filters or removes rows

**Characteristics:**
- Default `skip = TRUE` (usually not applied during bake on new data)
- Affects number of rows returned
- Often used for training data only
- Examples: `step_filter`, `step_sample`, `step_naomit`, `step_slice`

**Template:** See "Creating a Row-Operation Step" section below

## Creating a Modify-in-Place Step

This is the simplest type of step. It transforms existing columns without creating new ones.

### Complete Template

```r
#' Title for your preprocessing step
#'
#' `step_yourname()` creates a *specification* of a recipe step that will
#' [describe what the step does to the data].
#'
#' @param recipe A recipe object. The step will be added to the sequence of
#'   operations for this recipe.
#' @param ... One or more selector functions to choose variables for this step.
#'   See [recipes::selections()] for more details.
#' @param role Not used by this step since no new variables are created.
#' @param trained A logical to indicate if the quantities for preprocessing have
#'   been estimated.
#' @param [your_param] Description of your step-specific parameter. [Include
#'   type, default value, and what it controls].
#' @param columns A character vector of column names that will be populated
#'   (eventually) by the [terms] argument. This is `NULL` until computed by
#'   [prep()].
#' @param skip A logical. Should the step be skipped when the recipe is baked by
#'   [bake()]? While all operations are baked when [prep()] is run, some
#'   operations may not be able to be conducted on new data (e.g. processing the
#'   outcome variable(s)). Care should be taken when using `skip = TRUE` as it
#'   may affect the computations for subsequent operations.
#' @param id A character string that is unique to this step to identify it.
#'
#' @return An updated version of `recipe` with the new step added to the
#'   sequence of any existing operations.
#'
#' @family [your step category] steps
#' @export
#'
#' @details
#'
#' [Detailed explanation of what the step does, including any important
#' mathematical formulas or computational details.]
#'
#' The step estimates [what parameters] from the data used in the `training`
#' argument of [prep()]. [bake()] then applies [the transformation] to new
#' data sets using these [parameters].
#'
#' # Tidying
#'
#' When you [`tidy()`][recipes::tidy.recipe()] this step, a tibble is returned with
#' columns `terms`, `value`, and `id`:
#'
#' \describe{
#'   \item{terms}{character, the selectors or variables selected}
#'   \item{value}{numeric, the [description of stored values]}
#'   \item{id}{character, id of this step}
#' }
#'
#' # Case weights
#'
#' This step performs an unsupervised operation that can utilize case weights.
#' As a result, case weights are used with frequency weights as well as
#' importance weights. For more information, see the documentation in
#' [recipes::case_weights] and the examples on `tidymodels.org`.
#'
#' @examplesIf rlang::is_installed("modeldata")
#' data(biomass, package = "modeldata")
#'
#' biomass_tr <- biomass[biomass$dataset == "Training", ]
#' biomass_te <- biomass[biomass$dataset == "Testing", ]
#'
#' rec <- recipe(
#'   HHV ~ carbon + hydrogen + oxygen + nitrogen + sulfur,
#'   data = biomass_tr
#' )
#'
#' # Apply your step
#' step_trans <- rec |>
#'   step_yourname(carbon, hydrogen)
#'
#' # View before training
#' step_trans
#'
#' # Train the step
#' step_trained <- prep(step_trans, training = biomass_tr)
#'
#' # View after training
#' step_trained
#'
#' # Apply to new data
#' transformed_te <- bake(step_trained, biomass_te)
#'
#' # View results
#' biomass_te[1:5, c("carbon", "hydrogen")]
#' transformed_te[1:5, c("carbon", "hydrogen")]
#'
#' # View learned parameters
#' tidy(step_trans, number = 1)
#' tidy(step_trained, number = 1)
step_yourname <- function(
  recipe,
  ...,
  role = NA,
  trained = FALSE,
  your_param = default_value,
  columns = NULL,
  skip = FALSE,
  id = recipes::rand_id("yourname")
) {
  recipes::add_step(
    recipe,
    step_yourname_new(
      terms = rlang::enquos(...),
      trained = trained,
      role = role,
      your_param = your_param,
      columns = columns,
      skip = skip,
      id = id,
      case_weights = NULL
    )
  )
}

## Initialize the step - this is internal
step_yourname_new <- function(terms, role, trained, your_param, columns,
                              skip, id, case_weights) {
  recipes::step(
    subclass = "yourname",
    terms = terms,
    role = role,
    trained = trained,
    your_param = your_param,
    columns = columns,
    skip = skip,
    id = id,
    case_weights = case_weights
  )
}

#' @export
prep.step_yourname <- function(x, training, info = NULL, ...) {
  # 1. Resolve variable selections to actual column names
  col_names <- recipes::recipes_eval_select(x$terms, training, info)

  # 2. Validate column types (adjust types as needed for your step)
  recipes::check_type(training[, col_names], types = c("double", "integer"))

  # 3. Extract case weights if applicable
  wts <- recipes::get_case_weights(info, training)
  were_weights_used <- recipes::are_weights_used(wts, unsupervised = TRUE)
  if (isFALSE(were_weights_used)) {
    wts <- NULL
  }

  # 4. Compute parameters needed for transformation
  # Use for-loops over map() for consistency and better error handling
  params <- vector("list", length(col_names))
  names(params) <- col_names

  for (col_name in col_names) {
    # Your computation here
    # Example: compute mean
    if (is.null(wts)) {
      params[[col_name]] <- mean(training[[col_name]], na.rm = x$your_param)
    } else {
      # Handle weighted computation
      params[[col_name]] <- weighted.mean(
        training[[col_name]],
        w = as.double(wts),
        na.rm = x$your_param
      )
    }
  }

  # Convert list to named vector if appropriate
  params <- unlist(params)

  # 5. Optional: Add warnings or checks
  # Example: check for infinite values
  inf_cols <- col_names[is.infinite(params)]
  if (length(inf_cols) > 0) {
    cli::cli_warn(
      "Column{?s} {.var {inf_cols}} returned Inf or NaN. \\
      Consider checking your data before preprocessing."
    )
  }

  # 6. Return updated step with trained = TRUE and parameters stored
  step_yourname_new(
    terms = x$terms,
    role = x$role,
    trained = TRUE,
    your_param = x$your_param,
    columns = col_names,  # Store resolved column names
    skip = x$skip,
    id = x$id,
    case_weights = were_weights_used
  )
}

#' @export
bake.step_yourname <- function(object, new_data, ...) {
  # 1. Get column names from trained step
  col_names <- object$columns

  # 2. Validate required columns exist in new data
  recipes::check_new_data(col_names, object, new_data)

  # 3. Apply transformation using stored parameters
  # Use for-loop for consistency
  for (col_name in col_names) {
    param <- object$columns[[col_name]]  # or however you stored it
    # Example transformation: subtract mean
    new_data[[col_name]] <- new_data[[col_name]] - object$your_param
  }

  # 4. Return modified data
  new_data
}

#' @export
print.step_yourname <- function(x, width = max(20, options()$width - 30), ...) {
  title <- "Your operation description for "
  recipes::print_step(
    x$columns,
    x$terms,
    x$trained,
    title,
    width,
    case_weights = x$case_weights
  )
  invisible(x)
}

#' @rdname tidy.recipe
#' @export
tidy.step_yourname <- function(x, ...) {
  if (recipes::is_trained(x)) {
    # When trained, return actual values
    res <- tibble::tibble(
      terms = names(x$your_stored_param),
      value = unname(x$your_stored_param)
    )
  } else {
    # When untrained, return placeholders
    term_names <- recipes::sel2char(x$terms)
    res <- tibble::tibble(
      terms = term_names,
      value = rlang::na_dbl
    )
  }
  res$id <- x$id
  res
}
```

### Key Implementation Notes

**For prep():**
- Always use `recipes_eval_select()` to resolve variable selections
- Use `check_type()` to validate column types early
- Handle case weights with `get_case_weights()` and `are_weights_used()`
- Use for-loops, not `map()`, for better error messages
- Return a new step object (don't modify in place)
- Set `trained = TRUE` in the returned object

**For bake():**
- Always validate with `check_new_data()`
- Use for-loops for applying transformations
- Work with columns in place (modify `new_data` directly)
- Return the modified data frame

**For print():**
- Use the `print_step()` helper function
- Pass `case_weights` if your step supports them

**For tidy():**
- Return a tibble with at minimum: `terms`, relevant value columns, and `id`
- Handle both trained and untrained states
- Use `sel2char()` to convert untrained term selections to strings

## Creating a Create-New-Columns Step

This template is for steps that create new columns from existing ones.

### Key Differences from Modify-in-Place

1. `role` default is `"predictor"` (not `NA`)
2. Includes `keep_original_cols` parameter
3. Calls `remove_original_cols()` in `bake()`
4. May need to implement `.recipes_estimate_sparsity()` for sparse support
5. `tidy()` returns column names created, not just input columns

### Complete Template

```r
#' Title for your step that creates new columns
#'
#' `step_yournewcols()` creates a *specification* of a recipe step that will
#' create new columns based on [description].
#'
#' @inheritParams step_center
#' @param ... One or more selector functions to choose variables for this step.
#'   See [recipes::selections()] for more details.
#' @param role For model terms created by this step, what analysis role should
#'   they be assigned? By default, the new columns created by this step will
#'   be used as predictors in a model.
#' @param [your_params] Description of step-specific parameters.
#' @param columns A character vector of column names that will be populated
#'   (eventually) by the [terms] argument. This is `NULL` until computed by
#'   [prep()].
#' @param [learned_info] Description of information learned during prep().
#'   This is `NULL` until computed by [prep()].
#' @param keep_original_cols A logical to keep the original variables in the
#'   output. Defaults to `FALSE`.
#'
#' @return An updated version of `recipe` with the new step added to the
#'   sequence of any existing operations.
#'
#' @family [your step category] steps
#' @export
#'
#' @details
#'
#' [Detailed explanation of the step, including formulas and computational
#' details.]
#'
#' When you [`tidy()`][recipes::tidy.recipe()] this step, a tibble is returned with
#' columns `terms`, `columns`, and `id`:
#'
#' \describe{
#'   \item{terms}{character, the selectors or variables selected}
#'   \item{columns}{character, names of the new columns created}
#'   \item{id}{character, id of this step}
#' }
#'
#' # Case weights
#'
#' This step performs an unsupervised operation that can utilize case weights.
#' As a result, case weights are used with frequency weights as well as
#' importance weights. For more information, see the documentation in
#' [recipes::case_weights] and the examples on `tidymodels.org`.
#'
#' @examplesIf rlang::is_installed("modeldata")
#' data(biomass, package = "modeldata")
#'
#' rec <- recipe(HHV ~ ., data = biomass)
#'
#' # Create new columns
#' step_trans <- rec |>
#'   step_yournewcols(carbon, hydrogen)
#'
#' step_trained <- prep(step_trans, training = biomass)
#'
#' # See new columns created
#' transformed <- bake(step_trained, biomass)
#' names(transformed)
#'
#' # Keep original columns
#' step_keep <- rec |>
#'   step_yournewcols(carbon, hydrogen, keep_original_cols = TRUE)
#'
#' step_keep_trained <- prep(step_keep, training = biomass)
#' transformed_keep <- bake(step_keep_trained, biomass)
#' names(transformed_keep)
#'
#' tidy(step_trans, number = 1)
#' tidy(step_trained, number = 1)
step_yournewcols <- function(
  recipe,
  ...,
  role = "predictor",
  trained = FALSE,
  your_param = default_value,
  columns = NULL,
  learned_info = NULL,
  keep_original_cols = FALSE,
  skip = FALSE,
  id = recipes::rand_id("yournewcols")
) {
  recipes::add_step(
    recipe,
    step_yournewcols_new(
      terms = rlang::enquos(...),
      trained = trained,
      role = role,
      your_param = your_param,
      columns = NULL,
      learned_info = learned_info,
      keep_original_cols = keep_original_cols,
      skip = skip,
      id = id,
      case_weights = NULL
    )
  )
}

step_yournewcols_new <- function(terms, role, trained, your_param, columns,
                                  learned_info, keep_original_cols, skip, id,
                                  case_weights) {
  recipes::step(
    subclass = "yournewcols",
    terms = terms,
    role = role,
    trained = trained,
    your_param = your_param,
    columns = columns,
    learned_info = learned_info,
    keep_original_cols = keep_original_cols,
    skip = skip,
    id = id,
    case_weights = case_weights
  )
}

#' @export
prep.step_yournewcols <- function(x, training, info = NULL, ...) {
  col_names <- recipes::recipes_eval_select(x$terms, training, info)
  recipes::check_type(training[, col_names], types = c("double", "integer"))

  wts <- recipes::get_case_weights(info, training)
  were_weights_used <- recipes::are_weights_used(wts, unsupervised = TRUE)
  if (isFALSE(were_weights_used)) {
    wts <- NULL
  }

  # Compute information needed to create new columns
  # Example: compute coefficients, levels, loadings, etc.
  learned_info <- compute_your_transformation(
    training[, col_names],
    wts,
    x$your_param
  )

  step_yournewcols_new(
    terms = x$terms,
    role = x$role,
    trained = TRUE,
    your_param = x$your_param,
    columns = col_names,
    learned_info = learned_info,
    keep_original_cols = x$keep_original_cols,
    skip = x$skip,
    id = x$id,
    case_weights = were_weights_used
  )
}

#' @export
bake.step_yournewcols <- function(object, new_data, ...) {
  col_names <- object$columns
  recipes::check_new_data(col_names, object, new_data)

  # Create new columns based on learned information
  # Example: create interaction terms, dummy variables, etc.
  new_cols <- create_new_columns(
    new_data[, col_names],
    object$learned_info,
    object$your_param
  )

  # Add new columns to data
  new_data <- vctrs::vec_cbind(new_data, new_cols)

  # Optionally remove original columns
  new_data <- recipes::remove_original_cols(new_data, object, col_names)

  new_data
}

#' @export
print.step_yournewcols <- function(x, width = max(20, options()$width - 30), ...) {
  title <- "Creating new columns from "
  recipes::print_step(
    x$columns,
    x$terms,
    x$trained,
    title,
    width,
    case_weights = x$case_weights
  )
  invisible(x)
}

#' @rdname tidy.recipe
#' @export
tidy.step_yournewcols <- function(x, ...) {
  if (recipes::is_trained(x)) {
    # Return information about created columns
    res <- tibble::tibble(
      terms = rep(x$columns, times = lengths_of_new_cols),
      columns = names_of_new_columns_created
    )
  } else {
    term_names <- recipes::sel2char(x$terms)
    res <- tibble::tibble(
      terms = term_names,
      columns = rlang::na_chr
    )
  }
  res$id <- x$id
  res
}

# Optional: Implement sparsity estimation if your step can create sparse columns
.recipes_estimate_sparsity.step_yournewcols <- function(x, data, ...) {
  # Estimate how many columns will be created and their sparsity
  # Return a list with n_cols (integer) and sparsity (numeric between 0 and 1)
  col_names <- recipes::recipes_eval_select(x$terms, data, recipes::get_recipe_info(data))

  # Example: dummy coding creates n-1 columns with estimated sparsity
  n_new_cols <- estimate_number_of_columns(data[, col_names])
  est_sparsity <- estimate_sparsity(data[, col_names])

  list(
    n_cols = n_new_cols,
    sparsity = est_sparsity
  )
}
```

### Additional Notes for Create-New-Columns Steps

**Column naming:**
- Use descriptive, consistent naming conventions
- Consider providing a `naming` parameter for custom naming functions
- Validate that new names don't conflict with existing columns using `recipes::check_name()`

**Sparsity support:**
- If your step creates sparse columns (like dummy variables), implement `.recipes_estimate_sparsity()`
- This allows workflows to optimize sparse data handling
- Return a list with `n_cols` (integer) and `sparsity` (numeric 0-1)

**keep_original_cols:**
- Always use `remove_original_cols()` in `bake()` to handle this parameter
- Don't implement the logic yourself - the helper does it correctly

## Creating a Row-Operation Step

Row-operation steps filter or remove rows. They typically have `skip = TRUE` by default.

### Key Characteristics

- Default `skip = TRUE` - usually not applied during `bake()` on new data
- Affects number of rows, not columns
- Often used only during training
- Examples: filtering, sampling, removing NAs

### Complete Template

```r
#' Filter rows based on condition
#'
#' `step_yourfilter()` creates a *specification* of a recipe step that will
#' remove rows based on [condition].
#'
#' @inheritParams step_center
#' @param ... One or more selector functions or expressions to filter rows.
#' @param [your_params] Parameters that control the filtering.
#'
#' @return An updated version of `recipe` with the new step added to the
#'   sequence of any existing operations.
#'
#' @family row operation steps
#' @export
#'
#' @details
#'
#' This step removes rows from the data during [prep()]. When `skip = TRUE`
#' (the default), the step is ignored during [bake()] on new data.
#'
#' Row-operation steps are typically used to clean or subset training data
#' before model fitting. They are usually not applied to new data during
#' prediction, which is why `skip = TRUE` is the default.
#'
#' When you [`tidy()`][recipes::tidy.recipe()] this step, a tibble is returned with
#' columns `terms` and `id`:
#'
#' \describe{
#'   \item{terms}{character, description of the filter}
#'   \item{id}{character, id of this step}
#' }
#'
#' # Case weights
#'
#' This step performs an unsupervised operation that is not influenced by case
#' weights.
#'
#' @examplesIf rlang::is_installed("modeldata")
#' data(biomass, package = "modeldata")
#'
#' rec <- recipe(HHV ~ ., data = biomass) |>
#'   step_yourfilter(your_params)
#'
#' # During prep, rows are removed
#' prepped <- prep(rec, training = biomass)
#'
#' # Check how many rows remain
#' nrow(biomass)
#' nrow(bake(prepped, new_data = NULL))
#'
#' # When skip = TRUE (default), bake on new data returns all rows
#' nrow(bake(prepped, new_data = biomass))
step_yourfilter <- function(
  recipe,
  ...,
  role = NA,
  trained = FALSE,
  your_param = default_value,
  skip = TRUE,  # Note: default is TRUE for row operations
  id = recipes::rand_id("yourfilter")
) {
  recipes::add_step(
    recipe,
    step_yourfilter_new(
      terms = rlang::enquos(...),
      trained = trained,
      role = role,
      your_param = your_param,
      skip = skip,
      id = id
    )
  )
}

step_yourfilter_new <- function(terms, role, trained, your_param, skip, id) {
  recipes::step(
    subclass = "yourfilter",
    terms = terms,
    role = role,
    trained = trained,
    your_param = your_param,
    skip = skip,
    id = id
  )
}

#' @export
prep.step_yourfilter <- function(x, training, info = NULL, ...) {
  # Row operations typically don't need to "learn" parameters
  # The filtering happens during prep itself

  # Optional: could store information about what was filtered

  step_yourfilter_new(
    terms = x$terms,
    role = x$role,
    trained = TRUE,
    your_param = x$your_param,
    skip = x$skip,
    id = x$id
  )
}

#' @export
bake.step_yourfilter <- function(object, new_data, ...) {
  # When skip = TRUE, this typically returns data unchanged
  # When skip = FALSE, apply the same filtering

  if (!object$skip) {
    # Apply filtering logic
    # Example: filter based on condition
    new_data <- new_data[your_filter_condition, , drop = FALSE]
  }

  new_data
}

#' @export
print.step_yourfilter <- function(x, width = max(20, options()$width - 30), ...) {
  title <- "Row filtering for "
  cat(title, "\n", sep = "")
  invisible(x)
}

#' @rdname tidy.recipe
#' @export
tidy.step_yourfilter <- function(x, ...) {
  res <- tibble::tibble(
    terms = "filter description",
    id = x$id
  )
  res
}
```

## Optional S3 Methods

### tunable() - For Hyperparameter Tuning

If your step has parameters that users might want to tune, implement `tunable()`:

```r
#' @export
tunable.step_yourname <- function(x, ...) {
  tibble::tibble(
    name = "your_param",
    call_info = list(
      list(pkg = "dials", fun = "your_param_range")
    ),
    source = "recipe",
    component = "step_yourname",
    component_id = x$id
  )
}
```

### required_pkgs() - For Package Dependencies

If your step needs external packages, declare them:

```r
#' @export
required_pkgs.step_yourname <- function(x, ...) {
  c("yourpackage", "anotherpackage")
}
```

Then check for them in your step function:

```r
step_yourname <- function(recipe, ...) {
  recipes::recipes_pkg_check(required_pkgs.step_yourname())

  # ... rest of function
}
```

### Sparsity Methods - For Sparse Data Support

If your step preserves sparsity:

```r
#' @export
.recipes_preserve_sparsity.step_yourname <- function(x, ...) {
  TRUE  # or FALSE
}
```

If your step creates sparse columns (see Create-New-Columns template above):

```r
#' @export
.recipes_estimate_sparsity.step_yourname <- function(x, data, ...) {
  # Return list(n_cols = <int>, sparsity = <0-1>)
}
```

## Testing Guide

Tests should be comprehensive but minimal in comments. Use `testthat` and follow these patterns.

### Required Test Categories

Every step needs these tests:

#### 1. Correctness Test

```r
test_that("working correctly", {
  # Setup test data
  rec <- recipe(mpg ~ ., data = mtcars) |>
    step_yourname(disp, hp)

  # Test untrained tidy()
  untrained_tidy <- tidy(rec, 1)
  expect_equal(untrained_tidy$value, rep(rlang::na_dbl, 2))

  # Test prep()
  prepped <- prep(rec, training = mtcars)

  # Verify parameters were learned correctly
  expect_equal(
    prepped$steps[[1]]$your_param,
    expected_values
  )

  # Test trained tidy()
  trained_tidy <- tidy(prepped, 1)
  expect_equal(trained_tidy$value, expected_values)

  # Test bake()
  results <- bake(prepped, mtcars)

  # Verify transformation was applied correctly
  expect_equal(
    results$disp,
    expected_transformed_values,
    tolerance = 1e-7
  )
})
```

#### 2. Single Predictor Test

```r
test_that("single predictor", {
  rec <- recipe(mpg ~ ., data = mtcars) |>
    step_yourname(disp)

  prepped <- prep(rec, training = mtcars)
  results <- bake(prepped, mtcars)

  # Verify it works with just one column
  expect_equal(
    results$disp,
    expected_values
  )
})
```

#### 3. Parameter Validation Test

```r
test_that("parameter validation works", {
  rec <- recipe(mpg ~ ., data = mtcars) |>
    step_yourname(disp, your_param = "invalid")

  # Use expect_snapshot for errors
  expect_snapshot(error = TRUE, prep(rec, training = mtcars))
})
```

#### 4. NA Handling Test (if applicable)

```r
test_that("na_rm argument works", {
  mtcars_na <- mtcars
  mtcars_na[1, c("disp", "hp")] <- NA

  rec_no_na_rm <- recipe(mpg ~ ., data = mtcars_na) |>
    step_yourname(disp, hp, na_rm = FALSE) |>
    prep()

  rec_na_rm <- recipe(mpg ~ ., data = mtcars_na) |>
    step_yourname(disp, hp, na_rm = TRUE) |>
    prep()

  # Verify NA handling is different
  expect_true(
    is.na(tidy(rec_no_na_rm, 1)$value[1])
  )
  expect_false(
    is.na(tidy(rec_na_rm, 1)$value[1])
  )
})
```

#### 5. Case Weights Test (if applicable)

```r
test_that("step works with case weights", {
  mtcars_freq <- mtcars
  mtcars_freq$cyl <- hardhat::frequency_weights(mtcars_freq$cyl)

  rec <- recipe(mpg ~ ., mtcars_freq) |>
    step_yourname(disp, hp) |>
    prep()

  # Verify weighted computation differs from unweighted
  expect_equal(
    tidy(rec, number = 1)[["value"]],
    expected_weighted_values
  )

  # Snapshot to show case weights were used
  expect_snapshot(rec)

  # Test importance weights (should be ignored for unsupervised)
  mtcars_imp <- mtcars
  mtcars_imp$wt_var <- hardhat::importance_weights(mtcars_imp$wt)

  rec_imp <- recipe(mpg ~ ., mtcars_imp) |>
    step_yourname(disp, hp) |>
    prep()

  # Should match unweighted
  expect_equal(
    tidy(rec_imp, number = 1)[["value"]],
    expected_unweighted_values
  )

  expect_snapshot(rec_imp)
})
```

#### 6. Infrastructure Tests (Required for ALL steps)

These ensure your step works correctly in edge cases:

```r
# Infrastructure tests section
test_that("bake method errors when needed columns are missing", {
  rec <- recipe(mpg ~ ., data = mtcars) |>
    step_yourname(disp, hp)

  rec_trained <- prep(rec, training = mtcars)

  expect_snapshot(
    error = TRUE,
    bake(rec_trained, new_data = mtcars[, 1:2])
  )
})

test_that("empty printing", {
  rec <- recipe(mpg ~ ., mtcars) |>
    step_yourname()

  expect_snapshot(rec)
  expect_snapshot(prep(rec, mtcars))
})

test_that("empty selection prep/bake is a no-op", {
  rec1 <- recipe(mpg ~ ., mtcars)
  rec2 <- step_yourname(rec1)

  rec1 <- prep(rec1, mtcars)
  rec2 <- prep(rec2, mtcars)

  baked1 <- bake(rec1, mtcars)
  baked2 <- bake(rec2, mtcars)

  expect_identical(baked1, baked2)
})

test_that("empty selection tidy method works", {
  rec <- recipe(mpg ~ ., mtcars) |>
    step_yourname()

  expect <- tibble::tibble(
    terms = character(),
    value = double(),
    id = character()
  )

  expect_identical(tidy(rec, number = 1), expect)

  rec <- prep(rec, mtcars)
  expect_identical(tidy(rec, number = 1), expect)
})

test_that("printing", {
  rec <- recipe(mpg ~ ., mtcars) |>
    step_yourname(disp)

  expect_snapshot(print(rec))
  expect_snapshot(prep(rec))
})

test_that("0 and 1 rows data work in bake method", {
  rec <- recipe(~., data = mtcars) |>
    step_yourname(mpg, disp) |>
    prep()

  expect_identical(
    nrow(bake(rec, dplyr::slice(mtcars, 1))),
    1L
  )
  expect_identical(
    nrow(bake(rec, dplyr::slice(mtcars, 0))),
    0L
  )
})
```

### Testing Best Practices

**Use expect_snapshot():**
- For error messages: `expect_snapshot(error = TRUE, ...)`
- For warnings: `expect_snapshot(warning = TRUE, ...)`
- For printed output: `expect_snapshot(print(object))`

**Minimal comments:**
- Let the test names be descriptive
- Don't over-comment obvious code

**Use meaningful data:**
- Use built-in datasets like `mtcars`, `iris`
- Or use `modeldata` package datasets
- Create minimal synthetic data when needed

**Precision:**
- Use `expect_identical()` for exact matches
- Use `expect_equal()` with `tolerance` for floating point
- Avoid `expect_true()` / `expect_false()` - be specific

**Test file naming:**
- Match source file: `R/yourname.R` → `tests/testthat/test-yourname.R`

## Helper Functions Reference

Key recipes functions you'll use frequently:

| Function | Purpose | Usage |
|----------|---------|-------|
| `recipes_eval_select()` | Convert quosures to column names | `prep()` method |
| `check_type()` | Validate column types | `prep()` method |
| `check_new_data()` | Verify columns exist | `bake()` method |
| `check_name()` | Prevent name conflicts | When creating new columns |
| `get_case_weights()` | Extract case weights | `prep()` method |
| `are_weights_used()` | Check if weights apply | `prep()` method |
| `rand_id()` | Generate unique IDs | Step constructor |
| `print_step()` | Standard printing | `print()` method |
| `remove_original_cols()` | Handle keep_original_cols | `bake()` method |
| `sel2char()` | Convert selections to strings | `tidy()` method |
| `is_trained()` | Check training status | `tidy()` method |

## Best Practices

### Code Style

**Use base pipe:**
```r
# Good
rec |> step_center(x, y)

# Avoid
rec %>% step_center(x, y)
```

**Anonymous functions:**
```r
# Single line: use backslash notation
map(x, \(i) i + 1)

# Multi-line: use function()
map(x, function(i) {
  result <- complex_computation(i)
  result + 1
})
```

**For-loops over map():**
```r
# Preferred for recipe steps (better error messages)
for (col in columns) {
  new_data[[col]] <- transform(new_data[[col]])
}

# Avoid
new_data <- map(columns, \(col) transform(new_data[[col]]))
```

**Error messages:**
```r
# Use cli::cli_abort()
if (invalid) {
  cli::cli_abort("{.arg param} must be positive, not {.val {param}}.")
}

# Avoid base stop()
stop("param must be positive")
```

**Format code:**
After writing, format with:
```r
air::air_format(".")
```

### Documentation Standards

**Be explicit about parameters:**
- Document what each parameter does
- Include type, default value, and valid range
- Explain how it affects the step

**Include examples:**
- Show basic usage
- Show with options/parameters
- Show `tidy()` output
- Use `@examplesIf rlang::is_installed("modeldata")`

**US English:**
- Use American spelling (e.g., "normalize" not "normalise")
- Use sentence case for descriptions

**Wrap roxygen at 80 characters:**
Use line breaks to keep documentation readable.

### Performance

**Avoid repeated computations:**
```r
# Good: compute once in prep()
prep.step_yourname <- function(x, training, ...) {
  means <- compute_means(training)  # Computed once
  # Store in step object
}

# Bad: compute in bake()
bake.step_yourname <- function(object, new_data, ...) {
  means <- compute_means(new_data)  # Computed every time!
}
```

**Use vectorized operations:**
```r
# Good
new_data$x <- new_data$x - mean_x

# Less good
for (i in seq_len(nrow(new_data))) {
  new_data$x[i] <- new_data$x[i] - mean_x
}
```

**Handle large data:**
- Consider memory usage in `prep()`
- Don't store entire datasets in step objects
- Store only necessary parameters/statistics

### Error Handling

**Validate early:**
- Check parameters in constructor if possible
- Validate data types in `prep()`
- Give clear, actionable error messages

**Example validation:**
```r
step_yourname <- function(recipe, ..., your_param = 1) {
  # Validate parameters early
  if (!is.numeric(your_param) || your_param <= 0) {
    cli::cli_abort("{.arg your_param} must be a positive number.")
  }

  # ... rest of function
}

prep.step_yourname <- function(x, training, ...) {
  # Validate data early
  col_names <- recipes_eval_select(x$terms, training, info)
  check_type(training[, col_names], types = c("double", "integer"))

  # ... rest of function
}
```

## Reference Examples

For real-world examples, see the recipes package source code:

- **Simple modify-in-place step:** `step_center()` in recipes/R/center.R
- **Create-new-columns step:** `step_dummy()` in recipes/R/dummy.R
- **Row-operation step:** `step_filter()` in recipes/R/filter.R
- **Testing patterns:** recipes/tests/testthat/test-center.R

Also see the [tidymodels developer guide](https://www.tidymodels.org/learn/develop/recipes/).

## Final Checklist

Before submitting your step, verify:

### Step Implementation
- [ ] Step constructor `step_<name>()` created
  - [ ] Correct signature: `recipe, ..., role = NA, trained = FALSE, [params], skip = FALSE, id`
  - [ ] Uses `enquos(...)` for variable selection
  - [ ] Calls `add_step()` with initialization function
  - [ ] Validates parameters if possible
- [ ] Step initialization `step_<name>_new()` created
  - [ ] Calls `step(subclass = "name", ...)`
  - [ ] No default values in parameters
- [ ] `prep.step_<name>()` method implemented
  - [ ] Uses `recipes_eval_select()` for selection
  - [ ] Uses `check_type()` for validation
  - [ ] Handles case weights correctly
  - [ ] Computes/stores parameters
  - [ ] Returns new step with `trained = TRUE`
  - [ ] Uses for-loops, not `map()`
- [ ] `bake.step_<name>()` method implemented
  - [ ] Uses `check_new_data()` for validation
  - [ ] Applies transformation correctly
  - [ ] Returns tibble
  - [ ] Uses `remove_original_cols()` if applicable
  - [ ] Uses for-loops, not `map()`
- [ ] `print.step_<name>()` method implemented
  - [ ] Uses `print_step()` helper
  - [ ] Shows case weights if applicable
- [ ] `tidy.step_<name>()` method implemented
  - [ ] Returns tibble with appropriate columns
  - [ ] Includes `id` column
  - [ ] Handles trained and untrained states
  - [ ] Uses `sel2char()` for untrained terms

### Optional Methods
- [ ] `tunable.step_<name>()` if parameters are tunable
- [ ] `required_pkgs.step_<name>()` if using external packages
- [ ] `.recipes_preserve_sparsity()` if preserving sparsity
- [ ] `.recipes_estimate_sparsity()` if creating sparse columns

### Documentation
- [ ] Complete roxygen2 documentation
  - [ ] All @param entries documented
  - [ ] @return documented
  - [ ] @details section with explanation
  - [ ] # Tidying section describing tidy() output
  - [ ] # Case weights section (or template)
  - [ ] @family tag for categorization
  - [ ] @export on appropriate functions
  - [ ] @examplesIf with working examples
  - [ ] Examples use `|>` pipe, not `%>%`
- [ ] Documentation builds without errors: `devtools::document()`

### Testing
- [ ] Test file created: `tests/testthat/test-<name>.R`
- [ ] Correctness test implemented
- [ ] Single predictor test implemented
- [ ] Parameter validation test implemented
- [ ] NA handling test (if applicable)
- [ ] Case weights test (if applicable)
- [ ] All 6 infrastructure tests implemented:
  - [ ] Missing columns error test
  - [ ] Empty printing test
  - [ ] Empty selection no-op test
  - [ ] Empty selection tidy test
  - [ ] Printing test
  - [ ] 0 and 1 row test
- [ ] Tests use `expect_snapshot()` appropriately
- [ ] Minimal or no comments in tests
- [ ] All tests pass: `devtools::test_active_file('R/<name>.R')`

### Code Quality
- [ ] Code formatted: `air::air_format(".")`
- [ ] Uses `cli::cli_abort()` not `stop()`
- [ ] Uses for-loops not `map()` in prep/bake
- [ ] Uses base pipe `|>` not `%>%`
- [ ] Error messages are clear and actionable
- [ ] No internal/non-exported functions used

### Final Validation
- [ ] Package checks pass: `devtools::check()`
- [ ] No NOTEs, WARNINGs, or ERRORs
- [ ] Examples run without error
- [ ] All tests pass

## Common Pitfalls

**Using internal recipes functions:**
Many recipes functions are internal. Stick to exported functions like those in the Helper Functions Reference table.

**Not handling empty selections:**
Always test with empty selections - your step should be a no-op.

**Forgetting infrastructure tests:**
All 6 infrastructure tests are required. Don't skip them.

**Using map() instead of for-loops:**
For-loops provide better error messages in recipe steps.

**Not validating in prep():**
Validate data types and column existence in `prep()`, not `bake()`.

**Storing too much in step objects:**
Only store parameters needed for transformation, not entire datasets.

**Inconsistent tidy() output:**
Always include an `id` column and handle both trained/untrained states.

**Not testing with case weights:**
If your step is unsupervised, test with both frequency and importance weights.

**Poor error messages:**
Use `cli::cli_abort()` with informative, actionable messages.

## Getting Help

If you get stuck:

1. Look at existing recipe steps as examples
2. Check the [tidymodels developer guide](https://www.tidymodels.org/learn/develop/recipes/)
3. Ask on [RStudio Community](https://community.rstudio.com/) with the `tidymodels` tag
4. File an issue on [GitHub](https://github.com/tidymodels/recipes/issues) if you find a bug
