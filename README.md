# DATC RDF Flow AutoTuner

This repository provides DATC RDF AutoTuner, a "no-human-in-loop" parameter tuning framework for commercial and academic RTL-to-GDS flows. AutoTuner provides a generic interface where users can define parameter configuration as a JSON object. It enables AutoTuner to easily support various tools and flows. AutoTuner also utilizes [METRICS2.1](https://github.com/ieee-ceda-datc/datc-rdf-Metrics4ML) to capture PPA of individual search trials. With the abundant features of METRICS2.1, users can explore various reward functions that steer the flow autotuning to different PPA goals.

This repo is originated from the official [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) repository.
We will update continuously to follow the OpenROAD-flow-scripts AutoTuner.


## Repository Structure

The repository consists of two sub-directories: `config_preset` and `scripts`.


### `scripts`

This directory contains top-level Python scripts for each of the supported search algorithms.

The script calls [genMassive.py](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/2e0de4384ca207593c80aa297064f62187b0c666/flow/util/genMassive.py) to generate run script for the OpenROAD flow. 
The script also calls [genMetrics.py](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/2e0de4384ca207593c80aa297064f62187b0c666/flow/util/genMetrics.py) to collect the metrics JSON file.

Supported search algorithms are as follows.

* Random/Grid Search 
* Population Based Training ([PBT](https://deepmind.com/blog/article/population-based-training-neural-networks))
* Tree Parzen Estimator ([HyperOpt](http://hyperopt.github.io/hyperopt))
* Bayesian + Multi-Armed Bandit ([AxSearch](https://ax.dev/))
* Tree Parzen Estimator + Covariance Matrix Adaptation Evolution Strategy ([Optuna](https://optuna.org/))
* Evolutionary Algorithm ([Nevergrad](https://github.com/facebookresearch/nevergrad))

User-settable coefficient values (`C_power`, `C_perform`, `C_area`) of three objectives to set the direction of tuning are written in each script.



### `config_preset`

Each config preset includes the pre-defined input parameter names, types, ranges and step sizes for a specific platform and design testcase as a JSON format. When type is `int` and `step` is `0`, it defines a constant parameter. When type is `float` and `step` is `0`, it defines a continuous real parameter. Users can manually modify or make a new config JSON to define the input parameters.


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
    - `-e <experiment name>`: user-defined experiment name. This name is used to create workspace in `flow/util/autotuner/results/<experiment name>`
    - `-j <number of jobs>`: set the number of concurrent jobs. (**Beware of memory usage**.)
    - `-n <number of trials>`: set the target number of trials. (**500~1000 is appropriate.)

3. Run python script with the from `flow/` directory.


### GUI 

Basically, progress is displayed at the terminal where you run, and when all runs are finished, the results are displayed. 
You could find the "Best config found" on the screen.

To use TensorBoard GUI, run `tensorboard --logdir=./util/autotuner/results/<experimental name>`. While TensorBoard running, you can open the webpage http://localhost:6006/ to see the GUI.

## Citation

Please cite the following paper.

* J. Jung, A. B. Kahng, S. Kim and R. Varadarajan, "METRICS2.1 and Flow Tuning in the IEEE CEDA Robust Design Flow and OpenROAD", Proc. ACM/IEEE International Conference on Computer-Aided Design, 2021.
