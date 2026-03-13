---
name: add-yardstick-metric
description: Guide for creating new yardstick metrics. Use when a developer needs to extend yardstick with a custom performance metric, including numeric, class, or probability metrics.
---

# Add Yardstick Metric

Guide for developing new metrics that extend the yardstick package. This skill provides best practices, code templates, and testing patterns for creating custom performance metrics.

## Overview

Creating a custom yardstick metric provides:
- Standardization with existing metrics
- Automatic error handling for types and lengths
- Support for multiclass implementations
- NA handling
- Grouped data frame support
- Integration with `metric_set()`

## Prerequisites

### Check and initialize project structure

**CRITICAL: Do this FIRST before attempting to create metrics**

```r
# Check if this is a new package or existing package
if (!file.exists("DESCRIPTION")) {
  # New package - create full structure
  usethis::create_package(".", open = FALSE)
  usethis::use_mit_license()  # or use_gpl3_license()
  usethis::use_package("yardstick")
  usethis::use_package("rlang")
  usethis::use_package("cli")
  usethis::use_testthat()
} else {
  # Existing package - ensure dependencies are present
  usethis::use_package("yardstick")
  usethis::use_package("rlang")
  usethis::use_package("cli")

  # Add testthat if not present
  if (!dir.exists("tests/testthat")) {
    usethis::use_testthat()
  }
}
```

## Metric types

Yardstick supports several metric types:
- **Numeric metrics**: For regression (e.g., MAE, RMSE, MSE)
- **Class metrics**: For classification (e.g., accuracy, precision, recall)
- **Probability metrics**: For class probabilities (e.g., ROC AUC, log loss)
- **Survival metrics**: For survival analysis
- **Quantile metrics**: For quantile predictions

## CRITICAL: Exported vs Internal Functions

Many yardstick helper functions are INTERNAL and not exported. Using them will cause runtime errors.

### ❌ Don't Use (Internal/Not Exported)
- `yardstick_mean()` - NOT EXPORTED
- `get_weights()` - NOT EXPORTED
- `metric_range()` - NOT EXPORTED
- `metric_optimal()` - NOT EXPORTED
- `metric_direction()` - NOT EXPORTED
- `data_altman()` - NOT EXPORTED (test helper)
- `data_three_class()` - NOT EXPORTED (test helper)

### ✅ Use Instead

**For weighted calculations:**
```r
# Instead of yardstick_mean(), use base R weighted.mean()
if (is.null(case_weights)) {
  mean(values)
} else {
  # Handle hardhat weights (convert to numeric)
  wts <- if (inherits(case_weights, "hardhat_importance_weights") ||
             inherits(case_weights, "hardhat_frequency_weights")) {
    as.double(case_weights)
  } else {
    case_weights
  }
  weighted.mean(values, w = wts)
}
```

**EXPORTED yardstick functions you CAN safely use:**
- `check_numeric_metric()` ✓
- `check_class_metric()` ✓
- `check_prob_metric()` ✓
- `yardstick_remove_missing()` ✓
- `yardstick_any_missing()` ✓
- `yardstick_table()` ✓
- `finalize_estimator()` ✓
- `validate_estimator()` ✓
- `abort_if_class_pred()` ✓
- `as_factor_from_class_pred()` ✓
- `numeric_metric_summarizer()` ✓
- `class_metric_summarizer()` ✓
- `prob_metric_summarizer()` ✓
- `new_numeric_metric()` ✓
- `new_class_metric()` ✓
- `new_prob_metric()` ✓

## Creating a numeric metric

Numeric metrics are the simplest to implement. They measure continuous predictions against continuous truth values.

### Step 1: Define the implementation function

Create the core calculation function. Use the `_impl` suffix:

```r
# Example: Mean Squared Error
mse_impl <- function(truth, estimate, case_weights = NULL) {
  errors <- (truth - estimate) ^ 2

  if (is.null(case_weights)) {
    mean(errors)
  } else {
    # Handle hardhat weights
    wts <- if (inherits(case_weights, "hardhat_importance_weights") ||
               inherits(case_weights, "hardhat_frequency_weights")) {
      as.double(case_weights)
    } else {
      case_weights
    }
    weighted.mean(errors, w = wts)
  }
}
```

**Key patterns:**
- Take `truth`, `estimate`, and optionally `case_weights`
- Return a single numeric value
- Use `weighted.mean()` for weighted calculations
- Handle hardhat weight classes by converting to numeric

### Step 2: Create the vector function

```r
mse_vec <- function(truth, estimate, na_rm = TRUE, case_weights = NULL, ...) {
  # Validate na_rm
  if (!is.logical(na_rm) || length(na_rm) != 1) {
    cli::cli_abort("{.arg na_rm} must be a single logical value.")
  }

  # Validate inputs
  yardstick::check_numeric_metric(truth, estimate, case_weights)

  # Handle NA values
  if (na_rm) {
    result <- yardstick::yardstick_remove_missing(truth, estimate, case_weights)
    truth <- result$truth
    estimate <- result$estimate
    case_weights <- result$case_weights
  } else if (yardstick::yardstick_any_missing(truth, estimate, case_weights)) {
    return(NA_real_)
  }

  mse_impl(truth, estimate, case_weights)
}
```

**Required elements:**
- Validate `na_rm` parameter explicitly
- Use `check_numeric_metric()` for validation
- Handle NA values consistently using `yardstick_remove_missing()`
- Return `NA_real_` if `na_rm = FALSE` and NAs present

### Step 3: Create the data frame method

```r
mse <- function(data, ...) {
  UseMethod("mse")
}

mse <- yardstick::new_numeric_metric(
  mse,
  direction = "minimize",  # or "maximize" or "zero"
  range = c(0, Inf)
)

#' @export
#' @rdname mse
mse.data.frame <- function(data, truth, estimate, na_rm = TRUE,
                           case_weights = NULL, ...) {
  yardstick::numeric_metric_summarizer(
    name = "mse",
    fn = mse_vec,
    data = data,
    truth = !!rlang::enquo(truth),
    estimate = !!rlang::enquo(estimate),
    na_rm = na_rm,
    case_weights = !!rlang::enquo(case_weights)
  )
}
```

**Key patterns:**
- Use `new_numeric_metric()` to create the metric function
- Set `direction` to "minimize", "maximize", or "zero"
- Specify `range` as `c(min, max)` (use `Inf` or `-Inf` for unbounded)
- Use `rlang::enquo()` and `!!` for NSE support
- Export the data frame method with `@export`

## Creating a class metric

Class metrics are more complex due to multiclass support.

### Step 1: Binary implementation

```r
# Example: Miss Rate (False Negative Rate)
miss_rate_binary <- function(data, event_level) {
  # data is a confusion matrix (table)
  col <- if (identical(event_level, "first")) {
    colnames(data)[[1]]
  } else {
    colnames(data)[[2]]
  }
  col2 <- setdiff(colnames(data), col)

  tp <- data[col, col]
  fn <- data[col2, col]

  fn / (fn + tp)
}
```

### Step 2: Multiclass implementation (optional)

```r
miss_rate_multiclass <- function(data, estimator) {
  # Calculate per-class values
  tp <- diag(data)
  tpfn <- colSums(data)
  fn <- tpfn - tp

  # For micro averaging, sum first
  if (estimator == "micro") {
    tp <- sum(tp)
    fn <- sum(fn)
  }

  # Return vector of per-class values (or single value for micro)
  fn / (fn + tp)
}
```

### Step 3: Estimator implementation

```r
miss_rate_estimator_impl <- function(data, estimator, event_level) {
  if (estimator == "binary") {
    miss_rate_binary(data, event_level)
  } else {
    # Calculate per-class metrics
    res <- miss_rate_multiclass(data, estimator)

    # Get weights based on class frequencies
    class_counts <- colSums(data)
    wt <- switch(estimator,
      "macro" = rep(1, length(res)),  # Equal weights
      "macro_weighted" = class_counts,  # Weighted by frequency
      "micro" = rep(1, length(res))  # Already aggregated
    )

    weighted.mean(res, wt)
  }
}

miss_rate_impl <- function(truth, estimate, estimator, event_level, case_weights) {
  xtab <- yardstick::yardstick_table(truth, estimate, case_weights = case_weights)
  miss_rate_estimator_impl(xtab, estimator, event_level)
}
```

### Step 4: Vector function

```r
miss_rate_vec <- function(truth, estimate, estimator = NULL, na_rm = TRUE,
                          case_weights = NULL, event_level = "first", ...) {
  # Validate na_rm
  if (!is.logical(na_rm) || length(na_rm) != 1) {
    cli::cli_abort("{.arg na_rm} must be a single logical value.")
  }

  yardstick::abort_if_class_pred(truth)
  estimate <- yardstick::as_factor_from_class_pred(estimate)

  estimator <- yardstick::finalize_estimator(
    truth,
    estimator,
    metric_class = "miss_rate"
  )

  yardstick::check_class_metric(truth, estimate, case_weights, estimator)

  if (na_rm) {
    result <- yardstick::yardstick_remove_missing(truth, estimate, case_weights)
    truth <- result$truth
    estimate <- result$estimate
    case_weights <- result$case_weights
  } else if (yardstick::yardstick_any_missing(truth, estimate, case_weights)) {
    return(NA_real_)
  }

  miss_rate_impl(truth, estimate, estimator, event_level, case_weights)
}
```

### Step 5: Data frame method

```r
miss_rate <- function(data, ...) {
  UseMethod("miss_rate")
}

miss_rate <- yardstick::new_class_metric(
  miss_rate,
  direction = "minimize",
  range = c(0, 1)
)

#' @export
#' @rdname miss_rate
miss_rate.data.frame <- function(data, truth, estimate, estimator = NULL,
                                 na_rm = TRUE, case_weights = NULL,
                                 event_level = "first", ...) {
  yardstick::class_metric_summarizer(
    name = "miss_rate",
    fn = miss_rate_vec,
    data = data,
    truth = !!rlang::enquo(truth),
    estimate = !!rlang::enquo(estimate),
    estimator = estimator,
    na_rm = na_rm,
    case_weights = !!rlang::enquo(case_weights),
    event_level = event_level
  )
}
```

### Step 6: Restrict estimator (optional)

If your metric only supports binary classification:

```r
finalize_estimator_internal.miss_rate <- function(metric_dispatcher, x,
                                                   estimator, call) {
  yardstick::validate_estimator(estimator, estimator_override = "binary")

  if (!is.null(estimator)) {
    return(estimator)
  }

  lvls <- levels(x)
  if (length(lvls) > 2) {
    cli::cli_abort(
      "A multiclass {.arg truth} input was provided, but only {.code binary} is supported."
    )
  }

  "binary"
}
```

## Documentation

### Roxygen template for numeric metrics

**Do NOT use `@template` tags - they won't exist in your package**

```r
#' Metric Name
#'
#' Brief description of what this metric measures.
#'
#' @family numeric metrics
#'
#' @param data A data frame containing the columns specified by `truth` and
#'   `estimate`.
#' @param truth The column identifier for the true results (numeric). This
#'   should be an unquoted column name.
#' @param estimate The column identifier for the predicted results (numeric).
#'   This should be an unquoted column name.
#' @param na_rm A logical value indicating whether NA values should be stripped
#'   before the computation proceeds. Default is `TRUE`.
#' @param case_weights The optional column identifier for case weights. This
#'   should be an unquoted column name. Default is `NULL`.
#' @param ... Not currently used.
#'
#' @return
#' A tibble with columns `.metric`, `.estimator`, and `.estimate` and 1 row of
#' values.
#'
#' For grouped data frames, the number of rows returned will be the same as the
#' number of groups.
#'
#' For `metric_name_vec()`, a single numeric value (or `NA`).
#'
#' @details
#' [metric_name()] is a metric that should be [maximized/minimized].
#' The output ranges from [min] to [max], with [optimal_value] indicating
#' perfect predictions.
#'
#' The formula for [metric name] is:
#'
#' \deqn{formula here}
#'
#' @author Your Name
#'
#' @examples
#' # Create sample data
#' df <- data.frame(
#'   truth = c(1, 2, 3, 4, 5),
#'   estimate = c(1.1, 2.2, 2.9, 4.1, 5.2)
#' )
#'
#' # Basic usage
#' metric_name(df, truth, estimate)
#'
#' @export
```

### Roxygen template for class metrics

```r
#' Metric Name
#'
#' Brief description of what this metric measures.
#'
#' @family class metrics
#'
#' @param data A data frame containing the columns specified by `truth` and
#'   `estimate`.
#' @param truth The column identifier for the true class results (factor). This
#'   should be an unquoted column name.
#' @param estimate The column identifier for the predicted class results
#'   (factor). This should be an unquoted column name.
#' @param estimator One of "binary", "macro", "macro_weighted", or "micro" to
#'   specify the type of averaging to be done. Default is `NULL` which
#'   automatically selects based on the number of classes.
#' @param na_rm A logical value indicating whether NA values should be stripped
#'   before the computation proceeds. Default is `TRUE`.
#' @param case_weights The optional column identifier for case weights. This
#'   should be an unquoted column name. Default is `NULL`.
#' @param event_level A string either "first" or "second" to specify which level
#'   of truth to consider as the "event". Default is "first".
#' @param ... Not currently used.
#'
#' @return
#' A tibble with columns `.metric`, `.estimator`, and `.estimate` and 1 row of
#' values.
#'
#' For grouped data frames, the number of rows returned will be the same as the
#' number of groups.
#'
#' For `metric_name_vec()`, a single numeric value (or `NA`).
#'
#' @section Multiclass:
#'
#' Explanation of multiclass behavior and estimator types.
#'
#' @details
#' [metric_name()] is a metric that should be [maximized/minimized].
#' The output ranges from [min] to [max].
#'
#' The formula for binary classification is:
#'
#' \deqn{formula here}
#'
#' @examples
#' # Binary classification
#' df <- data.frame(
#'   truth = factor(c("yes", "yes", "no", "no")),
#'   estimate = factor(c("yes", "no", "yes", "no"))
#' )
#'
#' metric_name(df, truth, estimate)
#'
#' @export
```

## Testing

### Create your own test data

**Do NOT rely on yardstick internal test helpers like `data_altman()` or `data_three_class()`**

```r
# Simple test data patterns for numeric metrics
test_that("Calculations are correct", {
  df <- data.frame(
    truth = c(1, 2, 3, 4, 5),
    estimate = c(1.1, 2.2, 2.9, 4.1, 4.8)
  )

  # Calculate expected value by hand
  expected <- ...  # manual calculation

  expect_equal(
    metric_name_vec(df$truth, df$estimate),
    expected
  )
})

# Binary classification test data
test_that("Binary calculations are correct", {
  df <- data.frame(
    truth = factor(c("yes", "yes", "no", "no"), levels = c("yes", "no")),
    estimate = factor(c("yes", "no", "yes", "no"), levels = c("yes", "no"))
  )

  # TP=1, FP=1, TN=1, FN=1
  # Calculate expected value from formula
  expect_equal(
    metric_name_vec(df$truth, df$estimate),
    expected_value
  )
})

# Multiclass test data
test_that("Multiclass calculations are correct", {
  df <- data.frame(
    truth = factor(c("A", "A", "B", "B", "C", "C")),
    estimate = factor(c("A", "B", "B", "C", "C", "A"))
  )

  # 2 correct (A-A, B-B), 4 incorrect
  # Calculate per-class and averaged metrics
  expect_equal(
    metric_name_vec(df$truth, df$estimate),
    expected_value
  )
})
```

### Standard test suite template

```r
test_that("Perfect predictions give optimal value", {
  truth <- c(10, 20, 30, 40, 50)
  estimate <- c(10, 20, 30, 40, 50)

  expect_equal(
    metric_name_vec(truth, estimate),
    optimal_value  # 0 for minimize, 1 for maximize, etc.
  )
})

test_that("Case weights calculations are correct", {
  df <- data.frame(
    truth = c(1, 2, 3),
    estimate = c(1.5, 2.5, 3.5),
    case_weights = c(1, 2, 1)
  )

  # Calculate weighted expectation by hand
  # errors = (0.5, 0.5, 0.5)
  # weighted = (1*0.5 + 2*0.5 + 1*0.5) / (1+2+1) = 2/4 = 0.5
  expect_equal(
    metric_name_vec(df$truth, df$estimate, case_weights = df$case_weights),
    expected_weighted_value
  )
})

test_that("Works with hardhat case weights", {
  df <- data.frame(
    truth = c(1, 2, 3),
    estimate = c(1.5, 2.5, 3.5)
  )

  imp_wgt <- hardhat::importance_weights(c(1, 2, 1))
  freq_wgt <- hardhat::frequency_weights(c(1, 2, 1))

  expect_no_error(
    metric_name_vec(df$truth, df$estimate, case_weights = imp_wgt)
  )

  expect_no_error(
    metric_name_vec(df$truth, df$estimate, case_weights = freq_wgt)
  )
})

test_that("na_rm argument works", {
  expect_error(
    metric_name_vec(c(1, 2), c(1, 2), na_rm = "yes"),
    "must be a single logical value"
  )
})

test_that("NA handling with na_rm = TRUE", {
  truth <- c(1, 2, NA, 4)
  estimate <- c(1.1, NA, 3.1, 4.1)

  # Only use non-NA pairs: (1, 1.1) and (4, 4.1)
  expect_equal(
    metric_name_vec(truth, estimate, na_rm = TRUE),
    expected_value_from_complete_pairs
  )
})

test_that("NA handling with na_rm = FALSE", {
  truth <- c(1, 2, NA)
  estimate <- c(1, 2, 3)

  expect_equal(
    metric_name_vec(truth, estimate, na_rm = FALSE),
    NA_real_
  )
})

test_that("Data frame method works", {
  df <- data.frame(
    truth = c(1, 2, 3),
    estimate = c(1.1, 2.2, 3.3)
  )

  result <- metric_name(df, truth, estimate)

  expect_s3_class(result, "tbl_df")
  expect_equal(result$.metric, "metric_name")
  expect_equal(result$.estimator, "standard")
  expect_equal(nrow(result), 1)
})

test_that("Metric has correct attributes", {
  expect_equal(attr(metric_name, "direction"), "maximize")  # or "minimize"
  expect_equal(attr(metric_name, "range"), c(min_value, max_value))
})

test_that("Works with metric_set", {
  df <- data.frame(
    truth = c(1, 2, 3, 4, 5),
    estimate = c(1.1, 2.2, 2.9, 4.1, 5.2)
  )

  metrics <- yardstick::metric_set(metric_name, yardstick::rmse)

  result <- metrics(df, truth, estimate)

  expect_s3_class(result, "tbl_df")
  expect_equal(nrow(result), 2)
  expect_true("metric_name" %in% result$.metric)
  expect_true("rmse" %in% result$.metric)
})

test_that("Works with grouped data", {
  df <- data.frame(
    group = rep(c("A", "B"), each = 3),
    truth = c(1, 2, 3, 4, 5, 6),
    estimate = c(1.1, 2.1, 3.1, 4.1, 5.1, 6.1)
  )

  result <- df |>
    dplyr::group_by(group) |>
    metric_name(truth, estimate)

  expect_equal(nrow(result), 2)
  expect_equal(result$group, c("A", "B"))
})
```

## File naming conventions

- **Source files**: `R/[type]-[name].R`
  - Examples: `R/num-mae.R`, `R/class-accuracy.R`, `R/prob-roc_auc.R`
- **Test files**: `tests/testthat/test-[type]-[name].R`
  - Examples: `test-num-mae.R`, `test-class-accuracy.R`
- Use prefixes: `num-`, `class-`, `prob-`, `surv-`, etc.

## Common patterns

### Confusion matrix metrics

Use `yardstick_table()` to create weighted confusion matrices:

```r
xtab <- yardstick::yardstick_table(truth, estimate, case_weights = case_weights)
```

Access elements (assuming 2x2 binary case):
```r
# Determine event and control levels
event <- if (identical(event_level, "first")) {
  levels(truth)[1]
} else {
  levels(truth)[2]
}
control <- setdiff(levels(truth), event)

# True positives: predicted event, actual event
tp <- xtab[event, event]

# False positives: predicted event, actual control
fp <- xtab[control, event]

# False negatives: predicted control, actual event
fn <- xtab[event, control]

# True negatives: predicted control, actual control
tn <- xtab[control, control]
```

### Multiclass averaging

Three types:
- **macro**: Unweighted average across classes (equal weight for each class)
- **macro_weighted**: Weighted by class frequency in truth
- **micro**: Pool all classes, calculate once

Calculate weights manually:
```r
# For macro
wt <- rep(1, n_classes)

# For macro_weighted
class_counts <- colSums(confusion_matrix)
wt <- class_counts

# Then use weighted.mean()
weighted.mean(per_class_values, wt)
```

### Case weights

Always include `case_weights = NULL` parameter. Handle hardhat weights:

```r
if (is.null(case_weights)) {
  mean(values)
} else {
  # Convert hardhat weights to numeric
  wts <- if (inherits(case_weights, "hardhat_importance_weights") ||
             inherits(case_weights, "hardhat_frequency_weights")) {
    as.double(case_weights)
  } else {
    case_weights
  }
  weighted.mean(values, w = wts)
}
```

## Step-by-step workflow

1. ✅ **Initialize/verify package structure** (run prerequisite code first)
2. ✅ **Create implementation function** `metric_impl()` with core logic
3. ✅ **Create vector function** `metric_vec()` with validation and NA handling
4. ✅ **Create generic and data frame method** with proper NSE (`enquo`, `!!`)
5. ✅ **Add comprehensive documentation** (use templates above, no external templates)
6. ✅ **Create tests** (use simple inline test data, not yardstick internals)
7. ✅ **Run and verify:**

```bash
# From command line
Rscript -e "devtools::document()"  # Generate documentation
Rscript -e "devtools::load_all()"  # Load package
Rscript -e "devtools::test()"      # Run tests
```

## Common pitfalls to avoid

1. ❌ Using internal yardstick functions that aren't exported (`yardstick_mean`, `get_weights`, etc.)
2. ❌ Referencing `@template` tags that don't exist in your package
3. ❌ Using `metric_range()`, `metric_optimal()`, `metric_direction()` in docs (not exported)
4. ❌ Assuming yardstick test helpers like `data_altman()` are available
5. ❌ Forgetting to handle hardhat weight classes (convert with `as.double()`)
6. ❌ Not validating inputs with `check_*_metric()` functions
7. ❌ Not validating `na_rm` parameter explicitly
8. ❌ Creating package in wrong directory or forgetting to check DESCRIPTION exists
9. ❌ Using `UseMethod()` after calling `new_*_metric()` (do it before)
10. ❌ Forgetting to export the data frame method with `@export`

## Pre-flight checklist

Before creating a metric, verify:
- [ ] DESCRIPTION file exists (if not, run package initialization)
- [ ] yardstick, rlang, cli are in Imports section
- [ ] testthat is configured (tests/testthat/ directory exists)
- [ ] R/ directory exists
- [ ] You know which type of metric (numeric/class/prob)
- [ ] You have the formula/calculation method clearly defined
- [ ] You understand how to handle the specific metric type's requirements

## Troubleshooting

### "Cannot find function 'yardstick_mean'"
- **Cause**: Using internal function that's not exported
- **Fix**: Use `weighted.mean()` from base R instead

### "Cannot find template 'return'"
- **Cause**: Using `@template return` which doesn't exist in your package
- **Fix**: Use explicit `@return` documentation from templates above

### "metric_range is not an exported object"
- **Cause**: Using internal function in roxygen dynamic code
- **Fix**: Write static documentation, don't use `metric_range()`, etc.

### "No applicable method for metric_name"
- **Cause**: Calling `UseMethod()` after `new_*_metric()`
- **Fix**: Call `UseMethod()` first, then `new_*_metric()`, then define `.data.frame` method

### Tests fail with "object 'data_altman' not found"
- **Cause**: Using yardstick internal test data
- **Fix**: Create simple test data inline (see Testing section)

## Best practices

### Code style
- Use base pipe `|>` not `%>%`
- Use `\() ...` for single-line anonymous functions
- Use `function() {...}` for multi-line functions
- Keep tests minimal with few comments

### Documentation
- Wrap roxygen at 80 characters
- Use US English and sentence case
- Reference formulas and academic sources when available
- Include practical examples
- Don't use dynamic roxygen code with non-exported functions

### Performance
- Separate multiclass logic for reusability
- Apply weighting once at the end
- Use vectorized operations
- Cache confusion matrices when possible

### Error messages
- Use `cli::cli_abort()` for errors
- Use `cli::cli_warn()` for warnings
- Include argument names in braces: `{.arg truth}`
- Include code in braces: `{.code binary}`

## References

- [Custom performance metrics tutorial](https://www.tidymodels.org/learn/develop/metrics/)
- [Yardstick reference](https://yardstick.tidymodels.org/reference/)
- [Multiclass metrics vignette](https://yardstick.tidymodels.org/articles/multiclass.html)
- Source examples: look at yardstick source on GitHub (but remember many helpers are internal)
