[![Python](https://github.com/stan-dev/posteriordb-python/actions/workflows/push.yml/badge.svg)](https://github.com/stan-dev/posteriordb-python/actions/workflows/push.yml)

# `posteriordb-python`: a Python library to work with `posteriordb`

This repository contains the Python package to efficiently work with the [posteriordb](https://github.com/stan-dev/posteriordb) repository. The R package includes convenience functions to access data, model code and information for individual posteriors, models, data and draws.

> [!IMPORTANT]
> This repository `posteriordb-python` only contains Python convience functions. The repository [`posteriordb`](https://github.com/stan-dev/posteriordb) contains the actual posteriors and associated models, data, and reference draws.

## Python versions

Currently only Python 3.6+ is supported. Python 3.5+ support can be added if needed. Support is not planned for Python 2.

## Installation

Installation from PyPI is recommended.

```bash
pip install posteriordb
```

Installing from the local clone.

```bash
git clone https://github.com/stan-dev/posteriordb-python
cd posteriordb-python
python setup.py bdist_wheel
pip install .
```

## Using the posterior database from Python

The included database contains convenience functions to access data, model code, and information for individual posteriors. This database can be created with local or online access to `posteriordb`.

For local access, ensure `posteriordb` is already dowloaded or clone it with:

```bash
git clone https://github.com/stan-dev/posteriordb.git
```

Then create the database by setting `pdb_path`:

```python
from posteriordb import PosteriorDatabase
pdb_path = path_to_your_posteriordb
my_pdb = PosteriorDatabase(pdb_path)
```
If you run this code in the same directory in which `posteriordb` was cloned, the path `pdb_path` will be `../posteriordb/posterior_database`.

For online access, use the `PosteriorDatabaseGithub` class with a [GitHub Personal Access Token (PAT)](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) to interact with `posteriordb` remotely. We recommend creating the `GITHUB_PAT` with read-only permissions and setting it as an environmental variable (therefore the `GITHUB_PAT` is not shown in the examples below).

If not explicitly defined, `PosteriorDatabase` and `PosteriorDatabaseGithub` will create a new (or use old database) located at `POSTERIOR_DB_PATH` if it's
defined. `PosteriorDatabaseGithub` will finally use `$HOME/.posteriordb/posterior_database` as a fallback location if no environmental variables have been set.
Each model and data is only downloaded and cached when needed.

```python
>>> from posteriordb import PosteriorDatabaseGithub
>>> import os
>>> # It is recommended that GITHUB_PAT is added to the user environmental variables
>>> # outside Python and not in a Python script as shown in this example code
>>> os.environ["GITHUB_PAT"] = "token-string-here"
>>> my_pdb = PosteriorDatabaseGithub()
```

To list the posteriors available, use `posterior_names`.

```python
>>> pos = my_pdb.posterior_names()
>>> pos[:5]

['roaches-roaches_negbin',
 'syn_gmK2D1n200-gmm_diagonal_nonordered',
 'radon_mn-radon_variable_intercept_centered',
 'syn_gmK3D2n300-gmm_nonordered',
 'radon-radon_hierarchical_intercept_centered']

```

In the same fashion, we can list data and models included in the database as

```python
>>> mn = my_pdb.model_names()
>>> mn[:5]

['gmm_diagonal_nonordered',
 'radon_pool',
 'radon_partial_pool_noncentered',
 'blr',
 'radon_hierarchical_intercept_noncentered']


>>> dn = my_pdb.dataset_names()
>>> dn[:5]

['radon_mn',
 'wells_centered',
 'radon',
 'wells_centered_educ4_interact',
 'wells_centered_educ4']


```

The posterior's name is made up of the data and model fitted
to the data. Together, these two uniquely define a posterior distribution.
To access a posterior object we can use the posterior name.

```python
>>> posterior = my_pdb.posterior("eight_schools-eight_schools_centered")
```

From the posterior we can access the dataset and the model

```python
>>> model = posterior.model
>>> data = posterior.data
```

We can also access the names of posteriors, models and datasets.

```python
>>> posterior.name
"eight_schools-eight_schools_centered"

>>> model.name
"eight_schools_centered"

>>> data.name
"eight_schools"

```

We can access the same model and dataset also directly from the posterior database
```python
>>> model = my_pdb.model("eight_schools_centered")
>>> data = my_pdb.data("eight_schools")
```

From the model we can access model code and information about the model

```python
>>> model.code("stan")
data {
  int <lower=0> J; // number of schools
  real y[J]; // estimated treatment
  real<lower=0> sigma[J]; // std of estimated effect
}
parameters {
  real theta[J]; // treatment effect in school j
  real mu; // hyper-parameter of mean
  real<lower=0> tau; // hyper-parameter of sdv
}
model {
  tau ~ cauchy(0, 5); // a non-informative prior
  theta ~ normal(mu, tau);
  y ~ normal(theta , sigma);
  mu ~ normal(0, 5);
}

>>> model.code_file_path("stan")
'/home/eero/posterior_database/content/models/stan/eight_schools_centered.stan'

>>> model.information
{'keywords': ['bda3_example', 'hiearchical'],
 'description': 'A centered hiearchical model for the 8 schools example of Rubin (1981)',
 'urls': ['http://www.stat.columbia.edu/~gelman/arm/examples/schools'],
 'title': 'A centered hiearchical model for 8 schools',
 'references': ['rubin1981estimation', 'gelman2013bayesian'],
 'added_by': 'Mans Magnusson',
 'added_date': '2019-08-12'}
```
Note that the references are referencing to BibTeX items that can be found in `content/references/references.bib`.

From the dataset we can access the data values and information about it

```python
>>> data.values()
{'J': 8,
 'y': [28, 8, -3, 7, -1, 1, 18, 12],
 'sigma': [15, 10, 16, 11, 9, 11, 10, 18]}

>>> data.file_path()
'/tmp/tmpx16edu0w'

>>> data.information
{'keywords': ['bda3_example'],
 'description': 'A study for the Educational Testing Service to analyze the effects of\nspecial coaching programs on test scores. See Gelman et. al. (2014), Section 5.5 for details.',
 'urls': ['http://www.stat.columbia.edu/~gelman/arm/examples/schools'],
 'title': 'The 8 schools dataset of Rubin (1981)',
 'references': ['rubin1981estimation', 'gelman2013bayesian'],
 'added_by': 'Mans Magnusson',
 'added_date': '2019-08-12'}
```

To access gold standard posterior draws we can use `reference_draws` as follows.

```python
>>> posterior.reference_draws_info()
{'name': 'eight_schools-eight_schools_noncentered',
 'inference': {'method': 'stan_sampling',
  'method_arguments': {'chains': 10,
   'iter': 20000,
   'warmup': 10000,
   'thin': 10,
   'seed': 4711,
   'control': {'adapt_delta': 0.95}}},
 'diagnostics': {'diagnostic_information': {'names': ['mu',
    'tau',
    'theta[1]',
    ...

>>> gs = posterior.reference_draws()
>>> import pandas as pd
>>> pd.DataFrame(gs)

	theta[1]	                                        theta[2]
0	[10.6802773011458, 6.45383910854259, -2.241629...	[9.71770681295263, 4.41030824418493, 0.7617047...
1	[5.70891361633589, 10.3012059848039, 4.2439533...	[-2.32310565394337, 14.8121789773659, 6.517256...
2	[7.23747096507585, -0.427831558524343, 9.14782...	[7.35425759420389, 8.69579738064637, 8.9058764...
3	[4.44915522912766, 2.34711393762556, 17.680378...	[2.4368039319606, 5.89809320808632, 8.63031558...
...
```
