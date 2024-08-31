# Create and Release a Python Private Package in GitHub: A Comprehensive Guide

# Introduction

As software engineers, we often find ourselves reusing code across different modules and projects. But let's face it, this repetition creates a challenge: when we need to adjust or fix that code, we have to make the same changes in multiple places. For those of us who value efficiency and automation, the solution is clear - create a separate package that can be installed and used across our projects.
However, when dealing with confidential code, we can't simply publish our package on public repositories like PyPI. Instead, we need to deploy it to a private repository such as GitHub or GitLab. This approach allows us to maintain security while still benefiting from the convenience of a reusable package.

In this tutorial, we’ll guide you through the process of:

1. Creating a Python package
2. Deploying the package to a private repository (GitHub)
3. Installing the package in a virtual environment (venv)

By following these steps, you'll be able to reduce code duplication and simplify maintenance of shared code across your projects.

Note: DRY doesn't just stand for "Don't Repeat Yourself" - it's also a lifestyle choice.

![meme.jpg](https://github.com/ABDELLAH-Hallou/my_package/blob/master/meme.jpg)

## 1. Setting Up Your Project Structure

First, let's set up a basic project structure for our Python package:

```python
my-package/
├── my_package/
│   ├── __init__.py
│   └── module1.py
├── setup.py
├── build.pipeline.yml
├── requirements.txt
├── .gitignore
├── README.md
├── MANIFEST.in
└── LICENSE
```

Let's break down the anatomy of our private Python package. Each file and directory plays a crucial role in making our package functional and installable:

- `my-package/`: This is the root directory of our project. It's like a house that contains all the rooms (files) we need.
- `my_package/`: This subdirectory is where the actual Python code lives. It's named the same as our package for clarity.
    - `__init__.py`: This file makes Python treat the directory as a package. It can be empty or can execute the initialization code for the package.
    - `module1.py`: This is where we put our main code. You can have multiple module files depending on your package's complexity.
- `setup.py`: Think of this as the instruction manual for our package. It contains metadata about our package (like its name and version) and lists its dependencies. This file is essential for making our package installable via pip.
- `requirements.txt`: This file lists all the external Python packages our project depends on. It's like a shopping list for pip, telling it exactly what to install to make our package work.
- `README.md`: This is the welcome mat of our project. It's usually the first thing people see when they visit our GitHub repository, so we use it to explain what our package does, how to install it, and how to use it.
- `.gitignore`: This file tells Git which files or directories to ignore. It's handy for keeping compiled code, temporary files, or sensitive information out of version control.
- `LICENSE`: This file specifies how others can use, modify, or distribute our package. It's crucial for open-source projects and helps protect our work.
- `MANIFEST.in`: This file is used to include non-Python files in our package distribution. If we have data files, documentation, or other resources that need to be included, we list them here.
- `build.pipeline.yml`: This file defines our Continuous Integration/Continuous Deployment (CI/CD) pipeline. It automates tasks like running tests and building the package when we push changes to our GitHub repository.

## 2. Creating the Package Code

Let's create a simple module within our package. In `my_package/module1.py`:

```python
class Hello:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return f"Hello, {self.name}!"
```

In `my_package/__init__.py`, we'll import our module:

```python
from .module1 import Hello
```

## 3. Creating `setup.py`

The `setup.py` file is crucial for packaging our project. Here's a basic example:

```python
from setuptools import setup, find_packages

with open('requirements.txt') as f:
    requirements = f.read().splitlines()

setup(
    name="my_package",
    version="0.1",
    include_package_data=True,
    python_requires='>=3.8',
    packages=find_packages(),
    setup_requires=['setuptools-git-versioning'],
    install_requires=requirements,
    author="Abdellah HALLOU",
    author_email="abdeallahhallou33@gmail.com",
    description="A short description of your package",
    long_description=open('README.md').read(),
    long_description_content_type="text/markdown",
    classifiers=[
        "Programming Language :: Python :: 3.8",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    version_config={
       "dirty_template": "{tag}",
    }
)
```

## 4. Creating `requirements.txt`

In our `requirements.txt` file, we include the necessary dependencies for building and distributing our package:

```python
setuptools==69.2.0
wheel
twine
```

## 5. Building and Installing Your Package

Install the requirements. To keep things simple, we will use the Python virtual environment.

```bash
python -m venv env
source env/bin/activate # for linux and mac
./env/Scripts/activate # for windows
pip install -r requirements.txt
```

To build our package:

```bash
python setup.py sdist bdist_wheel
```

To install our package locally for testing:

```bash
pip install -e .
```

You can commit your work and ignore the folders using `.gitignore` file:

 https://github.com/github/gitignore/blob/main/Python.gitignore

## 6. Publishing the Package **on GitHub with a tag**

To publish the package, first, create a `build.pipeline.yml` file at the root of the project `my-package/` and commit it. The deployment will be done with `twine`, the library that we installed before: 

```yaml
name: Python package

on:
  push:
    tags:
      - '*'

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.7", "3.8", "3.9", "3.10", "3.11"]
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python 3.8
        uses: actions/setup-python@v5
        with:
          python-version: 3.8
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if  [ -f requirements.txt ]; then pip install -r requirements.txt; fi
      - name: Build package
        run: |
          python setup.py sdist bdist_wheel
      - name: Publish package
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          pip install twine
          python -m twine upload --repository-url https://api.github.com/orgs/${{ github.repository_owner }}/packages/pypi/upload dist/*
```

If you need to include non-Python files with your module installation, you can use a `MANIFEST.in` file. This file specifies which additional files should be included in your package distribution.

```python
# To include all files and subdirectories within the my_package/directory_name directory
recursive-include my_package/directory_name *
# To include a specific file in your package.
include  my_package/file_name
```

Then upload the package:

```bash
git tag 0.0.1 -m "New Release"
git push origin 0.0.1
```

## 7. **Install the Package**

Create an access token:

- Go to [Settings](https://github.com/settings/profile) > [Developer Settings](https://github.com/settings/apps) > **Personal access tokens (classic)** > **Generate new token**.
- Ensure you check the **`write:packages`** scope to grant the necessary permissions.

Once you have your token, keep it secure, as you'll need it to install your package.

On your machine, you can install your private package using the following template:

```bash
# template
pip install git+https://{{ your access token }}@github.com/{{ username }}/{{ repository name}}.git@{{ tag/version }}#egg={{ package name }}
# example
pip install git+https://{{ your access token }}@github.com/ABDELLAH-Hallou/my-package-repo.git@0.0.1#egg=my_package
```

## Conclusion

Well done, you know now how to create and deploy your own private packages with Python on GitHub.
