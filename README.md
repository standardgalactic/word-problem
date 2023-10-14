# Word Problems

Data and code to evaluate how deep transformer models need to be to learn group multiplication.

## Installation

Use Conda to create an environment called `depth` with the requisite dependencies by running this from the root directory of the project:

```bash
conda env create && conda activate depth
```

## Use

### Training

To train a model, run `python main.py train`. Command-line arguments to configure the training run are the arguments to the `main` function, which is the most accurate documentation for what to do. Some important ones:

- `--group`: the name of the group to train on.
- `--k`: the length of the sequences to train on. Must be greater than 1.
- `--num_layers`: how many layers in the transformer encoder.
- `--epochs`: the number of epochs to train for.

Training on sequences of length `k` means that all `G=k` sequences will be split between the train and test sets. Additionally, all sequences of length 2 will be included in the training set. By default, lengths are 'strict' in the sense that models will see only sequences of length 2 and `k`; to include sequences of length 2 <= `m` <= `k`, you can pass the `--nostrict_len` flag.

```bash
# Trains a model on all sequences of length 2, 4 on data from S5
python main.py train --group S5 --k 4

# Trains a model on all sequences of length 2, 3, 4 on data from S5
python main.py train --group S5 --k 4 --nostrict_len
```

The combination of `group` and `k` determines which data files to use. Data files are stored by default in the `data/` directory and have the name `group=k.csv`.

### Training MLP Baselines

As a sanity check for what a single-layer should be able to compute, we can train a MLP to learn binary multiplication. We don't train with a train/test split since we are only concerned with whether or not a single layer can learn the function, not how well it generalizes. The only required argument is `--group`.

```bash
# Train a single-layer MLP on binary multiplication in Z60
python main.py train_mlp --group Z60
```

### Data

Data files are generated by calling `python generate_data.py` with the following arguments:

- `--group`: A string representing the group to use. Supported groups are of the form `G#`, where `G` is one of `S` (symmetric), `A` (alternating), or `Z` (cyclic), and `#` is an integer. You can also use the direct products of any of these groups by separating each with a `_x_`. So `--group=S5` generates data using elements of `S5` and `--group=A5_x_Z9` generates data using elements of `A5 x Z9`.
- `--k`: The length of the sequences to generate. Each sequence consists of `k` elements multiplied together.
- `--samples`: The number of examples to generate. If this is left blank, it will generate the maximum number of distinct sequences possible for the given group and sequence length, equal to `#(G)^k`. If this is set to be an integer, it will generate `min(samples, #(G)^k)` examples; note that we cap the number of examples to ensure that any partitions of the generate datasets are guaranteed to be sequence-wise disjoint. If `samples` is less than `#(G)^k`, examples will be generated randomly without replacement.
- `--data_dir`: The directory to save the data to. If this is left blank, it will save to a `data/` directory in the project root.
- `--seed`: The random seed to use for generating the data.
- `--overwrite`: Whether to overwrite an existing data file for the given values for `group` and `k`.

```bash
# Generate 100k 5-element sequences & their reductions from S5
python generate_data.py --group S5 --k 5 --samples 100000
```

Data files are named `group=k.csv`.

```csv
length,input,target
```

where `length` is equal to `k`, `input` is a series of space-separated integers corresponding to the element index of the group as defined in the `abstract_algebra` object, and `target` is the element the sequence of input elements multiplies to (again, as the element index of the group object).

### Logging

Run data is logged to Weights&Biases; add `WANDB_API_KEY=##################` to a `.env` file in the root directory of the project to automatically configure the W&B logger
