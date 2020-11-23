# Water Demand Forecasting
![alt text](img/readme/london_logo.png "The City of London, Canada")

The purpose of this project is to deliver a machine learning solution to
forecasting aggregate water demand. This work was led by the Artificial
Intelligence Research and Development Lab out of the Information
Technology Services division. This repository contains the code used to
fit multiple types of machine learning models to predict future daily
water consumption. This repository is intended to serve as an example
for other municipalities who wish to explore water demand forecasting in
their own locales.

## Table of Contents
1. [**_Getting Started_**](#getting-started)
2. [**_Use Cases_**](#use-cases)  
   i)
   [_Train a model and visualize forecasts_](#train-a-model-and-visualize-a-forecast)  
   ii) [_Train  all  models_](#train-all-models)  
   iii)
   [_Bayesian hyperparameter optimization_](#bayesian-hyperparameter-optimization)  
   iv)
   [_Forecasting  with  a  trained  model_](#forecasting-with-a-trained-model)  
   v) [_Cross validation_](#cross-validation)  
   vi)
   [_Client clustering experiment (using K-Prototypes)_](#client-clustering-experiment-using-k-prototypes)  
   vii) [_Model interpretability_](#model-interpretability)
4. [**_Data Preprocessing_**](#data-preprocessing)
5. [**_Project Structure_**](#project-structure)
6. [**_Project Config_**](#project-config)
7. [**_Contact_**](#contact)

## Getting Started
1. Clone this repository (for help see this
   [tutorial](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository)).
2. Install the necessary dependencies (listed in
   [requirements.txt](requirements.txt)). To do this, open a terminal in
   the root directory of the project and run the following:
   ```
   $ pip install -r requirements.txt
   ```
3. Obtain raw water consumption data and preprocess it accordingly. See
   [Data preprocessing](#data-preprocessing) for more details.
4. Update the _TRAIN >> MODEL_ field of [_config.yml_](config.yml) with
   the appropriate string representing the model type you wish to
   train. To train a model, ensure the _TRAIN >>
   EXPERIMENT_ field is set to _'train_single'_.
5. Execute [_train.py_](src/train.py) to train your chosen model on your
   preprocessed data. The trained model will be serialized within
   _results/models/_, and its filename will resemble the following
   structure: _{modeltype}{yyyymmdd-hhmmss}.{ext}_, where _{modeltype}_
   is the type of model trained, _{yyyymmdd-hhmmss}_ is the current
   time, and _{ext}_ is the appropriate file extension.
6. Navigate to _results/experiments/_ to see the performance metrics
   achieved by the model's forecast on the test set. The file name will
   be _{modeltype}_eval_{yyyymmdd-hhmmss}.csv_. You can find a
   visualization of the test set forecast in
   _img/forecast_visualizations/_. Its filename will be
   _{modeltype}_forecast_{yyyymmdd-hhmmss}.png_.


## Use Cases

### Train a model and visualize a forecast
1. Once you have obtained a preprocessed dataset (see step 3 of
   [_Getting Started_](#getting-started), ensure that the preprocessed
   dataset is located in _data/preprocessed/_ with filename
   _"preprocessed_data.csv"_.
2. In [config.yml](config.yml), set _EXPERIMENT >> TRAIN_ to
   _'single_train'_. Set _TRAIN >> MODEL_ to the appropriate string
   representing the model type you wish to train.
3. Execute [_train.py_](src/train.py) to train your chosen model on your
   preprocessed data. The trained model will be serialized within
   _results/models/_, and its filename will resemble the following
   structure: _{modeltype}{yyyymmdd-hhmmss}.{ext}_, where _{modeltype}_
   is the type of model trained, _{yyyymmdd-hhmmss}_ is the current
   time, and _{ext}_ is the appropriate file extension.
4. Navigate to _results/experiments/_ to see the performance metrics
   achieved by the model's forecast on the test set. The file name will
   be _{modeltype}_eval_{yyyymmdd-hhmmss}.csv_. You can find a
   visualization of the test set forecast in
   _img/forecast_visualizations/_. Its filename will be
   _{modeltype}_forecast_{yyyymmdd-hhmmss}.png_. This visualization
   displays training set predictions, test set forecasts, test set
   predictions (depending on the model type), training residuals, and
   test error. The image below is an example of one of a test set
   forecast visualization.

![alt text](img/readme/test_set_forecast_visualization.png "Test set
forecast visualization")

### Train all models
We investigated several different model types for the task of water
demand forecasting. As such, we followed the
[Strategy Design Pattern](https://www.tutorialspoint.com/design_pattern/strategy_pattern.htm)
to encourage rapid prototyping of our models. This pattern allows the
user to interact with models using a common interface to perform core
operations in the machine learning workflow (e.g. train, forecast,
serialize the model). We investigated the following models:
- [Prophet](https://facebook.github.io/prophet/)
- Recurrent neural network with [LSTM layer](https://www.tensorflow.org/api_docs/python/tf/keras/layers/LSTM)
- Recurrent neural network with [GRU layer](https://www.tensorflow.org/api_docs/python/tf/keras/layers/GRU)
- Convolutional neural network with [1D convolutional layers](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Conv1D)
- [ARIMA](https://www.statsmodels.org/stable/generated/statsmodels.tsa.arima.model.ARIMA.html)
- [SARIMAX](https://www.statsmodels.org/dev/generated/statsmodels.tsa.statespace.sarimax.SARIMAX.html)
- [Ordinary least-squares linear regression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html)
- [Random forest regression](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html)

The following steps describe how to train all models.
1. Once you have obtained a preprocessed dataset (see step 3 of
   [_Getting Started_](#getting-started), ensure that the preprocessed
   dataset is located in _data/preprocessed/_ with filename
   _"preprocessed_data.csv"_.
2. In [config.yml](config.yml), set _EXPERIMENT >> TRAIN_ to
   _'all_train'_.
3. Execute [_train.py_](src/train.py) to train each model on your
   preprocessed data. Performance metrics and test set forecast
   visualizations will be saved for each model, as described in Step 4
   of
   [Train a model and visualize a forecast](#train-a-model-and-visualize-a-forecast).

### Bayesian hyperparameter optimization
Hyperparameter optimization is an important part of the standard machine
learning workflow. We chose to conduct Bayesian hyperparameter
optimization. The results of the optimization informed the final
hyperparameters currently set in the _HPARAMS_ sections of
[config.yml](config.yml). The objective of the Bayesian optimization was
the minimization of mean absolute error in the average test set forecast
resulting from a time series cross validation. With the help of the
[scikit-optimize](https://scikit-optimize.github.io/stable/index.html)
package, we were able to visualize the effects of single hyperparameters
and pairs of hyperparameters on the objective.

To conduct your own Bayesian hyperparameter optimization, you may follow
the steps below. Note that if you are not planning on changing the
default hyperparameter ranges set in the _HPARAM_SEARCH_ section of
[config.yml](config.yml), you may skip step 2.
1. In [config.yml](config.yml), set _EXPERIMENT >> TRAIN_ to
   _'hparam_search'_. Set _TRAIN >> MODEL_ to the appropriate string
   representing the model type you wish to train.
2. In the _TRAIN >> HPARAM_SEARCH >> MAX_EVALS_ field of
   [config.yml](config.yml), set the maximum number of combinations of
   hyperparameters to test. In the _TRAIN >> HPARAM_SEARCH >>
   LAST_FOLDS_ field, set the number of folds from cross validation to
   average the test set forecast objective over. If you wish to change
   the objective metric, update _TRAIN >> HPARAM_SEARCH >>
   HPARAM_OBJECTIVE_ accordingly.
3. Set the ranges of hyperparameters you wish to study in the
   _HPARAM_SEARCH >> {MODEL}_ subsection of [config.yml](config.yml),
   where _{{MODEL}}_ is the model name you set in Step 1. The config
   file already has a comprehensive set of hyperparameter ranges defined
   for each model type, so you may not need to change anything in this
   step. Consider whether your hyperparameter ranges are sets, integer
   ranges, or float ranges, and whether they need to be investigated on
   a uniform or logarithmic range. See below for an example of how to
   correctly set ranges for different types of hyperparameters.
   ```
    FC0_UNITS:
      TYPE: 'set'
      RANGE: [32, 64, 128]
    DROPOUT:
      TYPE: 'float_uniform'
      RANGE: [0.0, 0.5]
    LR:
      TYPE: 'float_log'
      RANGE: [0.00001, 0.001]   
    PATIENCE:
      TYPE: 'int_uniform'
      RANGE: [5, 15]      
   ```
4. Execute [train.py](src/train.py). A CSV log detailing the trials in
   the Bayesian optimization will be located in
   _results/experiments/hparam_search\_{modelname}\_{yyyymmdd-hhmmss}.csv_,
   where _{modeltype}_ is the type of model investigated, and
   _{yyyymmdd-hhmmss}_ is the current time. Each row in the CSV will
   detail the hyperparameters used for a particular trial, along with
   the value of the objective from the cross validation run with those
   hyperparameters. The final row details the best trial, which contains
   the optimal hyperparameter values. The visualization of the search
   can be found at
   _img/experiment_visualizations/Bayesian_opt\_{modeltype}\_{yyyymmdd-hhmmss}.png_.
   See below for an example of a Bayesian hyperparameter optimization
   visualization for an LSTM-based model. For details on how to
   interpret this visualization, see
   [this entry](https://scikit-optimize.github.io/stable/modules/generated/skopt.plots.plot_objective.html)
   in the scikit-optimize docs.

![alt text](img/readme/hparam_visualization.png "A sample Bayesian
hyperparameter optimization visualization")

### Forecasting with a trained model
Once a model has been fitted, the user may wish to obtain water
consumption forecasts for future dates. Note that forecasts differ from
predictions in that each prediction may be used in part to make a
prediction for the next day. To obtain predictions for any number of
days following the final date in the training set, follow the steps
below.
1. Ensure that you have already trained a model (see
   [Train a model and visualize a forecast](#train-a-model-and-visualize-a-forecast)).
2. In [config.yml](config.yml), set _FORECAST >> MODEL_ to the type of
   model you trained. Set _FORECAST >> MODEL_PATH_ to the path of the
   serialization of the model. Set _FORECAST >> DAYS_ to the number of
   future days you wish to make forecasts for.
3. Execute _src/predict.py_. The model will be loaded from disk and will
   generate a forecast for the desired number of days.
4. A CSV containing the forecast will be saved to
   _'results/predictions/forecast\_{modelname}\_{days}d\_{yyyymmdd-hhmmss}.csv_,
   where _{modeltype}_ is the type of model investigated, _{days}_ is
   the length of the forecast in days, and _{yyyymmdd-hhmmss}_ is the
   current time.


### Cross validation
Cross validation helps us select a model that is as unbiased as possible
towards any particular dataset. By using cross validation, we can be
increasingly confident in the generalizability of our model in the
future. We employ a
[time series variant](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html)
of k-fold cross validation to validate our model. For the
_k<sup>th</sup>_ split, the training set comprises the first _k_ folds
of time series data and the test set is the _(k+1)<sup>th</sup>_ fold.
Remaining data is disregarded for the time being. To run a cross
validation experiment, follow the steps below.

1. Once you have obtained a preprocessed dataset (see step 3 of
   [_Getting Started_](#getting-started), ensure that the preprocessed
   dataset is located in _data/preprocessed/_ with filename
   _"preprocessed_data.csv"_.
2. In [config.yml](config.yml), set _EXPERIMENT >> TRAIN_ to
   _'cross_validation'_. Set _TRAIN >> MODEL_ to the appropriate string
   representing the model type you wish to train. Set _TRAIN >> N_FOLDS_
   to your chosen value for _k_.
3. Execute [_train.py_](src/train.py) to conduct cross validation with
   your chosen model. _k_ models will be trained, each on successively
   larger datasets. A spreadsheet detailing test set forecast
   performance metrics for each of the _k_ training experiments will be
   generated. The mean and standard deviation of each metric across the
   training experiments appear in the bottom 2 rows. Upon completion, it
   will be saved to
   _'results/predictions/cross_val\_{modelname}\_{yyyymmdd-hhmmss}.csv_,
   where _{modeltype}_ is the type of model trained and
   _{yyyymmdd-hhmmss}_ is the current time.


### Client clustering experiment (using K-Prototypes)
We were interested in investigating whether clients could be effectively
clustered. We developed a separate preprocessing method that produced a
feature vector for each client, including estimates of their monthly
consumption over the past year, and other features pertaining to their
water comsumption (e.g. meter type, land parcel area). Since our client
feature vectors consisted of numerical and categorical data, and in the
spirit of minimizing time complexity,
[k-prototypes](https://www.semanticscholar.org/paper/CLUSTERING-LARGE-DATA-SETS-WITH-MIXED-NUMERIC-AND-Huang/d42bb5ad2d03be6d8fefa63d25d02c0711d19728)
was selected as the clustering algorithm. We wish to acknowledge Nico de
Vos, as we made use of his
[implementation](https://github.com/nicodv/kmodes) of k-prototypes. This
experiment runs k-prototypes a series of times and selects the best
clustering (i.e. least average dissimilarity between clients and their
assigned clusters' centroids). Our intention was to provide city water
management with an alternative means by which to categorize clients. The
steps below detail how to cluster such a dataset.

1. Ensure that you have a CSV of client data saved to
   _'data/preprocessed/client_data.csv'_. The primary key should be
   named _CONTRACT_ACCOUNT_. Ensure that the lists located in the _DATA
   \>> CATEGORICAL_FEATS_ and _DATA >> BOOLEAN_FEATS_ fields of
   [config.yml](/config.yml) match the names of your categorical and
   boolean features respectively.
2. Set the _EXPERIMENT_ field of the _K-PROTOTYPES_ section of
   [config.yml](/config.yml) to _'cluster_clients'_. Consult
   [Project Config](#k-prototypes) before changing the default values of
   oter parameters in [config.yml](/config.yml).
3. Execute _[kprototypes.py](src/data/kprototypes.py)_.
4. 2 separate files will be saved once clustering is complete:
   1. A spreadsheet depicting cluster assignments by Contract Account
      number will be saved to
      _results/experiments/client_clusters\_{yyyymmdd-hhmmss}.csv_,
      where _{yyyymmdd-hhmmss}_ is the current time).
   2. Cluster centroids will be saved to a spreadsheet. Centroids have
      the same features as clients and their LIME explanations will be
      appended to the end by default. The spreadsheet will be saved to
      _results/experiments/cluster_centroids\_{yyyymmdd-hhmmss}.csv_.

A tradeoff for the efficiency of k-prototypes is the fact that the
number of clusters must be specified a priori. In an attempt to
determine the optimal number of clusters, the
[Average Silhouette Method](https://uc-r.github.io/kmeans_clustering#silo)
was implemented. During this procedure, different clusterings are
computed for a range of values of _k_. A graph is produced that plots
average Silhouette Score versus _k_. The higher average Silhouette Score
is, the more optimal _k_ is. To run this experiment, see the below
steps.
1. Follow steps 1 and 2 as outlined above in the clustering
   instructions.
2. Set the _K-PROTOTYPES >> EXPERIMENT_ field of
   [config.yml](/config.yml) to _'silhouette_analysis'_. At this time,
   you may wish to change the _K_MIN_ and _K_MAX_ field of the
   _K-PROTOTYPES_ section from their defaults. Values of _k_ in the
   integer range of _[K_MIN, K_MAX]_ will be investigated.
3. Execute _[kprototypes.py](src/data/kprototypes.py)_. An image
   depicting a graph of average Silhouette Score versus _k_ will be
   saved to
   _img/data_visualizations/_silhouette_plot\_{yyyymmdd-hhmmss}.png_.
   Upon visualizing this graph, note that a larger average Silhouette
   Score implies a better clustering. For example, in the below images,
   _k_ = 3 is the best number of clusters.
   ![alt text](img/readme/silhouette_plot_example.png "A sample
   Silhouette plot")

### Model interpretability
To gain insight into model functionality, we delved into the
interpretability of our selected model. Currently, only the Prophet
model is supported. The Prophet model is inherently interpretable, since
it is a composition of interpretable functional components for trend,
weekly periodicity, yearly periodicity, and holidays. The four
components can be saved and plotted by following the below steps:
1. In [config.yml](config.yml), set _TRAIN >> INTERPRETABILITY_ to
   _true_. Set _TRAIN >> MODEL_ to _'prophet'_.
2. Follow the steps in this document to run any training experiment
   (e.g. [single train](#train-a-model-and-visualize-a-forecast), [train
   all](#train-all-models), [cross validation](#cross-validation)).
3. Upon completion of training, the values comprising the 4 components
   are saved to separate files in a new folder named
   _results/interpretability/Prophet\_{yyyymmdd-hhmmss}/_, where
   _{yyyymmdd-hhmmss}_ is the current datetime. The trend and holiday
   components are saved to CSV files detailing weights for each date.
   The weekly and yearly seasonality components are saved to JSON files
   containing their respective preiodic function parameters. A
   visualization of the components of the Prophet model can be found in
   _img/interpretability\_visualizations/Prophet\_components\_{yyyymmdd-hhmmss}.png_.
   See below for an example of a visualization of the components of a
   Prophet model. ![alt text](img/readme/Prophet_components.png "An
   example of Prophet components")

## Data preprocessing
Our data preprocessing method is implemented in
[preprocess.py](src/data/preprocess.py). Since water consumption
database schemas vary considerably across jurisdictions, you likely will
have to implement your own preprocessing script. The data that was used
for our experiments was organized in a schema specific to London,
Canada. Your preprocessed dataset should be a CSV file with at least the
following columns:
   - _Date_: A timestamp representing a date
   - _Consumption_: A float representing an estimate of total water
     consumption on the date

Any other features you wish to use must have their own column. The model
assumes all datatypes are numerical. As a result, categorical and
Boolean features must be converted to a numeric representation. Below is
a high-level overview of our preprocessing strategy.

Raw consumption data was provided to us as a set of CSV files containing
consumption entries for each client at different billing periods. A
single row consisted of a client's identifying information (e.g.
contract account number), the start and end dates of their current
billing period, how much water was consumed over the current billing
period, and several other features pertaining to their water access
(e.g. meter type, land parcel area, flags for certain billing
considerations). These other features were numerical, categorical, or
Boolean. Our goal was to transform all the raw data into a dataset that
contained snapshots of the entire city's water consumption for each day.

Aggregate system daily water consumption was estimated as follows. An
estimate for each client's water consumption on a particular day was
arrived at by dividing their water consumption over the billing period
that the day fell under by the length of the billing period. We assumed
uniform daily usage over the billing period because we did not have
access to more granular data. The citywide estimate for water
consumption on that day is then the sum of the daily estimates for each
client.

Other features were aggregated to represent their state in the city as a
whole for a particular day. Their value for each client was taken to be
the value during the billing period including that day for that client.
For each day, the average and standard deviation of numerical client
features were taken to be daily estimates for that feature citywide. A
similar approach was taken for Boolean features, whose values were 0
(false) or 1 (true) in raw data. The daily citywide estimate for that
feature was the proportion of clients who had a value of "true" for that
feature on that day. Citywide daily estimates of categorical client
features were represented by a set of features for each value of the
categorical feature, whereby each feature in the set represented the
fraction of clients who had that value for that categorical feature. For
example, each client had a feature categorizing the client as living in
low, medium or high density residential regions. The preprocessed
dataset had a feature for each of these values, indicating the fraction
of clients living in low, medium or high density residential regions
respectively on a given day.

We later discovered that the features other than consumption did not
necessarily improve our forecasts. Thus, we abandoned the other features
in our experiments, reducing our preprocessed dataset to a univariate
dataset, mapping Date to Consumption. Note that some model architectures
available in this repository require univariate data; when training
these models, other features would be disregarded.

## Project Structure
The project looks similar to the directory structure below. Disregard
any _.gitkeep_ files, as their only purpose is to force Git to track
empty directories. Disregard any _.\__init\__.py_ files, as they are
empty files that enable Python to recognize certain directories as
packages.

```
├── data
│   ├── processed                 <- Products of preprocessing
│   ├── raw                       <- Raw data
│   └── serializations            <- Serialized sklearn transformers
|
├── img
|   ├── data_visualizations       <- Visualizations of preprocessed data, clusterings
|   ├── experiment_visualizations <- Visualizations for experiments
|   ├── forecast_visualizations   <- Visualizations of model forecasts
|   └── readme                    <- Image assets for README.md
├── results
│   ├── experiments               <- Experiment results
│   ├── logs                      <- TensorBoard logs
│   ├── models                    <- Trained model serializations
│   └── predictions               <- Model predictions
|
├── src
│   ├── data
|   |   ├── data_merge.py         <- Script for merging new raw data with existing raw data
|   |   ├── kprototypes.py        <- Script for learning client clusters
|   |   └── preprocess.py         <- Data preprocessing script
│   ├── models                    <- TensorFlow model definitions
|   |   ├── arima.py              <- Script containing ARIMA model class definition
|   |   ├── model.py              <- Script containing abstract model class definition
|   |   ├── nn.py                 <- Script containing neural network model class definitions
|   |   ├── prophet.py            <- Script containing Prophet model class definition
|   |   ├── sarimax.py             <- Script containing SARIMAX model class definition
|   |   └── skmodels.py           <- Script containing scikit-learn model class definitions
|   ├── visualization             <- Visualization scripts
|   |   └── visualize.py          <- Script for visualization production
|   ├── predict.py                <- Script for prediction on raw data using trained models
|   └── train.py                  <- Script for training experiments
|
├── .gitignore                    <- Files to be be ignored by git.
├── config.yml                    <- Values of several constants used throughout project
├── config_private.yml            <- Private information, e.g. database keys (not included in repo)
├── LICENSE                       <- Project license
├── README.md                     <- Project description
└── requirements.txt              <- Lists all dependencies and their respective versions
```

## Project Config
This project contains several configurable variables that are defined in
the project config file: [config.yml](config.yml). When loaded into
Python scripts, the contents of this file become a dictionary through
which the developer can easily access its members.

For user convenience, the config file is organized into major components
of the model development pipeline. Many fields need not be modified by
the typical user, but others may be modified to suit the user's specific
goals. A summary of the major configurable elements in this file is
below.

#### PATHS
- **RAW_DATA_DIR**: Directory containing raw data
- **RAW_DATASET**: Path to file containing aggregated raw data CSV
- **PREPROCESSED_DATA**: Path to CSV file containing preprocessed data
- **CLIENT_DATA**: Path to CSV file containing client data organized for
  K-Prototypes clustering
#### DATA
- **NUMERICAL_FEATS**: A list of names of numerical features in the raw
  data
- **CATEGORICAL_FEATS**: A list of names of categorical features in the
  raw data
- **BOOLEAN_FEATS**: A list of names of Boolean features in the raw data
- **TEST_FRAC**: Fraction of dates comprising the test set
#### TRAIN
- **MODEL**: The type of model to train. Can be set to one of One of
  _'prophet', 'lstm', 'gru', '1dcnn', 'arima', 'sarimax',
  'random_forest',_ or _'linear_regression'_.
- **EXPERIMENT**: The type of training experiment you would like to
  perform if executing [_train.py_](src/train.py). Choices are
  _'train_single', 'train_all', 'hparam_search', 'hparam_search',_ or
  _'cross_validation'_.
- **N_FOLDS**: Number of folds for time series k-fold cross validation
- **HPARAM_SEARCH**: Parameters associated with Bayesian hyperparameter
  search
  - **N_EVALS**: Number of combinations of hyperparameters to test
  - **HPARAM_OBJECTIVE**: Metric to optimize for. Set to one of _'MAPE',
    'MAE', 'MSE',_ or _'RMSE'_.
  - **LAST_FOLDS**: If you wish to speed up the k-fold cross validation
    for each step of the Bayesian hyperparameter optimization, only
    train models for values of _k_ in _[LAST_FOLDS, N_FOLDS]_.
#### FORECAST
- **MODEL**: The type of model for forecasting. Can be set to one of One
  of _'prophet', 'lstm', 'gru', '1dcnn', 'arima', 'sarimax',
  'random_forest',_ or _'linear_regression'_.
- **MODEL_PATH**: The path to a serialization of the chosen model type
- **DAYS**: The number of days to forecast
#### HPARAMS
Hyperparameter values for each model. These will be loaded for training
experiments. Hyperparameter field descriptions for the LSTM-based model
are included below, as an example.
- **LSTM**:
  - **UNIVARIATE**: Whether the model is to be trained on consumption
    data alone or inclusive of other features in the preprocessed
    dataset. Set to _true_ or _false_.
  - **T_X**: Length of input sequence length to LSTM layer. Also
    described as the number of past time steps to include with each
    example.
  - **BATCH_SIZE**: Mini-batch size during training
  - **EPOCHS**: Number of epochs to train the model for
  - **PATIENCE**: Number of epochs to continue training after
    non-decreasing loss on validation set before training is halted
    (i.e. early stopping)
  - **VAL_FRAC**: Fraction of training data to segment off for creation
    of validation set
  - **LR**: Learning rate
  - **LOSS**: Loss function. Can be set to _'mae', 'mse', 'huber_loss'_
    and more.
  - **UNITS**: Number of units in LSTM layer
  - **DROPOUT**: Dropout rate
  - **FC0_UNITS**: Number of nodes in first fully connected layer
  - **FC1_UNITS**: Number of nodes in second fully connected layer
#### HPARAM_SEARCH
Hyperparameter ranges to test over during Bayesian hyperparameter
optimization. There is a subsection for each model. Each hyperparameter
range has the following configurable fields:
- **{Hyperparameter name}**:
  - **TYPE**: The type of hyperparameter range. Can be set to either
    _'set', 'int_uniform', 'float_uniform'_ or _'float_log'_.
  - **RANGE**: If _TYPE_ is _'set_', specify a list of possible values
    (e.g. _[16, 32, 64, 128]_). Otherwise, specify a 2-element list
    whose elements are the lower and upper endpoints for the range (e.g.
    _[0.00001, 0.01]_).
#### K-PROTOTYPES
- **K**: Desired number of client clusters when running K-Prototypes
- **N_RUNS**: Number of attempts at running K-Prototypes. The best
  clustering is saved.
- **N_JOBS**: Number of parallel compute jobs to create for
  k-prototypes. This is useful when _N_RUNS_ is high and you want to
  decrease the total runtime of clustering. Before increasing this
  number, check how many CPUs are available to you.
- **K_MIN**: Minimum value of _k_ to investigate in the Silhouette
  Analysis experiment
- **K_MAX**: Maximum value of _k_ to investigate in the Silhouette
  Analysis experiment
- **FEATS_TO_EXCLUDE**: A list of names of features you wish to exclude
  from the clustering
- **EVAL_DATE**: Date at which to evaluate clients' most recent monthly
  consumption
- **EXPERIMENT**: The type of k-prototypes clustering experiment you
  would like to perform if executing
  [_kprototypes.py_](src/data/kprototypes.py). Choices are
  _'cluster_clients'_ or _'silhouette_analysis'_.

## Contact

**Matt Ross**  
Manager, Artificial Intelligence  
Information Technology Services, City Manager’s Office  
City of London  
Suite 300 - 201 Queens Ave, London, ON. N6A 1J1  
maross@london.ca

**Blake VanBerlo**  
Data Scientist  
City of London Research and Development Lab  
C: vanberloblake@gmail.com