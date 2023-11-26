This is a python package for testing purpose.

# 1. Install `twine`
You’ll need `twine` to upload your project distributions to PyPI (see below).
```bash
python3 -m pip install twine
```

# 2. setup.py
1. it contains a global `setup()` function, which contains important arguments!
2. It’s the command line interface for running various commands that relate to packaging tasks. To get a listing of available commands, run `python3 setup.py --help-commands`

# 3. README.md
It covers the goal of the project.

# 4. MANIFEST.in
A `MANIFEST.in` is needed when you need to package additional files that are not automatically included in a source distribution.

# 5. LICENSE.txt
consulting [this website](https://choosealicense.com/)

# 6. <your package> folder
Although it’s not required, the most common practice is to include your Python modules and packages under a single top-level package that has the same name as your project, or something very close.	

# 7. Build project
```bash
# install `build` package
python3 -m pip install build

# build the package
python3 -m build --sdist
```

# 8. Create wheel for the project
A wheel is a built package that can be installed without needing to go through the “build” process. Installing wheels is substantially faster for the end user than installing from a source distribution.
- If your project is pure Python then you’ll be creating a “Pure Python Wheel”
- If your project contains compiled extensions, then you’ll be creating what’s called a *Platform Wheel* 

## 8.1 Pure wheel
This command pack your project into a `dist` folder:

```bash
python3 -m build --wheel
```
\* If you run build without --wheel or --sdist, it will build both files for you; this is useful when you don’t need multiple wheels.

## 8.2 Platform Wheels
Platform Wheels are wheels that are specific to a certain platform like Linux, macOS, or Windows, usually due to containing compiled extensions.
```bash
python3 -m build --wheel
```

# 9. Setup PyPI account in your PC
Create a `$HOME/.pypirc` file:
```bash
[pypi]
username = __token__
password = <the token value, including the `pypi-` prefix>
```

# 10. Check & upload to PyPI
The directory `dist/` was created under your project’s root directory. Check the generated distribution of the project:
```bash
# check the grammar in dist/ directory
twine check dist/*

# upload distribution
twine upload dist/*
```


---

Example: `setup.py`:
```python
from setuptools import find_packages, setup

setup(
    name='alice',
    version='1.0.0',
    description="CH's first python dependency",
    long_description="CH's first python dependency",
    long_description_content_type="CH's first python dependency",
    author='CH.He',
    author_email='Chuying.He@ibm.com',
    license='MIT',
    keywords='test project alice',
    packages=find_packages(),    # Set packages to a list of all packages in your project, including their subpackages, sub-subpackages, etc. Although the packages can be listed manually, setuptools.find_packages() finds them automatically. 
    install_requires=[
        'setuptools',
        'panda',
    ],
    # url='https://github.com/pypa/sampleproject',
    # include_package_data=True,
    # cmdclass={
    #     "package": Package
    # }
    # python_requires='>=3',    # if your project only runs on certain Python versions,
    # package_data={            # additional files need to be installed into a package. For example, documentation that might be of interest to programmers using the package.
    #     'sample': ['package_data.dat'],
    # },
    # data_files=[('my_data', ['data/data_file'])], # Sometimes you may need to place data files outside of your packages. The data_files directive allows you to do that. 

)
```

---
Example: `main.py`:
```python
def greet(name):
    return f"Hello, {name}!"

def add_numbers(a, b):
    return a + b

if __name__ == "__main__":
    print(greet("Alice"))
    print(add_numbers(5, 7))
```