# RDAM: Region-Aware Diffusion for Adversarial Makeup-Based Facial Privacy Protection

## After the paper is accepted, we will further improve the code in detail

## Abstract

Facial privacy protection has become increasingly important with the widespread deployment of face recognition (FR) systems.However, existing adversarial makeup methods struggle to disentangle makeup-related attributes from identity-related features and preserve fine-grained facial details during diffusion-based generation. To address these challenges, we propose RDAM, a region-aware diffusion-based adversarial makeup framework for proactive facial privacy protection. RDAM adopts a two-stage design. First, we develop an implicit makeup difference extraction strategy that disentangles makeup-related variations from intrinsic facial features in the latent space through latent inversion and reverse guidance. Second, we generate adversarial makeup faces through a region-aware diffusion process. Specifically, we design a facial feature-oriented attention mechanism and incorporate it into large-kernel convolution and multi-scale facial feature fusion to enhance the representation of global facial structures and fine-grained local details. In addition, a joint optimization objective combines ensemble adversarial guidance with a facial-aware guidance loss to improve transferability to unseen black-box FR models while maintaining visual naturalness. Extensive experiments on CelebA-HQ and LADN demonstrate that RDAM achieves superior black-box attack transferability and image fidelity compared with existing state-of-the-art makeup-based attack methods.

## Setup

- ### Build environment

```shell
cd RDAM
# use anaconda to build environment 
conda create -n RDAM python=3.8
conda activate RDAM
# install packages
pip install -r requirements.txt
pip install git+https://github.com/openai/CLIP.git
```

## Pretrained models and datasets



```shell
mkdir pretrained
mv celeba_hq.ckpt pretrained/
mv makeup.pt pretrained/
mv model_ir_se50.pth pretrained/
mv shape_predictor_68_face_landmarks.dat pretrained/
```

- Please download the target FR models, MT-datasets and target images [here](https://drive.google.com/file/d/1IKiWLv99eUbv3llpj-dOegF3O7FWW29J/view?usp=sharing). Unzip the assets.zip file in `RDAM/assets`.
- Please download the [CelebAMask-HQ](https://drive.google.com/file/d/1badu11NqxGf6qM3PTTooQDJvQbejgbTv/view) dataset and unzip the file in `RDAM/assets/datasets`.

The final project should be like this:

```shell
RDAM
  └- assets
     └- datasets
     	└- CelebAMask-HQ
     	└- MT-dataset
     	└- pairs
     	└- target
     	└- test
     └- models
  └- pretrained
       └- celeba_hq.ckpt
       └- ...
  └- ...
```

## Quick Start

### Makeup removal (Optional)


```shell
python main.py --makeup_removal --config MT.yml --exp ./runs/test --do_train 1 --do_test 1 --n_train_img 200 --n_test_img 100 --n_iter 7 --t_0 300 --n_inv_step 40 --n_train_step 6 --n_test_step 40 --lr_clip_finetune 8e-6 --model_path pretrained/makeup.pt
```

### Adversarial makeup transfer


```shell
python main.py --makeup_transfer --config celeba.yml --exp ./runs/test --do_train 1 --do_test 1 --n_train_img 200 --n_test_img 100 --n_iter 4 --t_0 60 --n_inv_step 20 --n_train_step 6 --n_test_step 6 --lr_clip_finetune 8e-6 --model_path pretrained/celeba_hq.ckpt --target_img 1 --target_model 2 --ref_img 'XMY-060'
```

- `target_img`: Choose the target identity to attack, a total of 4 options are provided (see details in our supplementary materials).

- `target_model`: Choose the target FR model to attack, including `[IRSE50, IR152, Mobileface, Facenet]`.
- `ref_img`: Choose the provided makeup style to transfer, including `['XMY-060', 'XYH-045', 'XMY-254', 'vRX912', 'vFG137']`. In addition, by generating pairs of makeup and non-makeup images through makeup removal, you can also transfer the makeup style you want. (Save `{ref_name}_m.png`, `{ref_name}_nm.png`, and `{ref_name}_mask.png` to `RDAM/assets/datasets/pairs`.)

### Edit one image

You can edit one image for makeup removal and transfer by running the following command:

```shell
# makeup removal
python main.py --edit_one_image_MR --config MT.yml --exp ./runs/test --n_iter 1 --t_0 300 --n_inv_step 40 --n_train_step 6 --n_test_step 40 --img_path {IMG_PATH} --model_path {MODEL_PATH}

# adversarial makeup removal
python main.py --edit_one_image_MT --config celeba.yml --exp ./runs/test --n_iter 1 --t_0 60 --n_inv_step 20 --n_train_step 6 --n_test_step 6 --img_path {IMG_PATH} --model_path {MODEL_PATH}
```

- `img_path`: Path of an image to edit.
- `model_path`: Path of fine-tuned model.
