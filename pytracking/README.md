# PyTracking

A general python library for visual tracking algorithms. The following trackers are integrated with the toolkit,  

1. **ATOM**: Accurate Tracking by Overlap Maximization, Martin Danelljan and Goutam Bhat and Fahad Shahbaz Khan and Michael Felsberg, CVPR 2019 \[[Paper](https://arxiv.org/pdf/1811.07628.pdf)\]\[[Raw results](https://drive.google.com/drive/folders/1MdJtsgr34iJesAgL7Y_VelP8RvQm_IG_)\]
2. **ECO**: Efficient Convolution Operators for Tracking, Martin Danelljan and Goutam Bhat and Fahad Shahbaz Khan and Michael Felsberg, CVPR 2017 \[[Paper](https://arxiv.org/pdf/1611.09224.pdf)\]

## Table of Contents

* [Running a tracker](#running-a-tracker)
* [Overview](#overview)
* [Trackers](#trackers)
   * [ATOM](#ATOM)
   * [ECO](#ECO)
* [Libs](#libs)
* [Integrating a new tracker](#integrating-a-new-tracker)


## Running a tracker
The installation script will automatically generate a local configuration file  "evaluation/local.py". In case the file was not generated, run ```evaluation.environment.create_default_local_file()``` to generate it. Next, set the paths to the datasets you want
to use for evaluations. You can also change the path to the networks folder, and the path to the results folder, if you do not want to use the default paths. If all the dependencies have been correctly installed, you are set to run the trackers.  

The toolkit provides 3 ways to run a tracker.  

**Run the tracker on webcam feed**   
This is done using the run_webcam script. The arguments are the name of the tracker, and the name of the parameter file.  
```bash
python run_webcam.py tracker_name parameter_name    
```  

**Run the tracker on some dataset sequence**
This is done using the run_tracker script. 
```bash
python run_tracker.py tracker_name parameter_name --dataset_name dataset_name --sequence sequence --debug debug --threads threads
```  

Here, the dataset_name can be either ```'otb'``` (OTB-2015), ```'nfs'``` (Need for Speed), ```'uav'``` (UAV123), ```'tpl'``` (Temple128), ```'tn'``` (TrackingNet test set), ```'gott'``` (GOT-10k test set), 
```'gotv'``` (GOT-10k val set), ```'lasot'``` (LaSOT) or ```'vot'``` (VOT2018). The sequence can either be an integer denoting the index of the sequence in the dataset, or the name of the sequence, e.g. ```'Soccer'```.
The ```debug``` parameter can be used to control the level of debug visualizations. ```threads``` parameter can be used to run on multiple threads.

**Run the tracker on a set of datasets**  
This is done using the run_experiment script. To use this, first you need to create an experiment setting file in ```pytracking/experiments```. See [myexperiments.py](experiments/myexperiments.py) for reference. 
```bash
python run_experiment.py experiment_module experiment_name --dataset_name dataset_name --sequence sequence  --debug debug --threads threads
```  
Here, ```experiment_module```  is the name of the experiment setting file, e.g. ```myexperiments``` , and ``` experiment_name```  is the name of the experiment setting, e.g. ``` atom_nfs_uav``` .

## Overview
The tookit consists of the following sub-modules.  
 -  [evaluation](evaluation): Contains the necessary scripts for running a tracker on a dataset. It also contains integration of a number of standard tracking datasets, namely  [OTB-100](http://cvlab.hanyang.ac.kr/tracker_benchmark/index.html), [NFS](http://ci2cv.net/nfs/index.html),
 [UAV123](https://ivul.kaust.edu.sa/Pages/pub-benchmark-simulator-uav.aspx), [Temple128](http://www.dabi.temple.edu/~hbling/data/TColor-128/TColor-128.html), [TrackingNet](https://tracking-net.org/), [GOT-10k](http://got-10k.aitestunion.com/), [LaSOT](https://cis.temple.edu/lasot/), and [VOT2018](http://www.votchallenge.net/vot2018/).  
 - [experiments](experiments): The experiment setting files must be stored here,  
 - [features](features): Contains functions useful for feature extraction, including various data augmentation methods.  
 - [libs](libs): Includes libraries for optimization, dcf, etc.  
 - [parameter](parameter): Contains the parameter settings for different trackers.  
 - [tracker](tracker): Contains the implementations of different trackers.  
 - [utils](utils): Some uitl functions.  
 
## Trackers
 The toolkit contains the implementation of the following trackers.  
 
 
### [ATOM](tracker/atom)
Official implementation of the [**ATOM**](https://arxiv.org/pdf/1811.07628.pdf) tracker. ATOM is based on (i) a **target estimation** module that is trained offline, and (ii) **target classification** module that is trained online. The target estimation module is trained to predict the intersection-over-union (IoU) overlap between the target and a bounding box estimate. The target classification module is learned online using dedicated optimization techniques to discriminate between the target object and background.
 
![ATOM overview figure](utils/atom_overview.png)
 
##### Parameter Files
Two parameter settings are provided. These can be used to reproduce the results or as a starting point for your exploration.  
* **[default](parameter/atom/default.py)**: The default parameter setting that was used to produce all ATOM results in the paper, except on VOT.  
* **[default_vot](parameter/atom/default_vot.py)**: The parameters settings used to generate the VOT2018 results in the paper.  

The difference between these two stems for the fact that the VOT protocol measures robustness in a very different manner compared to other benchmarks. In most benchmarks, it is highly important to be able to robustly *redetect* the target after e.g. an occlusion or brief target loss. On the other hand, in VOT the tracker is reset if the prediction does not overlap with the target on a *single* frame. This is then counted as a tracking failure. The capability of recovering after target loss is meaningless in this setting. The ```default_vot``` setting thus focuses on avoiding target loss in the first place, while sacrificing re-detection ability. 

##### Raw Results
 The raw results used in the paper are available in this [google drive folder](https://drive.google.com/drive/folders/1MdJtsgr34iJesAgL7Y_VelP8RvQm_IG_).  
 **Note**: Due to the stochastic nature of the tracker, results can vary slightly between different runs. The results reported in the paper are hence an average over 5 runs for NFS, UAV123, OTB-100 and LaSOT datasets, and 15 runs for VOT2018. 
 
### [ECO](tracker/eco)
An unofficial implementation of the [**ECO**](https://arxiv.org/pdf/1611.09224.pdf) tracker. It is implemented based on an extensive and general library for [complex operations](libs/complex.py) and [Fourier tools](libs/fourier.py). The implementation differs from the version used in the original paper in a few important aspects. 
1. This implementation uses features from vgg-m layer 1 and resnet18 residual block 3.   
2. As in our later [UPDT tracker](https://arxiv.org/pdf/1804.06833.pdf), seperate filters are trained for shallow and deep features, and extensive data augmentation is employed in the first frame.  
3. The GMM memory module is not implemented, instead the raw projected samples are stored.  

Please refer to the [official implementation of ECO](https://github.com/martin-danelljan/ECO) if you are looking to reproduce the results in the ECO paper or download the raw results.
 
## Libs
The pytracking repository includes some general libraries for implementing and developing different kinds of visual trackers, including deep learning based, optimization based and correlation filter based. The following libs are included:

* [**Optimization**](libs/optimization.py): Efficient optimizers aimed for online learning, including the Gauss-Newton and Conjugate Gradient based optimizer used in ATOM.
* [**Complex**](libs/complex.py): Complex tensors and operations for PyTorch, which can be used for DCF trackers.
* [**Fourier**](libs/fourier.py): Fourier tools and operations, which can be used for implementing DCF trackers.
* [**DCF**](libs/dcf.py): Some general tools for DCF trackers.
 
## Integrating a new tracker  
 To implement a new tracker, create a new module in "tracker" folder with name your_tracker_name. This folder must contain the implementation of your tracker. Note that your tracker class must inherit from the base tracker class ```tracker.base.BaseTracker```.
 The "\_\_init\_\_.py" inside your tracker folder must contain the following lines,  
```python
from .tracker_file import TrackerClass

def get_tracker_class():
    return TrackerClass
```
Here, ```TrackerClass``` is the name of your tracker class. See the [file for ATOM](tracker/atom/__init__.py) as reference.

Next, you need to create a folder "parameter/your_tracker_name", where the parameter settings for the tracker should be stored. The parameter fil shall contain a ```parameters()``` function that returns a ```TrackerParams``` struct. See the [default parameter file for ATOM](parameter/atom/default.py) as an example.

 
 