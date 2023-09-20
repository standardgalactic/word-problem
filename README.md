# Word Problems

Data and code to evaluate how deep transformer models need to be to learn group multiplication.

## Installation

Use Conda to create an environment called `depth` with the requisite dependencies by running this from the root directory of the project:

```bash
conda env create && conda activate depth
```

## Use

To train a model, run `python depth_test.py`. Command-line arguments to configure the training run are the arguments to the `main` function, which is the most accurate documentation for what to do. Some important ones:

- `--group`: the name of the file in the `data/` directory, without the `.csv` extension.
- `--num_layers`: how many layers in the transformer encoder.
- `--epochs`: the number of epochs to train for.

You must include exactly one of `--k` or `--max_len`. These values specify how long the sequences of the dataset are in terms of the number of elements multiplied together. If `--k` is passed, sequences will be of length 2 or `k`; if `--max_len` is passed, they will be of lengths between 2 and `max_len`. If `k` or `max_len` are greater than 2, the training set will include all sequences of length 2 plus a random subset of the longer sequences; if `k` or `max_len` are equal to 2, the 2-element sequences will be split between the train and validation sets.

### Data

Data files are generated by calling `python generate_data.py` with the following arguments:

```bash
python generate_data.py --group <group> --num_examples <num_examples> --seq_length <seq_length> --data_dir <data_dir> --seed <seed>
```

- `--group`: A string representing the group to use. Supported groups are of the form `G#`, where `G` is one of `S` (symmetric), `A` (alternating), or `Z` (cyclic), and `#` is an integer. You can also use the direct products of any of these groups by separating each with a `_x_`. So `--group=S5` generates data using elements of `S5` and `--group=A5_x_Z9` generates data using elements of `A5 x Z9`.
- `--k`: The length of the sequences to generate. Each sequence consists of `k` elements multiplied together.
- `--samples`: The number of examples to generate. If this is left blank, it will generate the maximum number of distinct sequences possible for the given group and sequence length, equal to `#(G)^k`. If this is set to be an integer, it will generate `min(samples, #(G)^k)` examples; note that we cap the number of examples to ensure that any partitions of the generate datasets are guaranteed to be sequence-wise disjoint. If `samples` is less than `#(G)^k`, examples will be generated randomly without replacement.
- `--data_dir`: The directory to save the data to. If this is left blank, it will save to a `data/` directory in the project root.
- `--seed`: The random seed to use for generating the data.
- `--overwrite`: Whether to overwrite an existing data file for the given values for `group` and `k`.

Data files are named `group=k.csv`.

```csv
length,input,target
```

where `length` is equal to `k`, `input` is a series of space-separated integers corresponding to the element index of the group as defined in the `abstract_algebra` object, and `target` is the element the sequence of input elements multiplies to (again, as the element index of the group object).

### Logging

Run data is logged to Weights&Biases; add `WANDB_API_KEY=##################` to a `.env` file in the root directory of the project to automatically configure the W&B logger
