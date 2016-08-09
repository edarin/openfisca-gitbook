# For developers


Follow these steps if you plan to develop on OpenFisca-France or OpenFisca-Core.

### Supported operating systems

The supported operating systems are GNU/Linux distributions (in particular Debian and Ubuntu), Mac OS X and Microsoft Windows.

Other OS should work if they can execute Python and NumPy.

You need [Python](https://www.python.org/downloads/) and the [Git](http://www.git-scm.com/downloads) tool to be installed on your system.

### Install

Be sure to have the last version of `pip` and `wheel` Python packages.

```
pip install --upgrade pip wheel
```

> If you see a permission error, retry with the "root" user.

We recommend to use a virtualenv with [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) in order to isolate OpenFisca packages from your system.

Setup a virtualenv named "openfisca":

```
pip install virtualenvwrapper
mkvirtualenv openfisca
```

> As mentioned in the virtualenvwrapper documentation, don't forget to add this line to your `.bashrc` file:
>
>     TODO

Then install OpenFisca:

```
mkdir -p ~/Dev/openfisca
cd ~/Dev/openfisca

git clone https://github.com/openfisca/openfisca-core.git
cd openfisca-core
pip install --editable .
python setup.py compile_catalog

cd ..

git clone https://github.com/openfisca/openfisca-france.git
cd openfisca-france
pip install --editable .
python setup.py compile_catalog

cd ..
```

## Test the installation

To test if OpenFisca-France is correctly installed:

```bash
python -m openfisca_france.tests.test_basics
```

It should display (this could take one or two minutes):

```
OpenFisca-France basic test was executed successfully.
```

This means that OpenFisca is correctly installed on your machine.

The next step for you is to read the [Coding the legislation](../coding-the-legislation/index.html) section to know how to write legislation.

> If you want to use your local installation as an API (instead of importing it in python scripts), install **OpenFisca Web API** with [these instructions](https://github.com/openfisca/openfisca-web-api#install).

