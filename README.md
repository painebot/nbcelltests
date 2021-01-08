<img src="https://raw.githubusercontent.com/jpmorganchase/nbcelltests/main/docs/logo.png" width=400></img>


Cell-by-cell testing for production Jupyter notebooks in JupyterLab

[![Build Status](https://github.com/jpmorganchase/nbcelltests/workflows/Build%20Status/badge.svg?branch=main)](https://github.com/jpmorganchase/nbcelltests/actions?query=workflow%3A%22Build+Status)
[![codecov](https://codecov.io/gh/jpmorganchase/nbcelltests/branch/master/graph/badge.svg?token=wWAQ6QqP6M)](https://codecov.io/gh/jpmorganchase/nbcelltests)
[![Docs](https://img.shields.io/readthedocs/nbcelltests.svg)](https://nbcelltests.readthedocs.io)
[![PyPI](https://img.shields.io/pypi/l/nbcelltests.svg)](https://pypi.python.org/pypi/nbcelltests)
[![PyPI](https://img.shields.io/pypi/v/nbcelltests.svg)](https://pypi.python.org/pypi/nbcelltests)
[![npm](https://img.shields.io/npm/v/jupyterlab_celltests.svg)](https://www.npmjs.com/package/jupyterlab_celltests)


# Overview
`nbcelltests` is designed for writing tests for linearly executed notebooks. Its primary use is for unit testing reports. 

## Installation
Python package installation: `pip install nbcelltests`

To use in JupyterLab, you will also need the lab and server extensions. Typically, these are
automatically installed alongside nbcelltests, so you should not need to do anything special
to use them. The lab extension will require a rebuild of JupyterLab, which you'll be prompted
to do on starting JupyterLab the first time after installing celltests (or you can do manually
with `jupyter lab build`). Note that you must have node.js installed (as for any lab extension).

To see what extensions you have, check the output of `jupyter labextension list` (look for
`jupyterlab_celltests`), and `jupyter serverextension list` (look for `nbcelltests`).
If for some reason you need to manually install the extensions, you can do so as follows:

```bash
jupyter labextension install jupyterlab_celltests
jupyter serverextension enable --py nbcelltests
```

(Note: if using in an environment, you might wish to add `--sys-prefix` to the `serverextension` command.)

## "Linearly executed notebooks?"
When converting notebooks into html/pdf/email reports, they are executed top-to-bottom one time, and are expected to contain as little code as reasonably possible, focusing primarily on the plotting and markdown bits. Libraries for this type of thing include [Papermill](https://github.com/nteract/papermill), [JupyterLab Emails](https://github.com/timkpaine/jupyterlab_email), etc. 

## Doesn't this already exist?
[Nbval](https://github.com/computationalmodelling/nbval) is a great product (we leverage it in this project) and I recommend using it for notebook regression tests. But it only allows for testing for unexpected failures or simple output equality tests.

## So why do I want this again?
This doesn't necessarily help you if your data sources go down, but its likely you'll notice this anyway. Where this comes in handy is:

- when the environment (e.g. package versions) are changing in your system
- when you play around in the notebook (e.g. nonlinear execution) but aren't sure if your reports will still generate
- when your software lifecycle systems have a hard time dealing with notebooks (can't lint/audit them as code unless integrated nbdime/nbconvert to script, tough to test, tough to ensure what works today works tomorrow)

## So what does this do?
Given a notebook, you can write mocks and assertions for individual cells. You can then generate a testing script for this notebook, allowing you to hook it into your testing system and thereby provide unittests of your report. 

## Writing tests
When you write tests for a cell, we create a new method on a `unittest` class corresponding to the index of your cell, and including the cumulative tests for all previous cells (to mimic what has happened so far in the notebook's linear execution). You can write whatever mocking and asserts you like, and can call `%cell` to inject the contents of the cell into your test. 
![](https://raw.githubusercontent.com/timkpaine/nbcelltests/main/docs/demo.gif)
The tests themselves are stored in the cell metadata, similar to celltags, slide information, etc. 

## Running tests
You can run the tests offline from an `.ipynb` file, or you can execute them from the browser and view the results of `pytest-html`'s html plugin.
![](https://raw.githubusercontent.com/timkpaine/nbcelltests/main/docs/demo2.gif)

## Extra Tests
- Max number of lines per cell
- Max number of cells per notebook
- Max number of function definitions per notebook
- Max number of class definitions per notebook
- Percentage of cells tested

## Example
In the committed `examples/Example.ipynb` notebook, but modified so that cell 0 has its import statement copied 10 times (to trigger test and lint failures):


### Tests
The following output is generated by running `nbcelltests test examples/Example.ipynb`
```bash
examples/_Example_test.py::TestNotebook::test_cell_coverage PASSED                                                                               [ 20%]
examples/_Example_test.py::TestNotebook::test_code_cell_1 PASSED                                                                                 [ 40%]
examples/_Example_test.py::TestNotebook::test_code_cell_2 PASSED                                                                                 [ 60%]
examples/_Example_test.py::TestNotebook::test_code_cell_3 PASSED                                                                                 [ 80%]
examples/_Example_test.py::TestNotebook::test_code_cell_4 PASSED                                                                                 [100%]
```
### Lint
The following output is generated by running `nbcelltests lint examples/Example.ipynb`

```bash
PASSED: Checking lines in cell (max=10; actual=2) (Cell 1)
PASSED: Checking lines in cell (max=10; actual=1) (Cell 2)
PASSED: Checking lines in cell (max=10; actual=1) (Cell 3)
PASSED: Checking lines in cell (max=10; actual=1) (Cell 4)
PASSED: Checking cells per notebook (max=10; actual=4)
PASSED: Checking functions per notebook (max=10; actual=0)
PASSED: Checking classes per notebook (max=10; actual=0)
FAILED: Checking lint:
	examples/Example.ipynb (in /var/folders/s3/1mjw0y192zg3450tkkn1yfnm0000gn/T/tmpp91li59p.py):32:1: F821 undefined name 'test3'
	examples/Example.ipynb (in /var/folders/s3/1mjw0y192zg3450tkkn1yfnm0000gn/T/tmpp91li59p.py):32:6: W291 trailing whitespace
```

NB: In jupyterlab, notebooks will be lint checked in-process using the version of
python that is running jupyter lab itself. A notebook intended to be
run with a Python 2 kernel could therefore generate syntax errors
during lint checking.
