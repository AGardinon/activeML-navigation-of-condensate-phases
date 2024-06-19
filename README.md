# RobotLabExperiments

Repository intended for storing and managing the files generated during the Robotlab experiments.

Folder organization:
------------
    master_file/
    ├── master_file_version_0.csv
    ├── master_file_version_1.csv
    └── master_file_version_N.csv

    experiments/
    └── 1.experimentXXX
        ├── cycles
        │   ├── cycle_0
        │   │   ├── experimentXXX_cycle0_ouput_points.csv
        │   │   ├── experimentXXX_cycle0_ouput_points.ndx
        │   │   ├── experimentXXX_cycle0_validated_points.csv
        │   │   └── cycle_0_config.yaml
        │   ├── cycle_1
        │   │   ├── experimentXXX_cycle1_ouput_points.csv
        │   │   ├── experimentXXX_cycle1_ouput_points.ndx
        │   │   ├── experimentXXX_cycle1_validated_points.csv
        │   │   └── cycle_1_config.yaml
        │   └── cycle_N
        │
        └── dataset
            ├── DOE_experimentXXX.csv
            └── phase_diagram_config_experimentXXX.json
------------

## Runnig the experiments

To run the experiments the [ActiveLearningCLassiFier](https://github.com/AGardinon/ActiveLearningCLassiFier) package is required.
The package allow to apply the point selection and classification strategies to an input dataset for each cycle.

The general cycle is evaluated by running the `script.py` provided in the repository:
```bash
python src/script/activeCLFexperiment.py -c cycle_X_config.yaml
```
The `experiment_config.yaml` needs to be adjusted depending on the needs of the cycle.
In the following I will provide a couple of practical examples.

### Running **Cycle**$_{0}$

**Cycle**$_{0}$ refers to the initial cycle, where no information of the system is know.
The aim is to search for an _interesting_ set of starting points to then exploit with an active learning strategy.

```yaml
experimentID: 'experimentXXX'
cycleN: 0
rngSeed: 73

dataset: '../../dataset/DOE_experimentXXX.csv'
targetVariable: 'Phase'

validatedset:

newBatch: 18

clfModel: 
kParam1: 
kParam2: 

acqMode: 
entropyDecimals: 

screeningSelection: 
```

### Running **Cycle**$_{1}$ and further **Cycles**

To run **Cycle**$_{1}$ and further we require more information in the config file, as we need to search for new points and fit a classification model.

```yaml
experimentID: 'experimentXXX'
cycleN: 1
rngSeed: 73

dataset: '../../dataset/DOE_experimentXXX.csv'
targetVariable: 'Phase'

validatedset: ../cycle_0/experimentXXX_cycle0_validated_points.csv

newBatch: 18

clfModel: 'GaussianProcessClassifier'
kParam1: 1.0
kParam2: 1.0

acqMode: 'exploration'
entropyDecimals: 2

screeningSelection: 'FPS'
```

The config contains all the necessary information to run a single cycle search of new points based on a previous knowledge of a phase diagram.
The previous knowledge in this case is accounted by the argument:
```yaml
validatedset: ../cycle_0/experimentXXX_cycle0_validated_points.csv
```
that when specified merge validated (experimetally) points into the dataset.

All the other arguments are either self-explanatory or they can be understood by following the implementation of the [ActiveLearningCLassiFier](https://github.com/AGardinon/ActiveLearningCLassiFier).


## Managing the results files

During each cycle a bunch of experimental designs will be selected for the validation phase.
After said validation two main files will be updated:
1.  `N.experimentXXX/dataset/DOE_experimentXXX.csv`
2.  `master_file/master_file_version_X.csv` 

The first one serves as the main dataset used to select points from for a specific experiment.
It effectively represents the phase diagram space that it is going to be explored.

The second serves as a container to store and keep track of all the experimental designs that were tested during all the different experiments.

### End of the cycle routine

End-of-the-cycle routine refers to the actions required after the new points are selected by the active learning strategy and they are ready to be validated.
The routine contains the following steps:
1.  append the new points to the master file and save a new master file version.
2.  experimental validation of the points (robot experiments)
3.  update the validated points values in the master file (saved from p.1)
4.  save the updated version of the master file, overwriting to the save version created in p.1 