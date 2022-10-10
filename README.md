# Usage:

Get the code:
```
git clone https://github.com/rareylab/NetworkBalanceScaling.git
```



Run NBS to predict properties based on a training- and a query-dataset:
```
cd NetworkBalanceScaling
./workflow_predict.sh
```

Find NBS predictions in bace_NBS_predicted/bace_split.sdf:
```
...
>  <pic50_predicted>  (1)
7.34712390561689
...
```

Run NBS to optimize predictions generated by an exernal preditive model:
```
cd NetworkBalanceScaling
./workflow_optimize_predictions.sh
```

Find NBS-optimized predictions in bace_NBS_optimized_prediction/bace_result.sdf:
```
...
>  <pic50>  (1)
7.3010302

>  <pic50_predicted>  (1)
7.15287888112478

>  <deviation>  (1)
0.01553303034815148

>  <pic50_predicted_optimized>  (1)
7.168411911472931
...
```

### bace_NBS_optimized_prediction/evaluates.log
Since the bace dataset contains measured values for the test-set, we can calculate how good the optimization performed: 
```
StdDevs:
original all values:              1.3555922472779236
original only in net values:      1.368353774909065
original only NOT in net values:  1.2911598752840818
optimized all values:             1.1494894733393708
optimized only in net values:     1.1902379396695375

                              Root Mean Square Deviation    R²        Num_of_Samples
Predictions(all):             0.768106                      0.676815  152
Predictions(only in net):     0.635844                      0.781936  102
Predictions(only NOT in net): 0.984272                      0.407014  50

Optimized (all):              0.762088                      0.681860  152
Optimized (only):             0.624960                      0.789338  102

Scores (un_opt):              0.984272                      0.407014  50

Unoptimizable molecules (IDs) (50 mols):
BACE_1376
BACE_1371
...
```

**all**: scores for all test-compounds

**only in net**: only scores for compounds, that have a MMP connection to other compounds

**only NOT in net**: only compounds, that are not connected to other compounds and therefore, could not be optimized

**Unoptimizable molecules**: since some molecules have no MMP connection to other compounds, they couldn't be optimized, you find the list of IDs here


### Runtimes
workflow_predict.sh: ~1min30sec

workflow_optimize_predictions.sh: ~2min 


## Workflows

Please execute the following workflows depending on your preferences:

### workflow\_predict.sh

* installs the required conda environment if necessary
* generates a network
* optimizes the network and predicts missing molecular properties
* the output contains the predictions and the graph in separate files
* depending on hardware and data set size, this might take several minutes up to hours

### workflow\_optimize\_predictions.sh

* installs the required conda environment if necessary
* predicts missing molecular properties using a specified machine learning script
* generates a network containing the formerly made predictions
* optimizes the network, including the annotated predictions
* the output contains the predictions with optimization suggestions and the graph in separate files
* depending on hardware and data set size, this might take several minutes up to hours
* please note, that in the example workflow an existing model is used for a demo prediction
* building a new predictor with the provided scripts may take several hours

### Further Advice

* execute the shell scripts provided in this directory
* for the usage of Network Balance Scaling as a predictor, please try and modify workflow\_predict.sh
* for the usage of Network Balance Scaling to optimize existing predictions, please try and modify workflow\_optimize\_predictions.sh
  * this workflow contains a precalculated model
  * new models can be created with our inhouse predictor based on deepchem using the script predictors/bace\_predictor\_train.sh
  * to generate predictions with an existing model independent from the workflow, please try the script predictors/bace\_predictor\_predict.sh
* please note, that our workflows only include the BACE data set of moleculenet due to a vast RAM requirement for larger data sets of this benchmark
  * if you are interested in the other data sets of this benchmark, feel free to construct your pipeline and try our helper scripts in the utils directory
  * you can find precalculated benchmark scenarios and the original CSV files from the moleculenet benchmark in the data directory
* **note** that all scripts should be startet from main directory in order of the path management
* to evaluate the results you achieved, please check out our script **utils/evaluate\_RMSD.py**

## Dependencies

* python >= 3.6 is needed to run the scripts
* conda is required (anaconda or miniconda); conda-environments are built automaticaly
* if you use another python environment, please make sure to provide the below-listed dependencies

### Tool Dependencies
These listed versions are tested, other may also work.

* python-igraph 0.8.3 or 0.10.1
* cvxpy 1.1.10 or 1.2.1
* rdkit 2020.09.04 or 2022.03.05

### Hardware Dependencies

No non-standard hardware is required. However, problems occurred during python package installation on Windows 10 and 11 systems. Due to the vast RAM requirement, we recommend the use of a device with at least 16GB RAM and more for the use of larger data sets. This software was successfully tested on:

* openSUSE Leap 15.1
* SLES 12.5 (SUSE Linux Enterprise Server)
* Ubuntu 20.04.5
* macOS Big Sur Version 11.4
* macOS Monterey Version 12.6 arm64 (MacBook Pro, M1, 2020)

## generate\_network.py

A script for creating networks with or without existing predictions at red vertices.

## optimize\_network.py

A script for optimizing networks. It can be used to either predict unknown properties or to optimize existing predictions depending on the option `-p`

# Predictors

## Integrated Predictor

The integrated predictor is based on DeepChem (https://github.com/deepchem/). Thus, the corresponding python scripts **predictor/build\_model.py** and **predictor/predictor.py** additionally underly the MIT License. Our integrated predictor comprises:

* a trained graph convolution regression model for the BACE data set
* a trained message passing neural network model for the BACE data set
* the Python script **predictor/build\_model.py**, which can be used to create a model
* the Python script **predictor/predictor.py**, which can be used to create predictions using an existing model
* the bash script **predictor/bace\_predictor\_train.sh**, which trains a model based on the BACE data set
* the bash script **predictor/bace\_predictor\_predict.sh**, which predicts properties for the BACE test set
* training, validation and test set for BACE can be found in our **data**
* for the other data sets, please use the scripts provided in **utils** to create your own data splits and convert files with respective formats

### Integrated Predictor Dependencies

* deepchem 2.3 or 2.5
* rdkit 2020.09.04 or 2022.03.05
* tensorflow 1.14 or 2.10 (should fit to your deepchem version)

## Integrating Your Predictor

To incorporate existing predictions into your network, please mind the following:

* the resulting predictions need to be written to a SMILES file
* the SMILES file needs to follow the format **SMILES**\\t**ID**\\t**PROPERTY**
* using **workflow\_optimize\_predictions.sh** you need to replace the call of our default predictor based on deepchem (*./predictor/bace\_predictor\_predict.sh*)
* please remember to adapt the training, validation and query set at the beginning of the bash script
* **note:** Please remember that all molecules need to be included into the mmpdb index. Using the network generation with existing predictions independently from **workflow\_optimize\_predictions** requires the molecules with known properties (training and validation sets) in a first SMILES file, your predictions in a second SMILES file and the index including all (training, validation and query) molecules.

# File Formats

* the workflows are constructed to run with SD files as input
* the SD files are converted during the workflow using the scripts provided at **utils/**
* for directly using SMILES files, please note that the required format consists of three tab-separated columns:
  * **SMILES**\\t**ID**\\t**PROPERTY**
* the SMILES format is also a requirement for the input files used in mmpdb's MMP analysis