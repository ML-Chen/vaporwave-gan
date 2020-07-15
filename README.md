# Vaporwave GAN

Code snippets for a [StyleGAN2](https://github.com/NVlabs/stylegan2)-based generative adversarial network to generate vaporwave art, by Michael Chen, Swetha Rajagopalan, James Thomas, Haoyong Xue, and Julia Zhu. A project for CS 4641: Machine Learning, Summer 2020, with Xin Chen.

## Getting images

See `Image processing.md`.

## Training

Clone the stylegan2 repo in here: `git clone https://github.com/NVlabs/stylegan2`

Create a virtual environment. On Windows Command Prompt, this would be:

```cmd
virtualenv venv
.\venv\Scripts\activate
```

Then, install Tensorflow GPU 1.15 (Unix) or 1.14 (Windows):

```
pip install tensorflow-gpu==1.15
# or
pip install tensorflow-gpu==1.14
```

Install other dependencies (listed in the Dockerfile):

```
pip install scipy==1.3.3
pip install requests==2.22.0
pip install Pillow==6.2.1
```

Start training:

--dataset is the directory of the tfrecords  
--data-dir is the parent directory of that
--total-kimg: it will stop training at this many thousands of images

```
python run_training.py --num-gpus=1 --data-dir="D:\Dropbox (GaTech)\vaporwave\256x256png" --config=config-f --dataset=vw256 --total-kimg=1000
python run_generator.py generate-images --seeds=0-10 --truncation-psi=1.0 --network=results\00000-stylegan2-ds-1gpu-config-f\network-snapshot-000120.pkl
```

In `stylegan2/training/training_loop.py`, you can assign a path to `resume_pkl` to specify a pkl file to resume training from. The default is `None`, if you don't have a pkl file to resume training from. The resolution of the images have to match what it was previously trained on, but it shouldn't matter whether it's a PNG or JPG. `resume_kimg` allows you to say how many thousands of images have already been trained on. This parameter doesn't really affect anything.

As it is training, StyleGAN will save pkl files periodically, under where it says "# Save snapshots".
