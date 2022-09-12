---
title: The Hypermodern Python Stack
description: The 2022 edition of an ultra modern Python toolbox.
date: 2022-09-04
slug: '/blog/hypermodern-python'
type: post
---

All Python developers are challenged by the size and velocity of the Python ecosystem.

From newbies finding their first workflow to senior developers keeping up with new packages, we all struggle with Python's large & dynamic ecosystem.

This post cuts through the fog with a Hypermodern Python toolbox:

- **Python 3.10** for better error messages,
- **pyenv** & **pyenv-virtualenv** for managing Python versions and virtual environments,
- **Poetry** for managing Python dependencies,
- **Black** and **isort** for formatting Python code,
- **mypy** for static type checking,
- **pydantic** for organizing & validating data,
- **Typer** for CLIs,
- **zxpy** for running shell commands inside Python,
- **rich** for pretty printing to the terminal.

## Python 3.10

Python 3.10 added better error messages - it's had a large positive impact on my Python development.

The code below has a mistake - we attempt to assign a value to the first element of `data`, mistakenly refering to the non-existent `datas` variable instead:

```python
#  mistake.py
data = [1, 4, 8]
datas[0] = 2
```

Running this broken code with older versions of Python, we get an error traceback that points out that the variable `datas` doesn't exist:

```shell
$ python --version
3.8.13

$ python mistake.py
Traceback (most recent call last):
  File "mistake.py", line 2, in <module>
    datas[0] = 2
NameError: name 'datas' is not defined
```

Python 3.10 takes this diagnosis one step further and also suggests a solution:

```shell
$ python --version
3.10.6

$ python mistake.py
Traceback (most recent call last):
  File "/Users/adam/hypermodern-python-2022/mistake.py", line 2, in <module>
    datas[0] = 2
NameError: name 'datas' is not defined. Did you mean: 'data'?
```

This may seem like a minor improvement - yet I miss it when working with older versions of Python.

## Versions & Virtual Environments with pyenv & pyenv-virtualenv

The hardest thing about learning Python is learning how to install & manage it. Even senior developers can struggle with it, especially if Python is not their main language.

Reliable workflows for creating & deleting virtual environments are a sign of an experienced Python developer.  

Working with Python requires being able to easily work:

1. with different versions of Python,
2. in different Python virtual environments.  

<br />

[pyenv](https://github.com/pyenv/pyenv) is a tool for managing different versions of Python. It's an alternative to using miniconda or installing Python from a downloaded installer.

pyenv can be used to manage many versions of Python - below three versions of Python are installed & managed by pyenv:

```shell-session
$ pyenv versions
3.7.9
3.8.13
3.10.5
```

Installing a new version of Python is as simple as `pyenv install`:

```shell-session
$ pyenv install 3.10.6
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Installing Python-3.10.6...
python-build: use tcl-tk from homebrew
python-build: use readline from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.10.6 to /Users/adam/.pyenv/versions/3.10.6
```

If you are having an trouble getting pyenv setup, take a look at this [installer script for Ubuntu](https://github.com/ADGEfficiency/dotfiles/blob/master/ubuntu/pyenv), [installer script for MacOS](https://github.com/ADGEfficiency/dotfiles/blob/master/macos/pyenv) and [compiler flags](https://github.com/ADGEfficiency/dotfiles/blob/master/macos/pyenv-flags).

After installing this version of Python, we can now create a virtual environment using this Python version.  [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) is a tool for managing virtual environments - it's an alternative to venv or miniconda.

No suprises that pyenv-virtualenv it plays well with our pyenv installation of 3.10.6 above:

```shell-session
$ pyenv virtualenv 3.10.6 default
```

We now have a new virtual environment - using the exact version of Python we need:

```shell-session
$ pyenv versions
3.7.9
3.8.13
3.10.5
3.10.6
3.10.6/envs/default
```

_Tip_ - create a `.python-version` file to automatically switch to a virtual env when you enter a directory.

## Python Package Management with Poetry

Once you have a fresh Python setup in a virtual environment, often you will next install additional Python packages needed for your work.

[Poetry](https://python-poetry.org/docs/basic-usage/) is a tool for managing Python dependencies - it's an alternative to pip (the Python package manager than comes with Python).

Pip uses two files to manage a Python package:

- `requirements.txt` - a list of Python dependencies,
- `setup.py` - a Python script that describes the package.

<br />

Poetry uses two different files:

- `pyproject.toml` to describe our Python package,
- `poetry.lock` to define and lock all dependencies - similar to `pip freeze`.

<br />

These two files are both often generated automatically - `poetry.lock` is only ever generated automatically.

Poetry has two ways to start a new project:

- `poetry new` - start a fresh project,
- `poetry init` - in an existing project.

<br />

We can create a `pyproject.toml` for a project in an interactive way by first installing Poetry with pip, then running `poetry init` to create a `pyproject.toml`:

```shell-session
$ pip install -q poetry; poetry init
This command will guide you through creating your pyproject.toml config.

Package name [general]:  general
Version [0.1.0]:
Description []:
```

After running through the interactive session (where we specify our Python version and add the package mypy), we end up with a `pyproject.toml`:

```toml
#  pyproject.toml
[tool.poetry]
name = "general"
version = "0.1.0"
authors = ["Adam Green <adam.green@adgefficiency.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.10"
mypy = "^0.971"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

At this point we have not installed our mypy dependency into our virtual environment - we can do so with `poetry install`:

```shell-session
$ poetry install
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Package operations: 4 installs, 0 updates, 0 removals

  • Installing mypy-extensions (0.4.3)
  • Installing tomli (2.0.1)
  • Installing typing-extensions (4.3.0)
  • Installing mypy (0.971)
```

The install operation also creates a `poetry.lock` file:

```shell-session
$ head -n 12 poetry.lock
[[package]]
name = "mypy"
version = "0.971"
description = "Optional static typing for Python"
category = "main"
optional = false
python-versions = ">=3.6"

[package.dependencies]
mypy-extensions = ">=0.4.3"
tomli = {version = ">=1.1.0", markers = "python_version < \"3.11\""}
typing-extensions = ">=3.10"
```

While Poetry is great, it's not time to say goodbye to pip (Poetry itself needs to be installed with pip).

We can export our dependencies to a pip compatiable `requirements.txt`:

```shell
$ poetry export -f requirements.txt > requirements.txt
```

_Watch out for_ - Poetry has the ability to create it's own virtual environments. It's common to turn this off in some environments - such as inside Docker images.

## Formatting with black & isort

[black](https://github.com/psf/black) & [isort](https://github.com/PyCQA/isort) are tools for automatically formatting Python code - they are alternatives to tools like autopep8.

One way to use black and isort is to run them from a terminal.

The code below is in `bad_format.py` poorly formatted:

```python
#  bad_format.py
data=[1, 4, 8]
datas[0] = 2
```

We can run black from a terminal, pointing at `bad_format.py`:

```shell
$ black bad_format.py
reformatted test.py

All done! ✨ 🍰 ✨
1 file reformatted.
```

The result is nicely formatted Python code:

```python
#  bad_format.py
data = [1, 4, 8]
datas[0] = 2
```

The code below has imports that are out of order alphabetically and grouped incorrectly:

```python
#  bad_imports.py
import pandas as pd
import random
import collections
data = [1, 4, 8]
datas[0] = 2
```

We can use isort to fix these imports:

```shell
$ isort bad_imports.py
Fixing /Users/adam/dss/notes/content/ideas/temp/test.py
```

Our fixed file:

```python
#  bad_imports.py
import collections
import random

import pandas as pd

data = [1, 4, 8]
datas[0] = 2
```

_Tip_ - it's common to run these formatters on file save or in continuous integration - consider adding a format on save to your text editor.

## Static Type Checking with mypy

[mypy](http://www.mypy-lang.org/) is a tool for enforcing type safety in Python code - it's an alternative to type declarations being only for decoration.

In some parts of the Python world, Python has undergone a transition similar to the Javascript to Typescript transition - type safe Python code is now the standard. Running mypy on Python codebases has become a sign of quality and pride for modern Python developers.

The code below has an error - we attempt to divide a `str` by `10`:

```python
def process(user):
    user['name'] / 10

user = {'name': 'alpha'}
process(user)
```

We can catch this error by running mypy - catching the error without actually executing the Python code:

```shell-session
$ mypy --strict mypy_error.py
mypy_error.py:1: error: Function is missing a type annotation
mypy_error.py:5: error: Call to untyped function "process" in typed context
Found 2 errors in 1 file (checked 1 source file)
```

These first errors are because our Python code has zero typing - let's add two type annotations:

- `user: dict[str,str]` - `user` is a dictionary with strings as keys and values,
- `-> None:` - the `process` function returns None.

<br />

```python
#  mypy_intermediate.py
def process(user: dict[str,str]) -> None:
    user['name'] / 10

user = {'name': 'alpha'}
process(user)
```

Running mypy again, it points out the error in our code:

```shell-sesson
$ mypy --strict mypy_intermediate.py
mypy_fixed.py:2: error: Unsupported operand types for / ("str" and "int")
Found 1 error in 1 file (checked 1 source file)
```

This is a test we can do without writing any specific test cases - very cool.

Static type checking becomes another layer of testing, that will catch some bugs that many unit test suites won't.  Static typing will check more paths than a single unit test often does - catching edge cases that would otherwise only occur in production.

## Organize data with pydantic

[pydantic](https://pydantic-docs.helpmanual.io/) is a tool for organizing and validating data in Python - it's an alternative to using dictionaries or dataclasses.

pydantic is part of Python's typing revolution - pydantic's ability to create custom types makes writing typed Python a joy.

pydantic uses Python type hints to define data types. Imagine we want a user with a `name` and `id`:

```python
import uuid

users = [
    {'name': 'alpha', 'id': str(uuid.uuid4())},
    {'name': 'beta'},
    {'name': 'omega', 'id': 'invalid'}
]
```

We could model this with pydantic - introducing a class that inherits from `pydantic.BaseModel`:

```python
import uuid
import pydantic

class User(pydantic.BaseModel):
    name: str
    id: str = None

users = [
    User(name='alpha', 'id'= str(uuid.uuid4())),
    User(name='beta'),
    User(name='omega', id='invalid'),
]
```

A strength of pydantic is validation - we can introduce some validation of our user ids - below checking that the `id` is a valid GUID - otherwise setting to `None`:

```python
import uuid
import pydantic

class User(pydantic.BaseModel):
    name: str
    id: str = None

    @pydantic.validator('id')
    def validate_id(cls, user_id):
        try:
            user_id = uuid.UUID(user_id, version=4)
            print(f"{user_id} is valid")
            return user_id
        except ValueError:
            print(f"{user_id} is invalid")
            return None

users = [
    User(name='alpha', id= str(uuid.uuid4())),
    User(name='beta'),
    User(name='omega', id='invalid'),
]
[print(user) for user in users]
```

Running the code above, our pydantic model has rejected one of our ids:

```shell-session
$ python pydantic_eg.py
45f3c126-1f50-48bf-933f-cfb268dca39a is valid
invalid is invalid
name='alpha' id=UUID('45f3c126-1f50-48bf-933f-cfb268dca39a')
name='beta' id=None
name='omega' id=None
```

_Tip_ - you can generate Typescript types from pydantic models - making it possible to share the same data structures with your Typescript frontend and Python backend.

## Create CLIs with Typer

[Typer](https://typer.tiangolo.com/) is a tool for building command line interfaces (CLIs) using type hints - it's an alternative to argparse.

We can build a Python CLI with Poetry and Typer by first creating a Python package with Poetry, adding `typer` as a dependency).

Here we use `poetry new`, which will create more files & folders than `poetry init`:

```shell-session
$ poetry new general
$ tree
.
└── general
    ├── README.md
    ├── general
    │   └── __init__.py
    ├── pyproject.toml
    └── tests
        └── __init__.py
```

Then add a Python file `./general/cli.py` with our typer CLI:

```python
#  general/cli.py
import typer


def main(name: str) -> None:
    print(f"Hello {name}")


if __name__ == "__main__":
    typer.run(main)
```

We can now run this CLI by running `python general/cli.py`:

```shell-session
$ python general/cli.py omega
Hello omega
```

Typer gives us a nice `--help` for free:

```shell-session
$ python general/cli.py --help
Usage: cli.py [OPTIONS] NAME

Arguments:
  NAME  [required]

Options:
  --install-completion  Install completion for the current shell.
  --show-completion     Show completion for the current shell, to copy it or
                        customize the installation.
  --help                Show this message and exit.
```

We can take this one step further - by adding a script to our `pyproject.toml` - `general-cli` will point towards the `main` function in `general.cli`:

```toml
#  pyproject.toml
[tool.poetry.scripts]
general-cli = "general.cli:main"
```

This then allows us to run our Typer CLI using `poetry run general-cli`:

```shell-session
$ poetry run general-cli zeta
hello zeta
```

## Run Shell Commands in Python with zxpy

[zxpy](https://github.com/tusharsadhwani/zxpy) is a tool for running shell commands inside Python.

We will use the [Github CLI](https://cli.github.com/manual/) as a source of shell commands.

The Github CLI is a nice way to get data about your code on Github - for example here we:

```shell-session
$ gh search issues --repo mypy --json title | jq > issues.json
$ head -n 7 issues.json
[
  {
    "title": "Not evaluating Union[X, Y] from Type[Union[X, Y]] over (Type[T]) -> T function"
  },
  {
    "title": "Detect `Any` used as a metaclass"
  },
```

This JSON array (or list of dictionaries in Python) is data we want to work on in Python. We could read the `issues.json` file in Python - this would involve running the shell command and Python interpreter separatly.

With zxpy we can run the shell command right in Python - using the `~"shell-command"` syntax:

```python
#  zxpy_eg.py
import json

issues = json.loads(~"gh search issues --repo python/mypy --json title")
print(f"{len(issues)} issues")
print(f" first {issues[0]}")
print(" last {issues[-1]}")
```

```shell-session
$ zxpy zxpy_eg.py
30 issues
 first {'title': 'Cannot infer type of generic attributes in `match` statements when inheritance is involved'}
 last {'title': 'Parent modules are added as a dependency'}
```

*Tip* - f-strings in zxpy are written `~f"gh search issues --repo {repo}`.


## Pretty Print with Rich

[Rich](https://rich.readthedocs.io/en/stable/) is a tool for printing pretty text to a terminal - it's an alternative to the monotone terminal output of most Python programs.

```shell-session
import rich

user = {'name': 'omega', 'id': 'invalid'}
print(f" normal printing\nuser {user}\n")
rich.print(f" :wave: [bold blue]rich[/] [green]printing[/]\nuser {user}\n")
```

<center>
  <img src="/hypermodern-python/rich.png" width="80%" align="center">
  <br />
</center>

If you are happy with Rich you can simplify your code with:

```python
from rich import print
print('this will be printed with rich :clap:')
```

<center>
  <img src="/hypermodern-python/rich2.png" width="80%" align="center">
  <br />
</center>


## Summary

Our hypermodern Python toolbox is:

- **Python 3.10** for better error messages,
- **pyenv** & **pyenv-virtualenv** for managing Python versions and virtual environments,
- **Poetry** for managing Python packages & dependencies,
- **Black** and **isort** for formatting Python code,
- **mypy** for static type checking,
- **pydantic** for organizing & validating data,
- **Typer** for CLIs,
- **zxpy** for running shell commands inside Python,
- **rich** for pretty printing to the terminal.

---

Checkout other Python related posts on [pathlib versus os.path](https://datasciencesouth.com/blog/python-file-system) and [Pandas & Matplotlib for New Data Scientists](https://datasciencesouth.com/blog/python-libraries-new-data-scientists).