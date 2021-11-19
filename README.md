# shopty
[![Generic badge](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg)](CONTRIBUTING.md)
<a href="https://github.com/psf/black"><img alt="Code style: black" src="https://img.shields.io/badge/code%20style-black-000000.svg"></a>

# Simple Hyperparameter OPTimization in pYthon

### Install via pip
```bash
pip install shopty
```
### Install from source
```bash
git clone https://github.com/colligant/shopty
# optional: pip install flit
cd shopty && flit install
```

### What is the purpose of this tool?

Lots of other hyperparameter tuning libraries (at least the ones I've found, anyways)
require modifying a bunch of source code and make assumptions about your running environment. 

shopty is a simple library to tune hyperparameters either on your personal computer or a slurm-managed 
cluster that requires minimal code changes and uses a simple config file to do hyperparameter sweeps.

### Design
The `Supervisor` classes in `shopty` spawn (if on CPU) or submit (if on slurm) different experiments, each
with their own set of hyperparameters. Submissions are done within python by using `subprocess.call`. 

Each experiment writes a "checkpoint.txt" file to a directory assigned to it. The `Supervisor` class detects when each
experiment is done running and reads the "checkpoint.txt" file for the outcome of the experiment that wrote it.

### Source code modifications

See a simple example [here](./examples/train.py). A neural network example is
[here](./examples/train_more_complex.py).

Your training script must accept hyperparameters and a few shopty-specific variables as command-line arguments.
The shopty-specific args are accessible via
```python
from shopty import shopt_parser
argument_parser = shopt_parser()
```
and contain

```bash
--experiment_dir directory in which to run the experiment
--max_iter number of steps to run the experiment for
--load_from_ckpt whether or not to load the model/training state from <experiment_dir>/checkpoints/
```
Your code must know how to deal with all of these arguments.
When `--load_from_ckpt` is set, your training script must look for the most recent saved checkpoint in
`--experiment_dir/checkpoints/` and resume training from that state.

`--max_iter` is always going to be set - this is the number of steps to run your model for.
`--experiment_dir` is where the checkpoints are saved and where 'checkpoint.txt' is saved.
'checkpoint.txt' should contain one line that looks like this: 
`<your_metric_name_here>:<value after running training>`
Scheduling algorithms like `hyperband` use the <value after running training> to cull or keep experiments.

I've already figured out the code for this for [pytorch lightning](https://www.pytorchlightning.ai/) (PTL).
I highly recommend using PTL, as it does a lot of useful things for you under the hood.



### Hyperband (or related stop-and-start tuning algorithms)
If using the `hyperband` tuning algorithm the supervisor continues running performant experiments and throws
away bad ones. Each experiment saves its current state in its working directory. To continue the job, the supervisor
calls the experiment again with the `--load_from_ckpt` flag set. The experiment must know
how to handle this flag and perform the corresponding reloading-from-checkpoint logic.

### Random
If using the `random` tuning algorithm the supervisor just submits the number you choose and then exits.
It's up to you to monitor the spawned jobs after that.


