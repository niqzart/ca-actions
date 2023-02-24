# CA-actions

### Usage
This repository has a reusable workflow in it, which you can use in any of your projects. Simplest way is to create a [workflow](https://docs.github.com/en/actions/using-workflows/about-workflows) (e.g. `.github/workflows/main.yml`), specify any [triggers](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow) and call this repository's workflow:
```yaml
jobs:
  prepare:
    uses: niqzart/ca-actions/.github/workflows/full-workflow.yml@v1.0
    with:
      # specify arguments here (see documentation below)
```

The simplest example of a fully-functional workflow file:
```yaml
name: My Workflow

on:
  push:  # will run on any push

jobs:
  prepare:
    uses: niqzart/ca-actions/.github/workflows/full-workflow.yml@v1.0
    with:
      # specify arguments here (see documentation below)
```

### Multiple problems
This workflow runs all steps sequentially (as parallel running is not supported at step level), but it will run all steps if install completes successfully, regardless of other step's results. I.e. if pytest fails, flake8, isort and others will still run and report their errors if there are any, so you can fix all of them at once

### Interpreter & requirements
- `python_version`: python version to use, see [setup-python.python-version](https://github.com/actions/setup-python/blob/main/docs/advanced-usage.md#using-the-python-version-input) for more info
- `package_manager`: name of the package installer/manager to use and cache, see [setup-python.cache](https://github.com/actions/setup-python/blob/main/docs/advanced-usage.md#caching-packages) for more info
- `install_command`: the command you use to install everything needed for running tests, formatters & linters. This should install dev-dependencies. The default is `pip install -r requirements.txt`

```yaml
with:
  python_version: 3.11
  package_manager: poetry  # use poetry instead of pip
  install_command: poetry install  # install all dependencies normally
  # ...
```

### Code path
If you want to run all liters and formatters over a specific directory in your repository (e.g. `./backend`), you can pass it (e.g. `backend`) into `lint_directory` parameter. This does not apply to pytest & coverage runs

```yaml
with:
  # ...
  lint_directory: app/backend  # only code in app/backend needs linting & format checking
```

### Config files
Normally a project will have just one config file with all settings inside of it (i.e. `pyproject.toml` and `setup.cfg`). If that's the case, you should specify a relative path from the repository's root to that file in the `config_file` parameter (e.g. `pyproject.toml` or `backend/setup.cfg`)

Sometimes there could be different config files for different linters (e.g. flake8 doesn't like `pyproject.toml`). In this case you should specify config files on per-step bases using `<step-name>_config_file` parameter, described in documentation for that step

```yaml
with:
  # ...
  config_file: backend/pyproject.toml  # all linters use pyproject.toml
  flake8_config_file: backend/.flake8  # but flake8 uses .flake8
```

### Extra params
To pass any extra parameters to specific steps, you can use `<step-name>_extra_params` parameter. These parameters will be passed to the command along with the defaults, so you can't override them (might be a future feature)

### Disabling steps
You can disable any step in this workflow by passing `<step-name>: false`:
```yaml
with:
  # ...
  mypy: false  # disables mypy
```

### Running tests with pytest
Step name: `pytest`

- `pytest_workdir`: choose a directory to run pytest in. Depending on you project structure, tests can be situated in any folder (e.g. `app/backend/tests`) and in order to get working imports you'll need to switch the working directory for the pytest run (e.g. use `app/backend`). Uses `.` by default
- `pytest_tests`: [specify which tests to run](https://docs.pytest.org/en/7.1.x/how-to/usage.html#specifying-which-tests-to-run)

### Checking code coverage
Step name: `coverage`

Checks code coverage after running tests. Will report the results to the output of this step and fail the pipeline if `fail-under` is set. For more info on report parameters, visit the [official coverage documentation](https://coverage.readthedocs.io/en/7.2.0/cmd.html#coverage-summary-coverage-report).

- `coverage_path`: relative path from `pytest_workdir` to code that needs to be checked for coverage

### Linting with flake8
Step name: `flake8`

Runs [flake8](https://flake8.pycqa.org/) and reports the results to the output of this step and fail if flake8 found any non-ignored errors. All configuration for the run can be specified in a configuration file, more info on configuration in the [official documentation](https://flake8.pycqa.org/en/latest/user/configuration.html)

To use flake8 plugins you should add them to your dev-requirements and pass those in the `install_command` argument [described earlier](#interpreter--requirements). Installed plugins will run automatically

### Check import ordering with isort
Step name: `isort`

Runs [isort](https://pycqa.github.io/isort/) in check mode to verify the import order and report any problems in the output this step. All configuration for the run can be specified in a configuration file, more info on configuration in the [official documentation](https://pycqa.github.io/isort/docs/configuration/options.html)

### Check formatting with black
Step name: `black`

Runs [black](https://black.readthedocs.io/en/stable/) in check mode to verify proper code formatting is present and report any problems in the output this step

### Lint with mypy
Step name: `mypy`

Runs [mypy](https://mypy.readthedocs.io/en/stable/) to check typing and report any problems in the output this step. All configuration for the run can be specified in a configuration file, more info on configuration in the [official documentation](https://mypy.readthedocs.io/en/stable/config_file.html)
