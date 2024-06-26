# State tune for V5

## Whats in the box

`README.md` This file, cool stuff here

`requirements.txt` File including the needed pip packages for this to work

`rwkv_vocab_v20230424.txt` RWKV tokenizer vocab file for Eagle

`tokenizer.py` Tokenizer module for eagle

`notebook.ipynb` A jupyter notebook to explore state tuning

### Whats not in the box

`model.pth` The model is not included, Here is some v5 models for download: [Eagle 0.6->7B](https://huggingface.co/BlinkDL/rwkv-5-world/tree/main) and [EagleX v2](https://huggingface.co/RWKV/v5-EagleX-v2-7B-pth/tree/main)

## Install Requirements
```bash
pip install -r ./requirements.txt
```

## Run Script

Here are the available command line options and their defaults. Running `python3 ./state-tune.py` is equivilent to the below:
these also work as ENV variables in the format --key=value

**Training**
```sh
python3 ./train.py \
--learningrate 0.01 \
--batchsize 4 \
--exit_loss 1.5 \
--max_epochs 10 \
--dataset_walk shuffled \
--model_url "" \
--data_url "" \
--huggingface_dataset "lonestar108/naughty-chat" \
--model_location model.pth \
--data_path data.jsonl \
--save_filename state.pth \
--prompt_cutoff -1 \
--completion_cutoff -1 \
--max_time 3600 \
--prompt_formatter "user: {input}\n\nassistant:" \
--response_formatter " {output}"
```

**Testing**

This uses a temperature of 0

```sh
python3 ./train.py \
--model_location model.pth \
--save_filename state.pth \
--prompt "user: How is you day going?\n\nassistant:" \
--device cuda
```

## Explaination of arguments

**Learning Rate**

This is how hard the model tries to fit the data

**Batch size**

How much data to simultaniously do at once

**Exit loss**

Stop training if loss falls below this number

**Dataset walk**

How to sample the dataset for batches

values are:

`random`: just get some random lines for each batch

`sequential`: do everything in order

`shuffled`: randomize the order, but dont repeat anything

**Model url**

if the file does not exist, download from this file location

**Data url**

if the jsonl file does not exist, download it.

**Huggingface dataset**

Loads a huggingface dataset and puts it into a jsonl format, takes precedence over data url

**Model location**

Path to model file

**Data path**

Path to jsonl file

**Save filename**

path to save the tuned state to

**Prompt cutoff**

Only look at the last X tokens of the prompt, this saves you from oom, -1 to disable

**Completion cutoff**

Only look at the first X tokens of completion, this saves you from oom, -1 to disable

**Max time**

End training after this many seconds

**Prompt formatter**

Format the prompt into something more palatable for the model, {x} is replaced with that key in the jsonl

prompt is masked during training.

**Response formatter**

Format the response into something more palatable for the model, {x} is replaced with that key in the jsonl


## Required VRAM

`1B5`: 8GB

*More data incoming*

## Explaination on output format

output file is as such, with tensor dimensions

```py
{
    "blocks.0.ffn.shift.state": (1,1,dims),
    "blocks.0.att.shift.state": (1,1,dims),
    "blocks.0.att.wkvstae" : (heads, dims/heads, dims/heads),
    "blocks.1.ffn.shift.sta..."
}
```

## Debugging transpositions

Some implementations may handle the state with the last two dimensions transposed,

you may need to do .transpose(-1,-2) for your implementation