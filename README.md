# Bayesian Active Learning (Baal) [![CircleCI](https://circleci.com/gh/ElementAI/baal.svg?style=svg&circle-token=aa12d3134798ff2bf8a49cebe3c855b96a776df1)](https://circleci.com/gh/ElementAI/baal) [![Coverage Status](https://coveralls.io/repos/github/ElementAI/baal/badge.svg?branch=develop&t=HLyM0v)](https://coveralls.io/github/ElementAI/baal?branch=develop)


<p align="left">
  <img height=15% width=25% src="https://github.com/ElementAI/baal/blob/master/docs/literature/images/repo_logo_25_no_corner.svg">
</p>

BaaL is an active learning library developed at
[ElementAI](https://www.elementai.com/). This repository contains techniques
and reusable components to make active learning accessible for all.

## Installation and requirements

BaaL requires Python>=3.6

To install baal using pip: `pip install baal`

To install baal from source: `pip install -e .`

For requirements please see: _[requirements.txt](requirements.txt)_.

## Our background and Motivation

Active learning is a special case of machine learning in which a learning
algorithm is able to interactively query the user (or some other information
source) to obtain the desired outputs at new data points
([Wiki](https://en.wikipedia.org/wiki/Active_learning)).

The goal of baal team is to make a production level ready framework for
active-learning. In this repo, you could find a set of heuristics that we
normally use in our active learning setup. You could also find the SoTA
experiments which we tested and if their performance is competent to production
pipeline needs, they are constantly added to our baal package.


## BaaL Framework

At the moment BaaL supports the following methods to perform active learning.

- Monte-Carlo Dropout (Gal et al. 2015)

**Please see our Roadmap [below](./README.md#roadmap-subject-to-change-depending-on-the-community).**

The **Monte-Carlo Dropout** method is a known approximation for bayesian neural
networks. In this method, the dropout layer is used both in training and test
time. Using of dropout layer and iterating through the model in test time, help
us to calculate the uncertainty of the model prediction using one of our
provided uncertainty measurements in _src/baal/active/heuristics.py_.

The framework consists of four main parts as it is demonstrated in the flowchart below:

- ActiveLearningDataset
- Heuristics
- ModelWrapper
- ActiveLearningLoop

<p align="center">
  <img src="./docs/literature/images/Baalscheme.svg">
</p>

The dataset should be wrapped in our _[**ActiveLearningDataset**](src/baal/active/dataset.py)_ class. This will assure that the dataset is split in
training and pool sets. The pool set represents the rest of training set which are yet
to be labelled.


We provide a lightweight object _[**ModelWrapper**](src/baal/modelwrapper.py)_ similar to `keras.Model` to make it easier to train and test the model. If your model is not ready for active learning, we provide Modules to prepare them. 

For example, the _[**MCDropoutModule**](src/baal/bayesian/dropout.py)_ wrapper changes the existing dropout layer
to be used in both training and inference time and the ModelWrapper makes
the training and prediction outputs compatible with respect to number of
iterations. The number of iterations are defined as a hyper parameter in the
active learning set up which indicates the number of times the model will be
looped over with the test set to create a set of predictions for each of the
test set items.

In conclusion, your script would be similar to this:
```python
dataset = ActiveLearningDataset(your_dataset)
dataset.label_randomly(INITIAL_POOL)  # label some data
model = MCDropoutModule(your_model)
model = ModelWrapper(model, your_criterion)
active_loop = ActiveLearningLoop(dataset,
                                 get_probabilities=model.predict_on_dataset,
                                 heuristic=heuristics.BALD(shuffle_prop=0.1),
                                 ndata_to_label=NDATA_TO_LABEL)
for al_step in range(N_ALSTEP):
    model.train_on_dataset(dataset, optimizer, BATCH_SIZE, use_cuda=use_cuda)
    if not active_loop.step():
        # We're done!
        break
```


For a complete experiment, we provide _[experiments/](experiments/)_ to understand how to
write an active training process. Generally, we use the **ActiveLearningLoop**
provided at _[src/baal/active/active_loop.py](src/baal/active/active_loop.py)_.
This class provides functionality to get the predictions on the unlabeled pool
after each (few) epoch(s) and sort the next set of data items to be labeled
based on the calculated uncertainty of the pool.


### Roadmap (Subject to change depending on the community.)

* [x] Initial FOSS release with MCDropout (Gal et al. 2015)
* [ ] MCDropConnect (Mobiny et al. 2019)
* [ ] Bayesian layers (Shridhar et al. 2019)
* [ ] Unsupervised methods
* [ ] NNGP (Panov et al. 2019)
* [ ] SWAG (Zellers et al. 2018)


### Re-run our Experiments

```bash
docker build [--target prod_baal] -t baal .
docker run --rm baal python3 experiments/vgg_mcdropout_cifar10.py 
```

### Use BaaL for YOUR Experiments

Simply clone the repo, and create your own experiment script similar to the
example at _experiments/vgg_experiment.py_. Make sure to use the four main parts
of Baal framework. _Happy running experiments_

### Dev install

Simply build the dockerfile as below:

```bash
git clone git@github.com:ElementAI/baal.git
docker build [--target base_baal] -t baal-dev .
```

Now you have all the requirements to start contributing to BaaL. _**YEAH!**_

### Contributing!

To contribute, see [CONTRIBUTING.md](./CONTRIBUTING.md).


### Who We Are!

"There is passion, yet peace; serenity, yet emotion; chaos, yet order."

At ElementAI, BaaL team tests and implements the most recent papers on uncertainty estimation and active learning.
 BaaL team is here to serve you best:

- [Parmida Atighehchian](mailto:parmida@elementai.com)
- [Frédéric Branchaud-Charron](mailto:frederic.branchaud-charron@elementai.com)
- [Jan Freyberg](mailto:jan.freyberg@elementai.com)
- [Lorne Schell](mailto:lorne.schell@elementai.com)

### Licence
To get information on licence of this API please read [LICENCE](./LICENSE)