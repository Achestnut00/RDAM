# DiffGLAM: Adversarial Makeup via Diffusion with Efficient Global-Local Attention for Facial Privacy Protection

## After the paper is accepted, we will further improve the code in detail

## Abstract

The widespread deployment of facial recognition systems has raised significant privacy concerns, necessitating effective countermeasures. Adversarial makeup techniques have emerged as a promising approach, yet they often suffer from limited facial detail perception and poor transferability to black-box models. We propose DiffGLAM, a diffusion-based adversarial makeup framework incorporating efficient global-local attention (EGLA) for enhanced facial privacy protection. DiffGLAM introduces an E-LarK feature extraction module and an efficient multi-scale facial fusion (EMSFF) structure to improve makeup guidance and transferability. We introduce a face region-guided loss using EGLA for precise adversarial makeup localization, producing high-quality and semantically consistent images. 

## Setup

- ### Build environment

```shell
cd DiffGLAM
# use anaconda to build environment 
conda create -n diffglam python=3.8
conda activate diffglam
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

- Please download the target FR models, MT-datasets and target images [here](https://drive.google.com/file/d/1IKiWLv99eUbv3llpj-dOegF3O7FWW29J/view?usp=sharing). Unzip the assets.zip file in `DiffGLAM/assets`.
- Please download the [CelebAMask-HQ](https://drive.google.com/file/d/1badu11NqxGf6qM3PTTooQDJvQbejgbTv/view) dataset and unzip the file in `DiffGLAM/assets/datasets`.

The final project should be like this:

```shell
DiffGLAM
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
- `ref_img`: Choose the provided makeup style to transfer, including `['XMY-060', 'XYH-045', 'XMY-254', 'vRX912', 'vFG137']`. In addition, by generating pairs of makeup and non-makeup images through makeup removal, you can also transfer the makeup style you want. (Save `{ref_name}_m.png`, `{ref_name}_nm.png`, and `{ref_name}_mask.png` to `DiffGLAM/assets/datasets/pairs`.)

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
