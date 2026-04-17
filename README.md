# HiQST: A Unified Hierarchical Quantized Skill Framework for Multitask and Few-Shot Robotic Manipulation

[**Installation**](#installation) | [**Dataset Download**](#dataset-download) | [**Training**](#training) | [**Evaluation**](#evaluating) | 


<hr style="border: 2px solid gray;"></hr>

## Installation

Please run the following commands in the given order to install the dependency for hiqst
```
conda create -n hiqst python=3.10.14
conda activate hiqst
python -m pip install torch==2.2.0 torchvision==0.17.0
```
Note: Above automatically installs metaworld as python packages

Install LIBERO seperately
```
git clone https://github.com/Lifelong-Robot-Learning/LIBERO.git
cd LIBERO
python -m pip install -e .
```
Note: All LIBERO dependencies are already included in hiqst/requirements.txt

## Dataset Download
LIBERO: Please download the libero data seperately following their [docs](https://lifelong-robot-learning.github.io/LIBERO/html/algo_data/datasets.html#datasets).

MetaWorld: We have provided the script we used to collect the data using scripted policies in the MetaWorld package. Please run the following command to collect the data. This uses configs as per [collect_data.yaml](config/collect_data.yaml).
```
python scripts/generate_metaworld_dataset.py
```
We generate 100 demonstrations for each of 45 pretraining tasks and 5 for downstream tasks.

## Training
First set the path to the dataset `data_prefix` and `output_prefix` in [train_base](config/train_base.yaml). `output_prefix` is where all the logs and checkpoints will be stored.

We provide detailed sample commands for training all stages and for all baselines in the [scripts](scripts) directory. For all methods, [autoencoder.sh](scripts/hiqst/autoencoder.sh) trains the autoencoder (only used in hiqst and VQ-BeT), [main.sh](scripts/hiqst/main.sh) trains the main algorithm (skill-prior incase of hiqst), and [finetune.sh](scripts/hiqst/finetune.sh) finetunes the model on downstream tasks.

Run the following command to train hiqst's stage-0 i.e. the autoencoder. (ref: [autoencoder.sh](scripts/hiqst/autoencoder.sh))
```
python train.py --config-name=train_autoencoder.yaml \
    task=libero_90 \
    algo=hiqst \
    exp_name=final \
    variant_name=block_32_ds_4 \
    algo.skill_block_size=32 \
    algo.downsample_factor=4 \
    seed=0
```
The above command trains the autoencoder on the libero-90 dataset with a block size of 32 and a downsample factor of 4. The run directory will be created at `<output_prefix>/<benchmark_name>/<task>/<algo>/<exp_name>/<variant_name>/<seed>/<stage>`. For above command, it will be `./experiments/libero/libero_90/hiqst/final/block_32_ds_4/0/stage_0`.

Run the following command to train hiqst's stage-1 i.e. the skill-prior. (ref: [main.sh](scripts/hiqst/main.sh))
```
python train.py --config-name=train_prior.yaml \
    task=libero_90 \
    algo=hiqst \
    exp_name=final \
    variant_name=block_32_ds_4 \
    algo.skill_block_size=32 \
    algo.downsample_factor=4 \
    training.auto_continue=true \
    seed=0
```
Here, training.auto_continue will automatically load the latest checkpoint from the previous training stage.

Run the following command to finetune hiqst on a downstream tasks. (ref: [finetune.sh](scripts/hiqst/finetune.sh))
```
python train.py --config-name=train_fewshot.yaml \
    task=libero_long_fewshot \
    algo=hiqst \
    exp_name=final \
    variant_name=block_32_ds_4 \
    algo.skill_block_size=32 \
    algo.downsample_factor=4 \
    algo.l1_loss_scale=10 \
    training.auto_continue=true \
    seed=0
```
Here, algo.l1_loss_scale is used to finetune the decoder of the autoencoder while finetuning. 

For metaworld fewshot, set task=metaworld_ml45_prise_fewshot and algo.l1_loss_scale=0.

## Evaluating
Run the following command to evaluate the trained model. (ref: [eval.sh](scripts/eval.sh))
```
python evaluate.py \
    task=libero_90 \
    algo=hiqst \
    exp_name=final \
    variant_name=block_32_ds_4 \
    stage=1 \
    training.use_tqdm=false \
    seed=0
```
This will automatically load the latest checkpoint as per your exp_name, variant_name, algo, and stage. Else you can specify the checkpoint_path to load a specific checkpoint.

## Video
Libero-long

https://github.com/DJ9898123/video/blob/main/0.mp4


