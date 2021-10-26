# datc-rdf-flow-tuner
The flow-tuner repository consists of AutoTuner (blockbox hyperparameter optimization) for RTL-to-GDS flow, along with config json preset files and scripts, using the open-source RTL-to-GDS flow of the [OpenROAD tool](https://github.com/The-OpenROAD-Project) and [METRICS2.1](https://github.com/ieee-ceda-datc/datc-rdf-Metrics4ML), an open-source format for collecting design and tool metrics for an RTL-to-GDS flow.
This scripts also available at [OpenROAD-flow-scripts commit](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666), and this repo will update continusioly to follow the OpenROAD AutoTuner.


## Contents
* [Repository Structure](#repository-structure)
* [AutoTuner: Getting Started](#autotuner-getting-started)
* [Citation](#citation)

## Repository Structure
The repository contains the **'config_preset'** folder and **'scripts'** in the top. 


### scripts
This directory contains main top-level script files. Script files are divided according to each searching algorithm. 

The script calls [genMassive.py](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/2e0de4384ca207593c80aa297064f62187b0c666/flow/util/genMassive.py) script to generate runscript for OpenROAD flow. 
The script also calls [genMetrics.py](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/blob/2e0de4384ca207593c80aa297064f62187b0c666/flow/util/genMetrics.py) script to collect the metrics json file.

Supported searching algorithms are as follows.
- Random/Grid Search 
- Population Based Training ([PBT](https://deepmind.com/blog/article/population-based-training-neural-networks))
- Tree Parzen Estimator ([HyperOpt](http://hyperopt.github.io/hyperopt))
- Bayesian + Multi-Armed Bandit ([AxSearch](https://ax.dev/))
- Tree Parzen Estimator + Covariance Matrix Adaptation Evolution Strategy ([Optuna](https://optuna.org/))
- Evolutionary Algorithm ([Nevergrad](https://github.com/facebookresearch/nevergrad))


### config preset
Each config preset includes the pre-defined input hyperparameter name, type, range and step size for specific platform and design testcase as a JSON format. When type is int and step = 0, it means constant value. When type is float and step = 0, it means continuous range. User can manually modify or make a new config JSON file to define the input hyperparameters.




## AutoTuner: Getting Started

### Requirements
To run AutoTuner scripts, it requires [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666). 
Currently, AutoTuner uses OpenROAD flow scripts and METRIC2.1 collection scripts, and uses Python APIs, Ray and Tune. Thus, the following are required.


- [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) 
- [Ray and Tune](https://docs.ray.io/en/latest/installation.html)
- [tensorboardX](https://github.com/lanpa/tensorboardX)


### How to Run
1. Place the scripts under the 'flow/util/autotuner' directory in the [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) installed path.

2. Set the input arguments of the scripts.
- p {platform}: platform. E.g., sky130hd
- d {design}: design. E.g., ibex
- e {experiment name}: user-defined experiment name. This name is used to the created directory in the flow/util/autotuner/results/{experiment name}
- j {number of jobs}: set the number of concurrent jobs. (***Beware of memory usage)
- n (number of trials): set the target number of trials. (***500~1000 is appropriate)

3. Run python script with the from 'flow/' directory.


### GUI 
Basically, progress is displayed at the terminal where you run, and when all runs are finished, the the results are displayed. 
You could find the "Best config found" in the screen.

To use TensorBoard GUI, run "tensorboard --logdir=./util/autotuner/results/{your experimental name}". While TensorBoard running, you can open the webpage http://localhost:6006/ to see the GUI.

## Citation
Please cite the following paper

* J. Jung, A. B. Kahng, S. Kim and R. Varadarajan, "METRICS2.1 and Flow Tuning in the IEEE CEDA Robust Design Flow and OpenROAD", Proc. ACM/IEEE International Conference on Computer-Aided Design, 2021. (Invited) 
