# Automate file management in R with the {fs} package
Jadey Ryan
2024-09-24

Imagine you’re working on a project, and your folder is filled with
subfolders and files named in different styles: some in uppercase, some
in lowercase, and with a mixture of spaces, dashes, and underscores.

This disorganized chaos can make it hard to find anything, but the
[`{fs}` package](https://fs.r-lib.org/index.html) lets you quickly
rename and organize everything to a consistent case and format. This
guide will show you how to use `{fs}` to automate tedious tasks like
creating, renaming, moving, and deleting files and folders with just a
few lines of code.

`{fs}` has very nice, consistent syntax with the following four main
categories of functions:

- `path_` for manipulating and constructing paths
- `file_` for files
- `dir_` for directories (this guide uses the terms *folders* and
  *directories* interchangeably)
- `link_` for links

Directories and links are special types of files, so `file_` functions
will generally also work when applied to a directory or link.

Throughout this guide, we’ll use `{fs}` for file/directory handling and
`stringr` for string manipulation.

``` r
library(fs)
library(stringr)
```

## Create an example messy folder

To demonstrate the power of `{fs}` for efficient file management, we
first need to create a messy folder with inconsistently named files.

Let’s first create our example directory with nothing in it using
`dir_create()`, and then check that it exists with `dir_exists()`.

``` r
dir_create("fs-example")
dir_exists("fs-example")
#> fs-example 
#>       TRUE
```

Next, we’ll add some poorly named files with inconsistent case and
delimiters (spaces, underscores, and dashes) without any subfolder
organization. All `{fs}` functions are vectorized, so we can pass
multiple file names to the `path` argument of `file_create()` to create
all our files at once in our `fs-example` folder.

``` r
files <- c(
  "RAW-DATA.csv",
  "Clean Data.csv",
  "1-analysis.R",
  "2_visualization.R",
  "Very important report.qmd",
  "Very important report.pdf",
  "Very important report_final.pdf"
)

file_create("fs-example", files)
```

A cool safety feature of `{fs}` is the `*_create()` functions will not
overwrite existing files or folders.

``` r
dir_create("fs-example", "1-analysis.R")
#> Error: [EEXIST] Failed to make directory 'fs-example/1-analysis.R': file already exists

file_create("fs-example")
#> Error: [EISDIR] Failed to open 'fs-example': illegal operation on a directory
```

## Standardize file name case and format

Now we have a messy folder to clean up! Let’s take a look using one of
my favorite functions: `dir_tree()`. It provides a nicely formatted
directory tree and is a great sanity check before and after file
manipulation.

``` r
dir_tree("fs-example")
#> fs-example
#> ├── 1-analysis.R
#> ├── 2_visualization.R
#> ├── Clean Data.csv
#> ├── RAW-DATA.csv
#> ├── Very important report.pdf
#> ├── Very important report.qmd
#> └── Very important report_final.pdf
```

We’re ready to make our file names more consistent in case and format!
[Allison Horst](https://allisonhorst.com) created this beautiful graphic
that demonstrates the different case styles typically used in file
naming conventions and programming in general.

<img src="ah-case.png"
data-fig-alt="Cartoon representations of common cases in coding. A snake screams &quot;SCREAMING_SNAKE_CASE&quot; into the face of a camel (wearing ear muffs) with &quot;camelCase&quot; written along its back. Vegetables on a skewer spell out &quot;kebab-case&quot; (words on a skewer). A mellow, happy looking snake has text &quot;snake_case&quot; along it." />

Let’s get all the files in our directory into a character vector called
`files_old` using `dir_ls()`. Then, we’ll use `str_replace_all()` to
standardize the file name case and format to snake case, which uses all
lower case and dashes as the delimiter.

``` r
files_old <- dir_ls("fs-example")

files_old
#> fs-example/1-analysis.R
#> fs-example/2_visualization.R
#> fs-example/Clean Data.csv
#> fs-example/RAW-DATA.csv
#> fs-example/Very important report.pdf
#> fs-example/Very important report.qmd
#> fs-example/Very important report_final.pdf

files_new <- files_old |>
  str_to_lower() |>
  str_replace_all(" ", "-") |>
  str_replace_all("_", "-")

files_new
#> [1] "fs-example/1-analysis.r"                   
#> [2] "fs-example/2-visualization.r"              
#> [3] "fs-example/clean-data.csv"                 
#> [4] "fs-example/raw-data.csv"                   
#> [5] "fs-example/very-important-report.pdf"      
#> [6] "fs-example/very-important-report.qmd"      
#> [7] "fs-example/very-important-report-final.pdf"
```

To actually rename these files, we’ll use `file_move()`, and then print
the file names again to check it worked.

``` r
file_move(files_old, files_new)

dir_ls("fs-example")
#> fs-example/1-analysis.r
#> fs-example/2-visualization.r
#> fs-example/clean-data.csv
#> fs-example/raw-data.csv
#> fs-example/very-important-report-final.pdf
#> fs-example/very-important-report.pdf
#> fs-example/very-important-report.qmd
```

Our files are looking much more consistent and readable!

## Create subfolders and move files

To make the directory easier to navigate, we can create subfolders for
our R scripts, data, and reports.

``` r
subdirs <- c("R", "data", "reports")

dir_create("fs-example", subdirs)

dir_tree("fs-example")
#> fs-example
#> ├── 1-analysis.r
#> ├── 2-visualization.r
#> ├── R
#> ├── clean-data.csv
#> ├── data
#> ├── raw-data.csv
#> ├── reports
#> ├── very-important-report-final.pdf
#> ├── very-important-report.pdf
#> └── very-important-report.qmd
```

We can use `dir_ls()` with the `glob` argument to filter the file paths
to just one file type so we can easily move them into the appropriate
subfolder with `file_move()`.

A [glob](https://en.wikipedia.org/wiki/Glob_(programming)) is a pattern
to expand wildcards (an asterisk `*`) in a filepath. Since globs are
case-sensitive, in our first example below, we combine two globs: `*.r`
and `*.R` with the `|` OR operator to get the filepaths of all R
scripts, regardless if the extension is upper or lower case.

``` r
r_files <- dir_ls("fs-example", glob = "*.r|*.R")
data_files <- dir_ls("fs-example", glob = "*.csv")
reports <- dir_ls("fs-example", glob = "*.qmd|*.pdf")

file_move(r_files, "fs-example/R")
file_move(data_files, "fs-example/data")
file_move(reports, "fs-example/reports")
```

Let’s take a look at our directory tree to check our files are named and
organized as expected.

``` r
dir_tree("fs-example")
#> fs-example
#> ├── R
#> │   ├── 1-analysis.r
#> │   └── 2-visualization.r
#> ├── data
#> │   ├── clean-data.csv
#> │   └── raw-data.csv
#> └── reports
#>     ├── very-important-report-final.pdf
#>     ├── very-important-report.pdf
#>     └── very-important-report.qmd
```

Beautiful! Much more tidy!

## Delete files and directory

Lastly, let’s use `dir_delete()` to remove our entire example folder. If
we only wanted to delete a file or two, we could instead use
`file_delete()`.

``` r
dir_exists("fs-example")
#> fs-example 
#>       TRUE

dir_delete("fs-example")

dir_exists("fs-example")
#> fs-example 
#>      FALSE
```

## Use functions to easily repeat this process

As a bonus, we can package this code into two functions that we can
incorporate into our workflow or reuse for many folders:

`clean_file_names()` will rename all files in a given directory to be
all lowercase and use only dashes as the delimiter.

``` r
clean_file_names <- function(folder) {
  files_old <- dir_ls(folder)

  files_new <- files_old |>
    str_to_lower() |>
    str_replace_all(" ", "-") |>
    str_replace_all("_", "-")

  file_move(files_old, files_new)
}

# Run the function
clean_file_names("fs-example")
```

`organize_files()` will create new folders for R scripts, data, and
reports, and then move all files with the appropriate extensions into
those subfolders.

Here, we change the code in the `file_move()` functions to make it work
within our custom function’s new `folder` argument. We use `str_glue()`
to glue together the folder and subfolder, which creates the string
`"fs-example/R"`. However, this string is not a filepath, so we have to
wrap this with the `path()` function (also from `{fs}`) to turn the
string into a complete filepath.

``` r
organize_files <- function(folder) {
  subdirs <- c("R", "data", "reports")

  dir_create(folder, subdirs)

  r_files <- dir_ls(folder, glob = "*.r|*.R")
  data_files <- dir_ls(folder, glob = "*.csv|*.xlsx")
  reports <- dir_ls(folder, glob = "*.qmd|*.pdf")

  file_move(r_files, path(str_glue("{folder}/R")))
  file_move(data_files, path(str_glue("{folder}/data")))
  file_move(reports, path(str_glue("{folder}/reports")))
}

# Run the function
organize_files("fs-example")
```

You can easily adapt these two functions for your own work by changing
the `str_` functions and arguments in `clean_file_names()` to use your
preferred case and delimiter. Or, in `organize_files()`, you can create
other subdirectories, call them something different, or add more file
extensions to be moved into those subfolders.

**Caution**: when adapting the above functions, make a copy of the
folder and files you want to rename and organize to experiment with.
There is no “undo” button, so test the function first on this copy to be
sure you are happy with the results.

## Conclusion

We hope this guide helps you with your file cleaning and organizing
endeavors! While we focused mostly on creating, renaming, moving, and
deleting files and directories, `{fs}` can also help you
programmatically query and change file permissions and metadata. Learn
more about these functions through the [package documentation
website](https://fs.r-lib.org/index.html) and [Tidyverse blog
post](https://www.tidyverse.org/blog/2018/01/fs-1.0.0/).
