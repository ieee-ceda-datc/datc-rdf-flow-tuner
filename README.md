# DATC RDF Flow AutoTuner

This repository provides _AutoTuner_, a "no-human-in-loop" parameter tuning framework for commercial and academic RTL-to-GDS flows. AutoTuner provides a generic interface where users can define parameter configuration as JSON objects. This enables AutoTuner to easily support various tools and flows. AutoTuner also utilizes [METRICS2.1](https://github.com/ieee-ceda-datc/datc-rdf-Metrics4ML) to capture PPA of individual search trials. With the abundant features of METRICS2.1, users can explore various reward functions that steer the flow autotuning to different PPA goals.

This repo is originated from the official AutoTuner `flow/util/distributed.py` in [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/a102324a030796b99b164a1f83b54da514860342) (ORFS) repository, and is to be updated continuously. 

AutoTuner provides two main functionalities as follows.
* Automatic hyperparameter tuning framework for ORFS
* Parametric sweeping experiments for ORFS


## Changelog

12/07/2021 - Update AutoTuner v2 version. The detailed changes are as follows.
* Change the location and the name of the script in ORFS repository
* Support for parameter sweeping experiments
* Integrate scripts that were divided by search algorithm into one (distributed.py)
* Support job distribution feature by using multiple Google cloud platform (GCP) instances

10/25/2021 - Commit AutoTuner v1 version (autotuner-v1 tag in the ORFS repository [link](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tags)).

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

User-settable coefficient values (`C_power`, `C_perform`, `C_area`) of three objectives to set the direction of tuning are written in the script.
Each coefficient is expressed as a global variable at the top of the script (coeff_erform=`C_perform`, coeff_power=`C_power` and coeff_area=`C_area`).
Efforts to optimize each of the objectives are proportional to the specified coefficients.


### `config_preset`

This directory contains a set of predefined input parameter config files.
Each config file defines a set of tunable tool parameters, described as JSON objects.
Here's an example of the config file.

```json
{
    "_SDC_FILE_PATH": "constraint.sdc",     // pointer of the base SDC file for modification
    "_SDC_CLK_PERIOD": {                    // parameter name (clock period) for sweeping/tuning. 
        "type": "float",                    // parameter type for sweeping/tuning
        "minmax": [                         // min-to-max range for sweeping/tuning. 
            1.0,                            // The unit follows the default value of each technology std cell library.
            3.7439
        ],
        "step": 0                           // step ‘0’ for type ‘float’ means continuous step for sweeping/tuning
    },
    "CORE_MARGIN": {                        // parameter name (die-to-core margin) for sweeping/tuning
        "type": "int",
        "minmax": [
            2,
            2
        ],
        "step": 0                           // step ‘0’ for type ‘int’ means the constant parameter.
    },
}
```

Each parameter is defined with a key-value pair, where the key defines the parameter name, and the value is another JSON object defining the parameter specification, which has `type`, `minmax`, and `step` attributes. Attribute `type` defines the data type of the parameter, and `minmax` defines the parameter's range. When `type` is `int` and `step` is `0`, this defines a constant parameter. When `type` is `float` and `step` is `0`, this defines a continuous real parameter. 

Users can manually modify or make their own config JSON file to define the tunable parameters.


## Getting Started

### Prerequisite

To run AutoTuner scripts, it requires [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/a102324a030796b99b164a1f83b54da514860342). 
Currently, AutoTuner uses OpenROAD flow scripts and METRIC2.1 collection scripts and uses Python APIs as well as Ray Tune. Thus, the following are required.

- [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/2e0de4384ca207593c80aa297064f62187b0c666) and related packages
- [Ray and Tune](https://docs.ray.io/en/latest/installation.html) and related packages
- [TensorboardX](https://github.com/lanpa/tensorboardX)


### How to Run

1. Place the scripts under the `flow/util/` directory in the [OpenROAD-flow-scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/a102324a030796b99b164a1f83b54da514860342) installed path.

2. “distributed.py” scripts handles sweeping and tuning of ORFS parameters.
 
For both sweep and tune modes <mode>:
    python3 distributed.py -h
 
Note: the order of the parameters matter. Arguments --design, --platform and
--config are always required and should precede <mode>.
 
AutoTuner:
    python3 distributed.py tune -h
Example:
    python3 distributed.py --design gcd --platform sky130hd \
                           --config ../designs/sky130hd/gcd/autotuner.json \
                           tune
    
 
Parameter sweeping:
    python3 distributed.py sweep -h
    Example:
    python3 distributed.py --design gcd --platform sky130hd \
                           --config distributed-sweep-example.json \
                           sweep


3. Run Python script at the `flow/util/` directory.

### List of input arguments
    
* Target design
    - --design
        - Name of the design for autotuning
    - --platform
        - Name of the platform for autotuning
* Experiment setup
    - --config
        - Configuration file that sets which knobs to use for autotuning
    - --experiment
        - Experiment name. This parameter is used to prefix the FLOW_VARIANT and to set the Ray log destination
    - --resume
        - Resume previous run
* Git setup
    - --git-clean
        - Clean binaries and build files 
    - --git-clone
        - Force new git clone
    - --git-clone-args
        - Additional git clone arguments
    - --git-latest
        - Use the latest version of OR app
    - --git-or-branch
        - OR app branch to use
    - --git-orfs-branch
        - ORFS branch to use
    - --git-url
        - ORFS repo URL to use
    - --build-args
        - Additional arguments given to ./build_openroad.sh
* For AutoTuner
    - --algorithm
        - Search algorithm to use for autotuning
    - --eval
        - Evaluate function to use with search algorithm
    - --samples
        - Number of samples for autotuning
    - --iterations
        - Number of iterations for autotuning
    - --reference
        - Reference file for use with ‘PPAImprov’ evaluation function
    - --perturbation
        - Perturbation interval for PopulationBasedTraining
    - --seed
        - Random seed for parameter selection during autotuning
* Workload
    - --jobs
        - Max number of concurrent jobs
    - --openroad-threads
        - Max number of threads OR app can use
    - --server
        - The address of Ray server to connect
    - --port
        - The port of Ray server to connect
    - -v, --verbose
        - Verbosity level
            - 0: only print Ray status
            - 1: also print training stderr
            - 2: also print training stdout

    
### GUI 

Basically, progress is displayed at the terminal where you run, and when all runs are finished, the results are displayed. 
You could find the "Best config found" on the screen.

To use TensorBoard GUI, run `tensorboard --logdir=./<logpath>`. While TensorBoard is running, you can open the webpage http://localhost:6006/ to see the GUI.
    


## Citation

Please cite the following paper.

* J. Jung, A. B. Kahng, S. Kim and R. Varadarajan, "METRICS2.1 and Flow Tuning in the IEEE CEDA Robust Design Flow and OpenROAD", [(.pdf)](https://vlsicad.ucsd.edu/Publications/Conferences/388/c388.pdf), [(.pptx)](https://vlsicad.ucsd.edu/Publications/Conferences/388/c388.pptx), [(.mp4)](https://vlsicad.ucsd.edu/Publications/Conferences/388/c388.mp4), Proc. ACM/IEEE International Conference on Computer-Aided Design, 2021.

