# Unet: Practice
```angular2html
This repository has come from the following repository.
More utils and its applications have been added
# tested on py36-pytor1120-tf1150-gpu.yml
- python 3.6
- cudatoolkit 10.1
- pytorch 1.5.1
- tensorflow-gpu 1.14.0
- tensorboard 1.14.0
- future 0.18.2
- combine anaconda (pytorch) pip (tensorflow)
# updates:
dataset.py
- mask image is converted to gray image with 'Image.open().convert('L') 
- 1channel->3channel inputs -> back into original state 
         if len(img_nd.shape) == 2: # 1channel -> 3 channels by sangkny
            # img_nd = np.expand_dims(img_nd, axis=2)
            img_nd = np.repeat(img_nd[:, :, np.newaxis], 3, axis=2)

train.py 
- loss.backward()에서 죽어서.. cuda -> cpu 로 바꾸어 보니... 제대로 돌아가고 있음(cuda function 문제, 버전등), 너무 오래 걸림.
- 보통은 python 2.x 와 python 3. 일때.. data[0] /data의 사용을 잘 못 하여 문제가 되는 데... 이때는 loss.item() 또는 loss.data를 사용하면 됨.
- 최종적으로 cudaToolkit 9.2 -> 10.1 그리고 이에 맞게 pyTorch 1.2.0 -> 1.5.0, tensorflow(tensorboard) 1.10.0 -> 1.14.0 으로 수정후 정상적으로 돌아감
- 본 repository의 특징은 Summary 를 이용해서 나름 그래픽컬하게 진행상황을 보여주는 것이어서 tensorboar 1.14.0에 맞게 구성을 해야 함.

```
# UNet: semantic segmentation with PyTorch

![input and output for a random image in the test dataset](https://framapic.org/OcE8HlU6me61/KNTt8GFQzxDR.png)


Customized implementation of the [U-Net](https://arxiv.org/abs/1505.04597) in PyTorch for Kaggle's [Carvana Image Masking Challenge](https://www.kaggle.com/c/carvana-image-masking-challenge) from high definition images.

This model was trained from scratch with 5000 images (no data augmentation) and scored a [dice coefficient](https://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient) of 0.988423 (511 out of 735) on over 100k test images. This score could be improved with more training, data augmentation, fine tuning, playing with CRF post-processing, and applying more weights on the edges of the masks.

The Carvana data is available on the [Kaggle website](https://www.kaggle.com/c/carvana-image-masking-challenge/data).

## Usage
**Note : Use Python 3**
### Prediction

You can easily test the output masks on your images via the CLI.

To predict a single image and save it:

`python predict.py -i image.jpg -o output.jpg`

To predict a multiple images and show them without saving them:

`python predict.py -i image1.jpg image2.jpg --viz --no-save`

```shell script
> python predict.py -h
usage: predict.py [-h] [--model FILE] --input INPUT [INPUT ...]
                  [--output INPUT [INPUT ...]] [--viz] [--no-save]
                  [--mask-threshold MASK_THRESHOLD] [--scale SCALE]

Predict masks from input images

optional arguments:
  -h, --help            show this help message and exit
  --model FILE, -m FILE
                        Specify the file in which the model is stored
                        (default: MODEL.pth)
  --input INPUT [INPUT ...], -i INPUT [INPUT ...]
                        filenames of input images (default: None)
  --output INPUT [INPUT ...], -o INPUT [INPUT ...]
                        Filenames of ouput images (default: None)
  --viz, -v             Visualize the images as they are processed (default:
                        False)
  --no-save, -n         Do not save the output masks (default: False)
  --mask-threshold MASK_THRESHOLD, -t MASK_THRESHOLD
                        Minimum probability value to consider a mask pixel
                        white (default: 0.5)
  --scale SCALE, -s SCALE
                        Scale factor for the input images (default: 0.5)
```
You can specify which model file to use with `--model MODEL.pth`.

### Training

```shell script
> python train.py -h
usage: train.py [-h] [-e E] [-b [B]] [-l [LR]] [-f LOAD] [-s SCALE] [-v VAL]

Train the UNet on images and target masks

optional arguments:
  -h, --help            show this help message and exit
  -e E, --epochs E      Number of epochs (default: 5)
  -b [B], --batch-size [B]
                        Batch size (default: 1)
  -l [LR], --learning-rate [LR]
                        Learning rate (default: 0.1)
  -f LOAD, --load LOAD  Load model from a .pth file (default: False)
  -s SCALE, --scale SCALE
                        Downscaling factor of the images (default: 0.5)
  -v VAL, --validation VAL
                        Percent of the data that is used as validation (0-100)
                        (default: 15.0)

```
By default, the `scale` is 0.5, so if you wish to obtain better results (but use more memory), set it to 1.

The input images and target masks should be in the `data/imgs` and `data/masks` folders respectively.

## Tensorboard
You can visualize in real time the train and test losses, the weights and gradients, along with the model predictions with tensorboard:

`tensorboard --logdir=runs`

You can find a reference training run with the Caravana dataset on [TensorBoard.dev](https://tensorboard.dev/experiment/1m1Ql50MSJixCbG1m9EcDQ/#scalars&_smoothingWeight=0.6) (only scalars are shown currently).

## Notes on memory

The model has be trained from scratch on a GTX970M 3GB.
Predicting images of 1918*1280 takes 1.5GB of memory.
Training takes much approximately 3GB, so if you are a few MB shy of memory, consider turning off all graphical displays.
This assumes you use bilinear up-sampling, and not transposed convolution in the model.

---

Original paper by Olaf Ronneberger, Philipp Fischer, Thomas Brox: [https://arxiv.org/abs/1505.04597](https://arxiv.org/abs/1505.04597)

![network architecture](https://i.imgur.com/jeDVpqF.png)
