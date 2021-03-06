# 2D to 3D Reconstruction

Take out your phone, take a video, and get a 3D model!

## Installation

This project runs on Python 3.5+. We recommend doing the following installations inside a [Python virtual environment](https://docs.python.org/3/tutorial/venv.html) or [Conda environment](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html). Use [pip](https://pip.pypa.io/en/stable/) to install the dependencies for this project:

```bash
pip install -r requirements.txt
```
This project requires a machine learning model to run the depth predication. This can be either generated via the instructions
found in the [DenseDepth Github](https://github.com/ialhashim/DenseDepth). Alternately you can download one of the pertained
models from the DenseDepth repository.

## Files and Pipelines

Here we give a brief overview of the files in this repository and what each file’s function is.

- `dense_depth/`: Folder containing clone of [DenseDepth repository](https://github.com/ialhashim/DenseDepth). It is here so that we can use the DenseDepth model to predict depth maps.
- `examples/`: Folder containing several image sets for running our pipeline on
- `models/`: Folder to put models in
- `utils/homography_utils/`: RANSAC and feature matching algorithms
- `utils/open3d_fpfh.py`: Functions from Open3D documentation that we use in our pipeline
- `utils/`: Our utils folder. This includes a file for every method in the pipeline that we use: ICP, rigid 3D, 3D homography, and some other miscellaneous utils.
- `reconstruct.py` and `reconstruct_rgbd.py`: These are the two main pipelines in our model.
- `requirements.txt`: List of Python packages needed to run this project

We will go through `reconstruct.py` and `reconstruct_rgbd.py` in more detail below.

#### Pre-trained Models

To run `reconstruct.py`, we need the DenseDepth model pretrained on NYU Depth v2. We have linked the models here.
* [NYU Depth V2](https://s3-eu-west-1.amazonaws.com/densedepth/nyu.h5) (165 MB)
* [KITTI](https://s3-eu-west-1.amazonaws.com/densedepth/kitti.h5) (165 MB)

## Usage

There are 2 main pipelines set up for our project. The difference between the two is that `reconstruct.py` uses machine learning to generate the initial depth maps, whereas `reconstruct_rgbd.py` takes in depth images. Since some of the algorithms we worked with did not work too well on mediocre depth, we have the second option of taking good depth.

The steps to the pipelines are as follows:

1. `reconstruct.py` will use [DenseDepth repository](https://github.com/ialhashim/DenseDepth) to generate depth maps for every RGB frame of the video given. `reconstruct_rgbd.py` will skip this step as we have already provided it with corresponding depth frames (hence RGBD).
 
2. Create a point cloud for all of the images using the depth maps. 

3. Apply one of the point cloud matching algorithms to merge all the generated point clouds. There are several options here:

  - __FPFH + RANSAC__: Matches FPFH  descriptors using a RANSAC based 3D transformation estimation

  - __FPFH + FAST__: Matches FPFH  descriptors using the Fast global registration algorithm 

  - __3D Rigid Body Transformation__: Matches two sets of 3D points by finding the optimal rigid body transform transformation using least squares. The 3D points’ initial correspondences are computed using SIFT and RANSAC.

  - __3D Homography and ICP__: Matches two sets of 3D points by finding the optimal 3D homography matrix using constrained least squares. The 3D points’ initial correspondences are computed using SIFT and RANSAC. This global registration is then further refined by running ICP with partial matching on the two point clouds.

4. Generate a mesh for the merged point clouds using either the Poisson or the ball point surface reconstruction algorithms.

The final result of this pipeline is a dense 3D mesh!

We have listed command line usage for both pipelines below.

### Note
for both pipelines all the images need to be number in an increasing order starting at 1 

### Reconstruct

```bash
usage: reconstruct.py [-h] [--model MODEL] [--rgb RGB]
                      [--mode {fpfh,rigid3d,3dhomo}] [--voxel VOXEL] [--fast]
                      [--surface {poisson,ball_point}] [--save_intermediate]
                      [--out_folder OUT_FOLDER] [--out_name OUT_NAME] [--plot]

Depth Generation, Point Cloud Registration, and 3D Model Reconstruction

optional arguments:
  -h, --help            show this help message and exit
  --model MODEL         Trained Keras model file. Requires TensorFlow and
                        Keras.
  --rgb RGB             Input filename or folder for RGB images
  --mode {fpfh,rigid3d,3dhomo}
                        Global registration method
  --voxel VOXEL         Size of voxel to downsample for FPFH. Do not use if
                        not using FPFH for mode option.
  --fast                Enable to use fast global registration for FPFH. Do
                        not use if not using FPFH for mode option
  --surface {poisson,ball_point}
                        Method of generating surface mesh
  --save_intermediate   Enable to store intermediate results (in out_folder)
  --out_folder OUT_FOLDER
                        Path to folder to save generated point clouds and
                        meshes in
  --out_name OUT_NAME   Name of image set to save as
  --plot                Enable to plot intermediate results in pipeline

Note: input folder must be the format 
./examples/sofa2/*.jpg

```

### Reconstruct RGBD

```bash
usage: reconstruct_rgbd.py [-h] [--rgb RGB] [--depth DEPTH] [--inter INTER]
                           [--mode {fpfh,rigid3d,3dhomo}] [--voxel VOXEL]
                           [--fast] [--surface {poisson,ball_point}]
                           [--save_intermediate] [--out_folder OUT_FOLDER]
                           [--out_name OUT_NAME] [--plot]

Depth Point Cloud Registration and 3D Model Reconstruction

optional arguments:
  -h, --help            show this help message and exit
  --rgb RGB             Input filename or folder for RGB images
  --depth DEPTH         Input filename or folder for depth images
  --inter INTER         Read point clouds
  --mode {fpfh,rigid3d,3dhomo}
                        Global registration method
  --voxel VOXEL         Size of voxel to downsample for FPFH. Do not use if
                        not using FPFH for mode option.
  --fast                Enable to use fast global registration for FPFH. Do
                        not use if not using FPFH for mode option
  --surface {poisson,ball_point}
                        Method of generating surface mesh
  --save_intermediate   Enable to store intermediate results (in out_folder)
  --out_folder OUT_FOLDER
                        Path to folder to save generated point clouds and
                        meshes in
  --out_name OUT_NAME   Name of image set to save as
  --plot                Enable to plot intermediate results in pipeline

```

## Examples

Below we have a bunch of command line arguments for using our pipeline to generate meshes, 
using the [NYU Depth V2](https://s3-eu-west-1.amazonaws.com/densedepth/nyu.h5) (165 MB) model. 
Place The nyu.h5 file into the `models/` folder. Then just start up your virtual environment (or wherever you installed your dependencies), cd into this project's repo, and run these in your terminal!

The results that we generated running these examples can be found [here](https://drive.google.com/drive/folders/1907IghbC04UDFmIyf7IN3_m3hfr55C8D?usp=sharing).

### Reconstruct

#### Front-Side of a car - [Expected Results](https://drive.google.com/drive/folders/16GOdR8XLdmlZUz_nOIaPe3V-olYNschd?usp=sharing)

```
python reconstruct.py --rgb ./examples/car/*.jpg --mode rigid3d --surface poisson --save_intermediate --out_folder ./examples/car/rigid3d --out_name car_poisson
```

```
python reconstruct.py --rgb ./examples/car/*.jpg --mode 3dhomo --surface poisson --save_intermediate --out_folder ./examples/car/homography --out_name car_poisson
```

```
python reconstruct.py --rgb ./examples/car/*.jpg --mode fpfh --voxel 5 --fast --surface poisson --save_intermediate --out_folder ./examples/car/fpfh_fast --out_name car_poisson
```

```
python reconstruct.py --rgb ./examples/car/*.jpg --mode fpfh --voxel 5 --surface poisson --save_intermediate --out_folder ./examples/car/fpfh_ransac --out_name car_poisson
```

#### Helmet - [Expected Results](https://drive.google.com/drive/folders/1-1sxXc9Dr4v-WDGaYHU6BEEBEbjXdy3-?usp=sharing)

```
python reconstruct.py --rgb ./examples/helmet/*.jpg --mode fpfh --voxel 5 --fast --surface poisson --save_intermediate --out_folder ./examples/helmet/fpfh_fast --out_name helmet_poisson
```

```
python reconstruct.py --rgb ./examples/helmet/*.jpg --mode fpfh --voxel 5 --surface poisson --save_intermediate --out_folder ./examples/helmet/fpfh_ransac --out_name helmet_poisson
```

### Reconstruct RGBD

#### kitchen - [Expected Results](https://drive.google.com/drive/folders/17qIa8pR72RaxYaetcps2Wue85QA5NI69?usp=sharing) 

```
python reconstruct_rgbd.py --rgb ./examples/kitchen_rgbd/imgs/*.png --depth ./examples/kitchen_rgbd/depth/*.png --inter true --mode rigid3d --save_intermediate --surface poisson --out_folder ./examples/kitchen_rgbd/rigid3d --out_name kitchen_rgbd_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/kitchen_rgbd/imgs/*.png --depth ./examples/kitchen_rgbd/depth/*.png --inter true --mode 3dhomo --save_intermediate --surface poisson --out_folder ./examples/kitchen_rgbd/homography --out_name kitchen_rgbd_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/kitchen_rgbd/imgs/*.png --depth ./examples/kitchen_rgbd/depth/*.png --inter true --mode fpfh --fast --voxel 20 --save_intermediate --surface poisson --out_folder ./examples/kitchen_rgbd/fpfh_fast --out_name kitchen_rgbd_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/kitchen_rgbd/imgs/*.png --depth ./examples/kitchen_rgbd/depth/*.png --inter true --mode fpfh --voxel 20 --save_intermediate --surface poisson --out_folder ./examples/kitchen_rgbd/fpfh_ransac --out_name kitchen_rgbd_poisson
```

#### Front-Side of a car - [Expected Results](https://drive.google.com/drive/folders/1YIQKlZqd-b1-qcg6YnWELlpVtZjb3H5j?usp=sharing)

```
python reconstruct_rgbd.py --rgb ./examples/car_rgbd/*.jpg --depth ./examples/car_rgbd/*.png --inter true --mode 3dhomo --save_intermediate --surface poisson --out_folder ./examples/car_rgbd/homography --out_name car_rgb_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/car_rgbd/*.jpg --depth ./examples/car_rgbd/*.png --inter true --mode fpfh --fast --voxel 10 --save_intermediate --surface poisson --out_folder ./examples/car_rgbd/fpfh_fast --out_name car_rgb_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/car_rgbd/*.jpg --depth ./examples/car_rgbd/*.png --inter true --mode fpfh --voxel 20 --save_intermediate --surface poisson --out_folder ./examples/car_rgbd/fpfh_ransac --out_name car_rgb_poisson
```

### Car bad matches - [Expected Results](https://drive.google.com/drive/folders/1DL-arCFvX3leTRjIgpitcVtzQ0gjEfgR?usp=sharing)
images of the car from frames that are far part from each other.

```
python reconstruct_rgbd.py --rgb ./examples/car_bad/*.jpg --depth ./examples/car_bad/*.png --inter true --mode 3dhomo --save_intermediate --surface poisson --out_folder ./examples/car_bad/homography --out_name car_rgb_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/car_bad/*.jpg --depth ./examples/car_bad/*.png --inter true --mode fpfh --fast --voxel 10 --save_intermediate --surface poisson --out_folder ./examples/car_bad/fpfh_fast --out_name car_rgb_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/car_bad/*.jpg --depth ./examples/car_bad/*.png --inter true --mode fpfh --voxel 20 --save_intermediate --surface poisson --out_folder ./examples/car_bad/fpfh_ransac --out_name car_rgb_poisson
```

### Front of a sofa - [Expected Results](https://drive.google.com/drive/folders/1nnzx70dAVy5FQu18beuImUsM8c4oBTR2?usp=sharing)

```
python reconstruct_rgbd.py --rgb ./examples/sofa_rgbd/*.jpg --depth ./examples/sofa_rgbd/*.png --inter true --mode fpfh --fast --voxel 10 --save_intermediate --surface poisson --out_folder ./examples/sofa_rgbd/fpfh_fast --out_name sofa_rgbd_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/sofa_rgbd/*.jpg --depth ./examples/sofa_rgbd/*.png --inter true --mode fpfh --voxel 20 --save_intermediate --surface poisson --out_folder ./examples/sofa_rgbd/fpfh_ransac --out_name sofa_rgbd_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/sofa_rgbd/*.jpg --depth ./examples/sofa_rgbd/*.png --inter true --mode fpfh --voxel 20 --save_intermediate --surface ball_point --out_folder ./examples/sofa_rgbd/fpfh_ransac --out_name sofa_rgbd_ball
```

### sofa bad matches - [Expected Results](https://drive.google.com/drive/folders/1s4D1enMqpJH8Vp1tGG72YhJs8n-5Pd8Q?usp=sharing)
images of the sofa from frames that are far part from each other.

```
python reconstruct_rgbd.py --rgb ./examples/sofa_rgbd/*.jpg --depth ./examples/sofa_rgbd/*.png --inter true --mode fpfh --fast --voxel 10 --save_intermediate --surface poisson --out_folder ./examples/sofa_bad/fpfh_fast --out_name sofa_rgbd_poisson
```

```
python reconstruct_rgbd.py --rgb ./examples/sofa_rgbd/*.jpg --depth ./examples/sofa_rgbd/*.png --inter true --mode fpfh --voxel 20 --save_intermediate --surface poisson --out_folder ./examples/sofa_bad/fpfh_ransac --out_name sofa_rgbd_poisson
```
