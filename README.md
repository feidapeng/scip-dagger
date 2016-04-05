# Learning Node Selection/Pruning Policy in Branch-and-Bound ILP Solver
Code for the paper "Learning to Search in Branch and Bound Algorithms" published in NIPS 2014. 
The main goal is to **speedup ILP solvers by prioritizing and pruning nodes** during exploration.
Currently it's tested on Linux only.

## Dependencies
### SCIP
This implementation is base on the open source ILP solver [SCIP 3.1.0](http://scip.zib.de/).
To have access to more features during solving, I had to change/add some API in the original SCIP package.
You can either get the affected source director src/scip from [this repo](https://github.com/hhexiy/scip) and make changes accordingly to your SCIP package,
or download the whole SCIP package I used [here](http://www.umiacs.umd.edu/~hhe/scip-3.1.0.tgz).
Then you need to change `SCIPDIR` in the `Makefile`.
### CPLEX
We used [CPLEX](http://www-03.ibm.com/software/products/en/ibmilogcpleoptistud) as the LP solver for SCIP. 
But you can also use other LP solvers. Please see details in SCIP documentation.
### LIBLINEAR
To train the (linear) policy, you need [LIBLINEAR](https://www.csie.ntu.edu.tw/~cjlin/liblinear/).
I used version 1.94 but it should be compatible with later versions.
After compiling put the executable `train` and `predict` in `bin`.
LIBLINEAR does not support instance weights; please also download [LIBLINEAR-weights](http://ntu.csie.org/~cjlin/libsvmtools/#weights_for_data_instances) and put the executable `train-w` in `bin`.

## Data preparation
This algorithm learns from *solved* problems.
Before learning, you need to put ILP problems in `dat/dataset/{train,dev,test}` and their corresponding solutions (with the same filename prefix) under `sol/dataset/{train,dev,test}`, where `dataset` should be the name of your dataset.
See `sample-dat` and `sample-sol` for example.
To obtain solutions by SCIP, you can use `scripts/run_scip.sh`.

## Learning the policy
To compile, run `make`. This will generate `bin/scipdagger`.
The main DAgger loop is in `scripts/train_bb.sh`. 
For example,
```
scripts/train_bb.sh -d sample/train -p 2 -n 24 -c 2 -w 8 -e sample -x .lp.gz
```
- `-d`: learn a policy from problems in `dat/sample/train`. 
- `-p` and `-n`: go through the whole training set for 2 passes and train a policy for every 24 problems. The total number of training examples should be dividable by the argument of `-n`.
- `-e`: specify the experiment name; used for logging purposes.
- `-x`: specify the suffix of problems in `dat`.
- `-c` and `-w`: hyperparameters for LIBLINEAR. `-c` is the SVM penalty parameter and we tried `{0.25, 0.5, 1, 2, 4, 8}`; `-w` is the weight on positive instances since the classification is highly imbalanced, and we tried `{1, 2, 4, 8}`.

**Note**: It will generate temporary training files (potentially large!) for LIBLINEAR; set `scratch` to point to a tmp location.

## Evaluation
To test the learned policy, use `scripts/test_bb.sh`.
Besides arguments the above arguments, you need to pass it the pruning policy (`-k`) and the selection policy (`-s`), whose locations are specified in `scripts/train_bb.sh`.

In addition, we may want to compare it with other methods.
`scripts/compare.sh` reads results from logs generated by `test_bb.sh` then compares it with SCIP and Gurobi using the same node or time constraints.
We can also rank the policy features (`scripts/rank_features.py`) by weights of a learned model; or run statistical tests (`scripts/ttest.sh`).
