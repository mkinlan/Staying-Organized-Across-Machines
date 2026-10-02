# Installing `myTemplates`

A local R package that creates a standard data-project folder structure with one command.

## 1. First, make sure your R package library location is where you want it to be (one-time, per machine)

Windows often installs R into `C:/Program Files/...`, which normal user accounts
can't write to. This causes `install.packages()` to fail with:

```
Warning in install.packages(...) :
  'lib = "C:/Program Files/R/..."' is not writable
```

Fix it by pointing R to a personal library folder instead.

**Step 1 — Find your personal library path:**
```r
Sys.getenv("R_LIBS_USER")
```
This prints something like:
```
C:\Users\<you>\AppData\Local/R/win-library/4.x
```

**Step 2 — Create that folder if it doesn't exist:**
```r
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
```

**Step 3 — Save it permanently in `.Renviron`.**
⚠️ Use forward slashes only — backslashes get stripped when `.Renviron` is parsed.
```r
writeLines(
  paste0("R_LIBS_USER=", gsub("\\\\", "/", Sys.getenv("R_LIBS_USER"))),
  "~/.Renviron"
)
```

**Step 4 — Restart Positron/RStudio completely**, then confirm:
```r
Sys.getenv("R_LIBS_USER")
```
This should print the same path with no missing characters.

## 2. Install `devtools` (if not already installed)

```r
install.packages("devtools")
```

## 3. Get the `myTemplates` package onto this machine

Clone or download this repo, then unzip/place it somewhere permanent, e.g.:
```
C:/Users/<you>/myTemplates
```

⚠️ Make sure `DESCRIPTION`, `R/`, and `inst/` end up **directly inside** that
folder — not nested inside an extra `myTemplates/myTemplates/` folder.

## 4. Install the package

```r
devtools::install("C:/Users/<you>/myTemplates")
```

## 5. (Optional) Auto-load the quick helper on every R startup

Open your `.Rprofile`:
```r
usethis::edit_r_profile()
```

Add this line inside the file that opens (then save it, e.g. Ctrl+S):
```r
source("C:/Users/<you>/myTemplates/new_project.R")
```

Restart Positron.

## 6. Use it

```r
# Direct call — builds the folder structure at the given path
myTemplates::data_project_template("C:/path/to/new-project")

# Or, if you set up the helper in step 5:
new_project("my-new-project")
```

## Folder structure this creates

```
your-project/
├── README.md
├── .gitignore
├── data/
│   ├── raw/
│   ├── interim/
│   └── processed/
├── notebooks/
├── src/
├── outputs/
│   ├── figures/
│   └── tables/
├── docs/
│   ├── data_dictionary.md
│   ├── decisions.md
│   └── project_notes.md
└── tests/
```

## Troubleshooting

| Symptom | Fix |
|---|---|
| `'lib = ...' is not writable` | Steps 1–2 weren't completed, or Positron wasn't restarted after editing `.Renviron` |
| `Could not find package root` | The zip/folder is nested one level too deep — move `DESCRIPTION`/`R`/`inst` up |
| `there is no package called 'myTemplates'` | Package installed before the library path fix — rerun `devtools::install(...)` after fixing `R_LIBS_USER` |
| `.Renviron` path looks scrambled when read back | You used backslashes — rewrite using forward slashes only |