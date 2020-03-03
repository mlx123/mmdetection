## Installation

### Requirements

- Linux (Windows is not officially supported)
- Python 3.5+
- PyTorch 1.1 or higher
- CUDA 9.0 or higher
- NCCL 2
- GCC 4.9 or higher
- [mmcv](https://github.com/open-mmlab/mmcv)

We have tested the following versions of OS and softwares:

- OS: Ubuntu 16.04/18.04 and CentOS 7.2
- CUDA: 9.0/9.2/10.0/10.1
- NCCL: 2.1.15/2.2.13/2.3.7/2.4.2
- GCC(G++): 4.9/5.3/5.4/7.3

### Install mmdetection

a. Create a conda virtual environment and activate it.

```shell
conda create -n open-mmlab python=3.7 -y
conda activate open-mmlab
```

b. Install PyTorch and torchvision following the [official instructions](https://pytorch.org/), e.g.,

```shell
conda install pytorch torchvision -c pytorch
```

c. Clone the mmdetection repository.

```shell
git clone https://github.com/open-mmlab/mmdetection.git
cd mmdetection
```

d. Install build requirements and then install mmdetection.
(We install pycocotools via the github repo instead of pypi because the pypi version is old and not compatible with the latest numpy.)

```shell
pip install -r requirements/build.txt
pip install "git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI"
pip install -v -e .  # or "python setup.py develop"
```

Note:

1. The git commit id will be written to the version number with step d, e.g. 0.6.0+2e7045c. The version will also be saved in trained models.
It is recommended that you run step d each time you pull some updates from github. If C++/CUDA codes are modified, then this step is compulsory.

2. Following the above instructions, mmdetection is installed on `dev` mode, any local modifications made to the code will take effect without the need to reinstall it (unless you submit some commits and want to update the version number).

3. If you would like to use `opencv-python-headless` instead of `opencv-python`,
you can install it before installing MMCV.

4. Some dependencies are optional. Simply running `pip install -v -e .` will only install the minimum runtime requirements. To use optional dependencies like `albumentations` and `imagecorruptions` either install them manually with `pip install -r requirements/optional.txt` or specify desired extras when calling `pip` (e.g. `pip install -v -e .[optional]`). Valid keys for the extras field are: `all`, `tests`, `build`, and `optional`.

### Another option: Docker Image

We provide a [Dockerfile](https://github.com/open-mmlab/mmdetection/blob/master/docker/Dockerfile) to build an image.

```shell
# build an image with PyTorch 1.1, CUDA 10.0 and CUDNN 7.5
docker build -t mmdetection docker/
```

### Prepare datasets

It is recommended to symlink the dataset root to `$MMDETECTION/data`.
If your folder structure is different, you may need to change the corresponding paths in config files.

```
mmdetection
├── mmdet
├── tools
├── configs
├── data
│   ├── coco
│   │   ├── annotations
│   │   ├── train2017
│   │   ├── val2017
│   │   ├── test2017
│   ├── cityscapes
│   │   ├── annotations
│   │   ├── leftImg8bit
│   │   │   ├── train
│   │   │   ├── val
│   │   ├── gtFine
│   │   │   ├── train
│   │   │   ├── val
│   ├── VOCdevkit
│   │   ├── VOC2007
│   │   ├── VOC2012

```
The cityscapes annotations have to be converted into the coco format using `tools/convert_datasets/cityscapes.py`:
```shell
pip install cityscapesscripts
python tools/convert_datasets/cityscapes.py ./data/cityscapes --nproc 8 --out_dir ./data/cityscapes/annotations
```
Current the config files in `cityscapes` use COCO pre-trained weights to initialize.
You could download the pre-trained models in advance if network is unavailable or slow, otherwise it would cause errors at the beginning of training.

### A from-scratch setup script

Here is a full script for setting up mmdetection with conda and link the dataset path (supposing that your COCO dataset path is $COCO_ROOT).

```shell
conda create -n open-mmlab python=3.7 -y
conda activate open-mmlab

conda install -c pytorch pytorch torchvision -y
git clone https://github.com/open-mmlab/mmdetection.git
cd mmdetection
pip install -r requirements/build.txt
pip install "git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI"
pip install -v -e .

mkdir data
ln -s $COCO_ROOT data
```

### Using multiple MMDetection versions

If there are more than one mmdetection on your machine, and you want to use them alternatively, the recommended way is to create multiple conda environments and use different environments for different versions.

Another way is to insert the following code to the main scripts (`train.py`, `test.py` or any other scripts you run)
```python
import os.path as osp
import sys
sys.path.insert(0, osp.join(osp.dirname(osp.abspath(__file__)), '../'))
```

Or run the following command in the terminal of corresponding folder to temporally use the current one.
```shell
export PYTHONPATH=`pwd`:$PYTHONPATH
```

#MLX补充
安装的几个坑：
1.pip install "git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI"出现了无法访问https://github.com/cocodataset/cocoapi.git的情况
方法：下载https://github.com/cocodataset/cocoapi.git后手动安装：
# COCOAPI=/path/to/clone/cocoapi
git clone https://github.com/cocodataset/cocoapi.git $COCOAPI
cd $COCOAPI/PythonAPI
# Install into global site-packages
make install
# Alternatively, if you do not have permissions or prefer
# not to install the COCO API into global site-packages
python setup.py install --user

2.安装cocoapi出现了找不到_mask.c的问题:
安装cpython: pip install cython --user

3.执行pip install -v -e .提示找不到CUDA
在安装pytoch时将conda install pytorch torchvision -c pytorch替换为conda install pytorch torchvision==0.2.2 cuda90 cudatoolkit=9.0 -y
4.安装前注意检查和设置gcc为满足的版本
vim ~/.bashrc
PATH="/mnt/lustre/share/gcc/gcc-5.3.0/bin:$PATH"
export CC="/mnt/lustre/share/gcc/gcc-5.3.0/bin/gcc"
export CXX="/mnt/lustre/share/gcc/gcc-5.3.0/bin/g++"
6.各个库的版本需要注意
addict          2.2.1               
certifi         2019.11.28          
cffi            1.13.2              
cycler          0.10.0              
Cython          0.29.15             
kiwisolver      1.1.0               
matplotlib      3.2.0rc3            
mmcv            0.3.2               
mmdet           1.1.0+51df8a9       /mnt/lustre/menglingxuan/mmdetection/mmdetection
numpy           1.18.1              
olefile         0.46                
opencv-python   4.2.0.32            
Pillow          6.2.2               
pip             20.0.2              
pycocotools     2.0                 
pycparser       2.19                
pyparsing       2.4.6               
python-dateutil 2.8.1               
PyYAML          5.3                 
setuptools      45.2.0.post20200209 
six             1.14.0              
terminaltables  3.1.0               
torch           1.1.0               
torchvision     0.2.2               
wheel           0.34.2  
