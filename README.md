# Semantic Instance Geometry Network for Unsupervised Perception

> Project page: [https://mengyuest.github.io/SIGNet/](https://mengyuest.github.io/SIGNet/)

> Original implementation trained on [KITTI](http://www.cvlibs.net/datasets/kitti/index.php): https://github.com/mengyuest/SIGNet

> Original paper: 
Y. Meng, Y. Lu, A. Raj, S. Sunarjo, R. Guo, T. Javidi, G. Bansal, D. Bharadia. **"SIGNet: Semantic Instance Aided Unsupervised 3D Geometry Perception"**,  (CVPR), 2019. \[[arXiv pdf](https://arxiv.org/pdf/1812.05642.pdf)\] 

This repository recreates SIGNet for the CARLA Simulator. It also contains detailed instructions about how to train SIGNet on new datasets.

## Prerequisites
1. Ubuntu 16.04, python3, tensorflow-gpu 1.10.1
2. Better to use virtual environment. For the rest of dependencies, please run `pip3 install -r requirements.txt`

## Training & Evaluating Depth on a Different Dataset

Training on a new dataset can be an involved process considering that everything in the original implementation is very specific to KITTI.

### Data Structuring
There are certain key directories in the config files that will need to be changed based on your dataset. Namely, `DATASET_DIR`, `FILELIST_DIR`, `INS_TRAIN_KITTI_DIR`, `INS_TEST_KITTI_DIR`, and `MASK_KITTI_DIR` will need to be updated.

A sample file hierarchy is shown below to aid the discussion.

Data is read using TensorFlow file readers and queues. For convenience, all sequences of (source, target, source) images belonging to a common video should be inside a common folder, such as `video1` below. Each sequence of images should have images which are concatenated horizontally in temporal order (e.g. `seq_x.png`). These should preferably be resized to dimension `(416, 1248, 3)` to match what was used for KITTI, as using a different size may result in lower performance.

Extended subdirectories can exist within this folder, and even the filenames can be different than what is given below - the only important thing is that the file paths for all images should be listed in a document like `train.txt`, which will be used to read the data.

Note that semantic segmentation data (`_.npy`, `_.raw` files), instance mask data (`_instance_new.npy`, `_instance_new.raw` files) and camera intrinsics data (`_cam.txt` files) need to be in the same folders as the image (see hierarchy below).

```
|-- train_data
|   |-- video1
|   |   |-- seq_1.png                   # RGB image sequence
|   |   |-- seq_1.npy                   # Semantic mask data
|   |   |-- seq_1.raw
|   |   |-- seq_1_instance_new.npy      # Instance mask data
|   |   |-- seq_1_instance_new.raw
|   |   |-- 1_cam.txt                   # Camera intrinsics data
|   |   |-- seq_2.png
|   |   |-- seq_2.npy
|   |   |-- seq_2.raw
|   |   |-- seq_2_instance_new.npy
|   |   |-- seq_2_instance_new.raw
|   |   |-- 2_cam.txt
|   |    ...
|   |    ...
|   |-- video2
|   |   ...
|   |   ...
|   |-- train.txt
|
|-- test_data
|   |-- video1
|   |   |-- img_1.png
|   |   |-- img_1.npy
|   |   |-- img_1.raw
|   |   |-- img_1_instance_new.npy
|   |   |-- img_1_instance_new.raw
|   |   |-- img_2.png
|   |   |-- img_2.npy
|   |   |-- img_2.raw
|   |   |-- img_2_instance_new.npy
|   |   |-- img_2_instance_new.raw
|   |    ...
|   |    ...
|   |-- video2
|   |    ...
|   |    ...
|   |-- test_eigen_file.txt
|
```

It is encouraged to go over how exactly these files are read into TF queues so that you can create/modify the functions as per your use case. This information can be found in `sig_main.py` and `data_loader.py`.

`bash run_depth_train.sh config/foobar.cfg` can be used to train depth.

### Generating Semantic Data
The SIGNet paper uses DeepLabv3+ to generate semantic maps. The `_.npy` files are simply the semantic segmentation image maps saved as `npy` files. The extension is updated to `.raw` for faster data loading.

### Generating Instance Masks
The SIGNet paper uses Mask-RCNN trained using FAIR's [Detectron]{https://github.com/facebookresearch/Detectron}. The Detectron code can be found in ________.
The `_instance_new.npy` files are generated using the script ______. Once again, the extension is updated to `.raw` for faster data loading.

### Generating Camera Intrinsics
The `_cam.txt` files contain 9 comma separated values representing the camera calibration matrix, in index order `(0,0), (0,1), (0,2), (1,0), (1,1), (1,2), (2,0), (2,1), (2,2)`.

### Evaluation

There are a few changes during evaluation. The image files which contained sequences earlier will now contain single images since we only need single image input for depth prediction.

You will also have to modify the ground truth depth generation based on your dataset. SIGNet for KITTI uses velodyne points to generate a sparse depth map for each image (details in paper). This may not be the case in your scenario.

In the end, ground truth depth files are stored as `models/gt_data/gt_depth.npy`. This `npy` file would contain a list with the depth maps of all test files in `test_eigen_file.txt`, in the same order.

`bash run_depth_test_eval.sh config/foobar.cfg` can be used to evaluate depth.
