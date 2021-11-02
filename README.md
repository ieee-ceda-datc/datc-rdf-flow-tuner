# DATC RDF Flow AutoTuner

This repository provides _AutoTuner_, a "no-human-in-loop" parameter tuning framework for commercial and academic RTL-to-GDS flows. AutoTuner provides a generic interface where users can define parameter configuration as JSON objects. This enables AutoTuner to easily support various tools and flows. AutoTuner also utilizes [METRICS2.1](https://github.com/ieee-ceda-datc/datc-rdf-Metrics4ML) to capture PPA of individual search trials. With the abundant features of METRICS2.1, users can explore various reward functions that steer the flow autotuning to different PPA goals.

This repo is originated from the official AutoTuner in the `flow/util/autotuner/` directory in [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) repository, and is to be updated continuously.


## Repository Structure

The repository consists of two subdirectories: `scripts` and `config_preset`.

### `scripts`

This directory contains top-level Python scripts, each of which implements a different search algorithm. Currently supported search algorithms are as follows.

* Random/Grid Search 
* Population Based Training ([PBT](https://deepmind.com/blog/article/population-based-training-neural-networks))
* Tree Parzen Estimator ([HyperOpt](http://hyperopt.github.io/hyperopt))
* Bayesian + Multi-Armed Bandit ([AxSearch](https://ax.dev/))
* Tree Parzen Estimator + Covariance Matrix Adaptation Evolution Strategy ([Optuna](https://optuna.org/))
* Evolutionary Algorithm ([Nevergrad](https://github.com/facebookresearch/nevergrad))

The script calls [genMassive.py](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/2e0de4384ca207593c80aa297064f62187b0c666/flow/util/genMassive.py) to generate run script for the OpenROAD flow. 
The script also calls [genMetrics.py](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/2e0de4384ca207593c80aa297064f62187b0c666/flow/util/genMetrics.py) to collect the METRICS2.1 JSON file.

User-settable coefficient values (`C_power`, `C_perform`, `C_area`) of three objectives to set the direction of tuning are written in each script.
Each coefficient is expressed as a global variable at the top of each script (coeffPerform=`C_perform`, coeffPower=`C_power` and coeffArea=`C_area`).
Efforts to optimize each of the objectives are proportional to the specified coefficients.

### `config_preset`

This directory contains a set of predefined input parameter config files.
Each config file defines a set of tunable tool parameters, described as JSON objects.
Here's an example of the config file ([config_fmax_asap7-gcd.json](./config_preset/config_fmax_asap7-gcd.json)).

```json
{
    "CLK_PERIOD": {"type": "float", "minmax": [100, 400], "step": 0 },
    "CORE_UTIL": {"type": "int", "minmax": [5, 99], "step": 1},
    "ASPECT_RATIO": {"type": "float", "minmax": [0.1, 2.0], "step": 0 },
    "CORE_DIE_MARGIN": {"type": "int", "minmax": [2,2], "step": 0 },
    "GP_PAD": {"type": "int", "minmax": [0,4], "step": 1 },
    "DP_PAD": {"type": "int", "minmax": [0,4], "step": 1 },
    "LAYER_ADJUST": {"type": "float", "minmax": [0.1,0.7], "step": 0 },
    "PLACE_DENSITY_LB_ADDON": {"type": "float", "minmax": [0.00,0.99], "step": 0 },
    "FLATTEN": {"type": "int", "minmax": [0,1], "step": 1 },
    "PINS_DISTANCE": {"type": "int", "minmax": [1,3], "step": 1 },
    "CTS_CLUSTER_SIZE": {"type": "int", "minmax": [10,40], "step": 1 },
    "CTS_CLUSTER_DIAMETER": {"type": "int", "minmax": [80,120], "step": 1 },
    "GR_OVERFLOW": {"type": "int", "minmax": [1,1], "step": 0 }
}
```

Each parameter is defined with a key-value pair, where the key defines the parameter name, and the value is another JSON object defining the parameter specification, which has `type`, `minmax`, and `step` attributes. Attribute `type` defines the data type of the parameter, and `minmax` defines the parameter's range. When `type` is `int` and `step` is `0`, this defines a constant parameter. When `type` is `float` and `step` is `0`, this defines a continuous real parameter. 

Users can manually modify or make their own config JSON file to define the tunable parameters.


## Getting Started

### Prerequisite

To run AutoTuner scripts, it requires [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666). 
Currently, AutoTuner uses OpenROAD flow scripts and METRIC2.1 collection scripts and uses Python APIs as well as Ray Tune. Thus, the following are required.

- [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) and related packages
- [Ray and Tune](https://docs.ray.io/en/latest/installation.html) and related packages
- [TensorboardX](https://github.com/lanpa/tensorboardX)


### How to Run

1. Place the scripts under the `'flow/util/autotuner` directory in the [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) installed path.

2. Set the input arguments of the scripts.

    - `-p <platform>`: platform, e.g., `sky130hd`.
    - `-d <design>`: design, e.g., `ibex`.
    - `-e <experiment name>`: user-defined experiment name. The experiment workspace will be created in `flow/util/autotuner/results/<experiment name>`.
    - `-j <number of jobs>`: set the number of concurrent jobs. (**Beware of memory usage**.)
    - `-n <number of trials>`: set the target number of trials. (**500~1000 is appropriate.)

3. Run Python script at the `flow/` directory.


### GUI 

Basically, progress is displayed at the terminal where you run, and when all runs are finished, the results are displayed. 
You could find the "Best config found" on the screen.

To use TensorBoard GUI, run `tensorboard --logdir=./util/autotuner/results/<experimental name>`. While TensorBoard is running, you can open the webpage http://localhost:6006/ to see the GUI.

## Citation

Please cite the following paper.

* J. Jung, A. B. Kahng, S. Kim and R. Varadarajan, "METRICS2.1 and Flow Tuning in the IEEE CEDA Robust Design Flow and OpenROAD", [(.pdf)](https://vlsicad.ucsd.edu/Publications/Conferences/388/c388.pdf), [(.pptx)](https://vlsicad.ucsd.edu/Publications/Conferences/388/c388.pptx), [(.mp4)](https://vlsicad.ucsd.edu/Publications/Conferences/388/c388.mp4), Proc. ACM/IEEE International Conference on Computer-Aided Design, 2021.
