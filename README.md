# Experimental Reproducibility in Data Science with Sacred

Demonstrate how to use Sacred to track machine learning experiments on popular kaggle titanic competition

## Getting Started

- Kaggle Titanic Competition
- Sacred

### Prerequisites

- python3 - https://www.python.org/downloads/
- mongo - https://docs.mongodb.com/manual/installation/

### Setup

1. Clone repo: `git clone https://github.com/gilt/odsc-2018`
2. Setup Virtual Environment: `python3 -m venv /PATH/TO/odsc-2018/sacred-demo`
3. Install python packages: `pip install -r requirements.txt`

## Run Experiment

### Running an experiment with all defaults
`python experiments/model_accuracy.py` - 
Notice that different runs yield different results since we have not  
controlled the random seed.

But if we fix the seed by running:
`python experiments/model_accuracy.py with seed=0`
we should end up with the same results on every run.

### Running a variant

#### Variants in experiment
To run a different variant of our experiment:
```python experiments/model_accuracy2.py print_config with variant_rand_params```

Similarly, we have a config option called `save_submission` which is `False`
by default. We can turn it on from the CLI, which causes a submission file
to be generated and tracked as an artifact.
```python experiments/model_accuracy2.py with seed=0 save_submission=True```

#### Variants in ingredients
We also defined a `variant_simple` in our
[preprocessing ingredient](ingredients/preproc.py). To run this variant:
```python experiments/model_accuracy2.py with preprocess.variant_simple seed=0```

We can even use `print_config` to show a dry run of config and what's changed
from the default
```python experiments/model_accuracy2.py print_config with seed=0 dataset.variant_split save_submission=True```

#### Vary a bunch of stuff
```python experiments/model_accuracy2.py with variant_rand_params save_submission=True dataset.variant_presplit```


### Running with a mongo observer
0. Launch local mongo instance: `mongod`
1. Run Experiment (result will be stored in `sacred` database in mongo): 
`python experiments/model_accuracy.py -m sacred`

## See Results

### Mongo
To look at all our runs on mongo:
```
mongo
use sacred
db.runs.find().pretty()
```

### Sacredboard
Start local Sacredboard server and connect to local MongoDB instance listening on 27017, database name `sacred`: `sacredboard -m sacred`

## Model Blending Workflow

Run the following to simulate various experiments with random parameter search:
```bash
python experiments/model_accuracy2.py -m sacredblender \
    with variant_rand_params dataset.variant_presplit save_submission=True
```
Note: we switched to a new database `sacredblender` in case there's any
garbage in the `sacred` database. This is required because we've hardcoded
the lookup to `sacredblender` database

Now run an experiment that blends the top 3 runs based on holdout performance:
```bash
python experiments/model_accuracy2.py -m sacred \
    with dataset.blend preprocess.variant_all save_submission=True
```
Note: we switched back to `sacred` database here. We also went with the
default parameters for the meta blender model

You can always drop a database by doing the following:
```bash
mongo
use sacredblender
db.dropDatabase()
```

## Hyperopt Hyper Parameter Optimization

We show a way to use [hyperopt](https://github.com/hyperopt/hyperopt), a python library for hyper parameter 
optimization, with sacred to run many experiments. 

### Define Search Space

Search configs are defined in [hyperopt_hpo_configs](hpo/hyperopt_hpo_configs.py) as dictionaries. There is one config
for each of the experiments defined in the experiments folder. If a new config or experiment is added, they should be 
included and appropriately mapped in [gather_experiments_and_configs](hpo/hyperopt_hpo.py). 

### Run 

hyperopt parameter search run command: 

`python hpo/hyperopt_hpo.py NUM_RUNS MONGO_HOST:MONGO_PORT MONGO_DB EXPERIMENT_FILE_NAME`

`NUM_RUNS` - Number of hyperopt trials 
`MONGO_HOST:MONGO_PORT` - Should be `127.0.0.1:27017` if running locally
`MONGO_DB` - Should be the `sacred` or `sacredblender` collection
`EXPERIMENT_FILE_NAME` - Key must exist in [exp_and_configs](hpo/hyperopt_hpo.py) dict

`python hpo/hyperopt_hpo.py 5 127.0.0.1:27017 sacredblender model_accuracy`

Note that we are writing to `sacredblender` here. So, the blended model could be created after running HPO and writing
out the results to `sacredblender`.

# References
- [sacred](https://github.com/IDSIA/sacred) [(pub)](http://ml.informatik.uni-freiburg.de/papers/17-SciPy-Sacred.pdf)
- [sacredboard](https://github.com/chovanecm/sacredboard)
- [Kaggle Titanic](https://www.kaggle.com/c/titanic)
- [hyperopt](https://github.com/hyperopt/hyperopt)
- [sacred-hyperopt](https://github.com/Lab41/pythia/blob/master/experiments/hyperopt_experiments.py) [(blogpost)](https://gab41.lab41.org/effectively-running-thousands-of-experiments-hyperopt-with-sacred-dfa53b50f1ec)