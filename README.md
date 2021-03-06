# `timeseries` package for fastai v2
> **`timeseries`** is a Timeseries Classification and Regression package for fastai v2.


<a href="https://colab.research.google.com/github/ai-fast-track/timeseries/blob/master/nbs/index.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

## Credit
> timeseries for fastai v2 was inspired by by Ignacio's Oguiza timeseriesAI (https: //github.com/timeseriesAI/timeseriesAI.git).> Inception Time model definition is a modified version of [Ignacio Oguiza] (https: //github.com/timeseriesAI/timeseriesAI/blob/master/torchtimeseries/models/InceptionTime.py) and [Thomas Capelle] (https://github.com/tcapelle/TimeSeries_fastai/blob/master/inception.py) implementaions

## Installing **`timeseries`** on local machine as an editable package

1- Only if you have not already installed `fastai v2` 
Install [fastai2](https://dev.fast.ai/#Installing) by following the steps described there.

2- Install timeseries package by following the instructions here below:

```
git clone https://github.com/ai-fast-track/timeseries.git
cd timeseries
pip install -e .
```

# pip installing **`timeseries`** from repo either locally or in Google Colab - Start Here

## Installing fastai v2

```
!pip install git+https://github.com/fastai/fastai2.git
```

## Installing `timeseries` package from github

```
!pip install git+https://github.com/ai-fast-track/timeseries.git
```

# *pip Installing - End Here*

# `Usage`

```
%reload_ext autoreload
%autoreload 2
%matplotlib inline
```

```
from fastai2.basics import *
```

```
from timeseries.all import *
```

# Tutorial on timeseries package for fastai v2

## Example : NATOS dataset

<img src="https://github.com/ai-fast-track/timeseries/blob/master/images/NATOPS.jpg?raw=1">

## Right Arm vs Left Arm (3: 'Not clear' Command (see picture here above))
<br>
<img src="https://github.com/ai-fast-track/timeseries/blob/master/images/ts-right-arm.png?raw=1"><img src="https://github.com/ai-fast-track/timeseries/blob/master/images/ts-left-arm.png?raw=1">

## Description
The data is generated by sensors on the hands, elbows, wrists and thumbs. The data are the x,y,z coordinates for each of the eight locations. The order of the data is as follows:

## Channels (24)

0.	Hand tip left, X coordinate
1.	Hand tip left, Y coordinate
2.	Hand tip left, Z coordinate
3.	Hand tip right, X coordinate
4.	Hand tip right, Y coordinate
5.	Hand tip right, Z coordinate
6.	Elbow left, X coordinate
7.	Elbow left, Y coordinate
8.	Elbow left, Z coordinate
9.	Elbow right, X coordinate
10.	Elbow right, Y coordinate
11.	Elbow right, Z coordinate
12.	Wrist left, X coordinate
13.	Wrist left, Y coordinate
14.	Wrist left, Z coordinate
15.	Wrist right, X coordinate
16.	Wrist right, Y coordinate
17.	Wrist right, Z coordinate
18.	Thumb left, X coordinate
19.	Thumb left, Y coordinate
20.	Thumb left, Z coordinate
21.	Thumb right, X coordinate
22.	Thumb right, Y coordinate
23.	Thumb right, Z coordinate

## Classes (6)
The six classes are separate actions, with the following meaning:
 
1: I have command 
2: All clear 
3: Not clear 
4: Spread wings 
5: Fold wings 
6: Lock wings

## Download data using `download_unzip_data_UCR(dsname=dsname)` method

```
dsname =  'NATOPS' #'NATOPS', 'LSST', 'Wine', 'Epilepsy', 'HandMovementDirection'
```

```
# url = 'http://www.timeseriesclassification.com/Downloads/NATOPS.zip'
path = unzip_data(URLs_TS.NATOPS)
path
```




    Path('/home/farid/.fastai/data/NATOPS')



## Why do I have to concatenate train and test data?
Both Train and Train dataset contains 180 samples each. We concatenate them in order to have one big dataset and then split into train and valid dataset using our own split percentage (20%, 30%, or whatever number you see fit)

```
fname_train = f'{dsname}_TRAIN.arff'
fname_test = f'{dsname}_TEST.arff'
fnames = [path/fname_train, path/fname_test]
fnames
```




    [Path('/home/farid/.fastai/data/NATOPS/NATOPS_TRAIN.arff'),
     Path('/home/farid/.fastai/data/NATOPS/NATOPS_TEST.arff')]



```
data = TSData.from_arff(fnames)
print(data)
```

    TSData:
     Datasets names (concatenated): ['NATOPS_TRAIN', 'NATOPS_TEST']
     Filenames:                     [Path('/home/farid/.fastai/data/NATOPS/NATOPS_TRAIN.arff'), Path('/home/farid/.fastai/data/NATOPS/NATOPS_TEST.arff')]
     Data shape: (360, 24, 51)
     Targets shape: (360,)
     Nb Samples: 360
     Nb Channels:           24
     Sequence Length: 51


```
items = data.get_items()
```

```
idx = 1
x1, y1 = data.x[idx],  data.y[idx]
y1
```




    '3.0'



```

# You can select any channel to display buy supplying a list of channels and pass it to `chs` argument
# LEFT ARM
# show_timeseries(x1, title=y1, chs=[0,1,2,6,7,8,12,13,14,18,19,20])

```

```
# RIGHT ARM
# show_timeseries(x1, title=y1, chs=[3,4,5,9,10,11,15,16,17,21,22,23])
```

```
# ?show_timeseries(x1, title=y1, chs=range(0,24,3)) # Only the x axis coordinates

```

```
seed = 42
splits = RandomSplitter(seed=seed)(range_of(items)) #by default 80% for train split and 20% for valid split are chosen 
splits
```




    ((#288) [304,281,114,329,115,130,338,294,94,310...],
     (#72) [222,27,96,253,274,35,160,172,302,146...])



# Using `Datasets` class

## Creating a Datasets object

```
tfms = [[ItemGetter(0), ToTensorTS()], [ItemGetter(1), Categorize()]]

# Create a dataset
ds = Datasets(items, tfms, splits=splits)
```

```
ax = show_at(ds, 2, figsize=(1,1))
```

    3.0



![svg](docs/images/output_33_1.svg)


# Create a `Dataloader` objects

## 1st method : using `Datasets` object

```
bs = 128                            
# Normalize at batch time
tfm_norm = Normalize(scale_subtype = 'per_sample_per_channel', scale_range=(0, 1)) # per_sample , per_sample_per_channel
# tfm_norm = Standardize(scale_subtype = 'per_sample')
batch_tfms = [tfm_norm]

dls1 = ds.dataloaders(bs=bs, val_bs=bs * 2, after_batch=batch_tfms, num_workers=0, device=default_device()) 
```

```
dls1.show_batch(max_n=9, chs=range(0,12,3))
```


![svg](docs/images/output_37_0.svg)


# Using `DataBlock` class

## 2nd method : using `DataBlock` and `DataBlock.get_items()` 

```
getters = [ItemGetter(0), ItemGetter(1)]  
tsdb = DataBlock(blocks=(TSBlock, CategoryBlock),
                   get_items=get_ts_items,
                   getters=getters,
                   splitter=RandomSplitter(seed=seed),
                   batch_tfms = batch_tfms)
```

```
tsdb.summary(fnames)
```

    Setting-up type transforms pipelines
    Collecting items from [Path('/home/farid/.fastai/data/NATOPS/NATOPS_TRAIN.arff'), Path('/home/farid/.fastai/data/NATOPS/NATOPS_TEST.arff')]
    Found 360 items
    2 datasets of sizes 288,72
    Setting up Pipeline: itemgetter -> ToTensorTS
    Setting up Pipeline: itemgetter -> Categorize
    
    Building one sample
      Pipeline: itemgetter -> ToTensorTS
        starting from
          ([[-0.540579 -0.54101  -0.540603 ... -0.56305  -0.566314 -0.553712]
     [-1.539567 -1.540042 -1.538992 ... -1.532014 -1.534645 -1.536015]
     [-0.608539 -0.604609 -0.607679 ... -0.593769 -0.592854 -0.599014]
     ...
     [ 0.454542  0.449924  0.453195 ...  0.480281  0.45537   0.457275]
     [-1.411445 -1.363464 -1.390869 ... -1.468123 -1.368706 -1.386574]
     [-0.473406 -0.453322 -0.463813 ... -0.440582 -0.427211 -0.435581]], 2.0)
        applying itemgetter gives
          [[-0.540579 -0.54101  -0.540603 ... -0.56305  -0.566314 -0.553712]
     [-1.539567 -1.540042 -1.538992 ... -1.532014 -1.534645 -1.536015]
     [-0.608539 -0.604609 -0.607679 ... -0.593769 -0.592854 -0.599014]
     ...
     [ 0.454542  0.449924  0.453195 ...  0.480281  0.45537   0.457275]
     [-1.411445 -1.363464 -1.390869 ... -1.468123 -1.368706 -1.386574]
     [-0.473406 -0.453322 -0.463813 ... -0.440582 -0.427211 -0.435581]]
        applying ToTensorTS gives
          TensorTS of size 24x51
      Pipeline: itemgetter -> Categorize
        starting from
          ([[-0.540579 -0.54101  -0.540603 ... -0.56305  -0.566314 -0.553712]
     [-1.539567 -1.540042 -1.538992 ... -1.532014 -1.534645 -1.536015]
     [-0.608539 -0.604609 -0.607679 ... -0.593769 -0.592854 -0.599014]
     ...
     [ 0.454542  0.449924  0.453195 ...  0.480281  0.45537   0.457275]
     [-1.411445 -1.363464 -1.390869 ... -1.468123 -1.368706 -1.386574]
     [-0.473406 -0.453322 -0.463813 ... -0.440582 -0.427211 -0.435581]], 2.0)
        applying itemgetter gives
          2.0
        applying Categorize gives
          TensorCategory(1)
    
    Final sample: (TensorTS([[-0.5406, -0.5410, -0.5406,  ..., -0.5630, -0.5663, -0.5537],
            [-1.5396, -1.5400, -1.5390,  ..., -1.5320, -1.5346, -1.5360],
            [-0.6085, -0.6046, -0.6077,  ..., -0.5938, -0.5929, -0.5990],
            ...,
            [ 0.4545,  0.4499,  0.4532,  ...,  0.4803,  0.4554,  0.4573],
            [-1.4114, -1.3635, -1.3909,  ..., -1.4681, -1.3687, -1.3866],
            [-0.4734, -0.4533, -0.4638,  ..., -0.4406, -0.4272, -0.4356]]), TensorCategory(1))
    
    
    Setting up after_item: Pipeline: ToTensor
    Setting up before_batch: Pipeline: 
    Setting up after_batch: Pipeline: Normalize
    
    Building one batch
    Applying item_tfms to the first sample:
      Pipeline: ToTensor
        starting from
          (TensorTS of size 24x51, TensorCategory(1))
        applying ToTensor gives
          (TensorTS of size 24x51, TensorCategory(1))
    
    Adding the next 3 samples
    
    No before_batch transform to apply
    
    Collating items in a batch
    
    Applying batch_tfms to the batch built
      Pipeline: Normalize
        starting from
          (TensorTS of size 4x24x51, TensorCategory([1, 5, 4, 5]))
        applying Normalize gives
          (TensorTS of size 4x24x51, TensorCategory([1, 5, 4, 5]))


```
# num_workers=0 is Microsoft Windows
dls2 = tsdb.dataloaders(fnames, num_workers=0, device=default_device())
```

```
dls2.show_batch(max_n=9, chs=range(0,12,3))
```


![svg](docs/images/output_43_0.svg)


## 3rd method : using `DataBlock` and passing `items` object to the `DataBlock.dataloaders()`

```
getters = [ItemGetter(0), ItemGetter(1)] 
tsdb = DataBlock(blocks=(TSBlock, CategoryBlock),
                   getters=getters,
                   splitter=RandomSplitter(seed=seed))
```

```
dls3 = tsdb.dataloaders(data.get_items(), batch_tfms=batch_tfms, num_workers=0, device=default_device())
```

```
dls3.show_batch(max_n=9, chs=range(0,12,3))
```


![svg](docs/images/output_47_0.svg)


## 4th method : using `TSDataLoaders` class and `TSDataLoaders.from_files()`

```
dls4 = TSDataLoaders.from_files(fnames, batch_tfms=batch_tfms, num_workers=0, device=default_device())
```

```
dls4.show_batch(max_n=9, chs=range(0,12,3))
```


![svg](docs/images/output_50_0.svg)


# Train Model

```
# Number of channels (i.e. dimensions in ARFF and TS files jargon)
c_in = get_n_channels(dls2.train) # data.n_channels
# Number of classes
c_out= dls2.c 
c_in,c_out
```




    (24, 6)



## Create model

```
model = inception_time(c_in, c_out).to(device=default_device())
model
```




    Sequential(
      (0): SequentialEx(
        (layers): ModuleList(
          (0): InceptionModule(
            (convs): ModuleList(
              (0): Conv1d(24, 32, kernel_size=(39,), stride=(1,), padding=(19,), bias=False)
              (1): Conv1d(24, 32, kernel_size=(19,), stride=(1,), padding=(9,), bias=False)
              (2): Conv1d(24, 32, kernel_size=(9,), stride=(1,), padding=(4,), bias=False)
            )
            (maxpool_bottleneck): Sequential(
              (0): MaxPool1d(kernel_size=3, stride=1, padding=1, dilation=1, ceil_mode=False)
              (1): Conv1d(24, 32, kernel_size=(1,), stride=(1,), bias=False)
            )
            (bn_relu): Sequential(
              (0): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
              (1): ReLU()
            )
          )
        )
      )
      (1): SequentialEx(
        (layers): ModuleList(
          (0): InceptionModule(
            (bottleneck): Conv1d(128, 32, kernel_size=(1,), stride=(1,))
            (convs): ModuleList(
              (0): Conv1d(32, 32, kernel_size=(39,), stride=(1,), padding=(19,), bias=False)
              (1): Conv1d(32, 32, kernel_size=(19,), stride=(1,), padding=(9,), bias=False)
              (2): Conv1d(32, 32, kernel_size=(9,), stride=(1,), padding=(4,), bias=False)
            )
            (maxpool_bottleneck): Sequential(
              (0): MaxPool1d(kernel_size=3, stride=1, padding=1, dilation=1, ceil_mode=False)
              (1): Conv1d(128, 32, kernel_size=(1,), stride=(1,), bias=False)
            )
            (bn_relu): Sequential(
              (0): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
              (1): ReLU()
            )
          )
        )
      )
      (2): SequentialEx(
        (layers): ModuleList(
          (0): InceptionModule(
            (bottleneck): Conv1d(128, 32, kernel_size=(1,), stride=(1,))
            (convs): ModuleList(
              (0): Conv1d(32, 32, kernel_size=(39,), stride=(1,), padding=(19,), bias=False)
              (1): Conv1d(32, 32, kernel_size=(19,), stride=(1,), padding=(9,), bias=False)
              (2): Conv1d(32, 32, kernel_size=(9,), stride=(1,), padding=(4,), bias=False)
            )
            (maxpool_bottleneck): Sequential(
              (0): MaxPool1d(kernel_size=3, stride=1, padding=1, dilation=1, ceil_mode=False)
              (1): Conv1d(128, 32, kernel_size=(1,), stride=(1,), bias=False)
            )
            (bn_relu): Sequential(
              (0): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
              (1): ReLU()
            )
          )
          (1): Shortcut(
            (act_fn): ReLU(inplace=True)
            (conv): Conv1d(128, 128, kernel_size=(1,), stride=(1,), bias=False)
            (bn): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
          )
        )
      )
      (3): SequentialEx(
        (layers): ModuleList(
          (0): InceptionModule(
            (bottleneck): Conv1d(128, 32, kernel_size=(1,), stride=(1,))
            (convs): ModuleList(
              (0): Conv1d(32, 32, kernel_size=(39,), stride=(1,), padding=(19,), bias=False)
              (1): Conv1d(32, 32, kernel_size=(19,), stride=(1,), padding=(9,), bias=False)
              (2): Conv1d(32, 32, kernel_size=(9,), stride=(1,), padding=(4,), bias=False)
            )
            (maxpool_bottleneck): Sequential(
              (0): MaxPool1d(kernel_size=3, stride=1, padding=1, dilation=1, ceil_mode=False)
              (1): Conv1d(128, 32, kernel_size=(1,), stride=(1,), bias=False)
            )
            (bn_relu): Sequential(
              (0): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
              (1): ReLU()
            )
          )
        )
      )
      (4): SequentialEx(
        (layers): ModuleList(
          (0): InceptionModule(
            (bottleneck): Conv1d(128, 32, kernel_size=(1,), stride=(1,))
            (convs): ModuleList(
              (0): Conv1d(32, 32, kernel_size=(39,), stride=(1,), padding=(19,), bias=False)
              (1): Conv1d(32, 32, kernel_size=(19,), stride=(1,), padding=(9,), bias=False)
              (2): Conv1d(32, 32, kernel_size=(9,), stride=(1,), padding=(4,), bias=False)
            )
            (maxpool_bottleneck): Sequential(
              (0): MaxPool1d(kernel_size=3, stride=1, padding=1, dilation=1, ceil_mode=False)
              (1): Conv1d(128, 32, kernel_size=(1,), stride=(1,), bias=False)
            )
            (bn_relu): Sequential(
              (0): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
              (1): ReLU()
            )
          )
        )
      )
      (5): SequentialEx(
        (layers): ModuleList(
          (0): InceptionModule(
            (bottleneck): Conv1d(128, 32, kernel_size=(1,), stride=(1,))
            (convs): ModuleList(
              (0): Conv1d(32, 32, kernel_size=(39,), stride=(1,), padding=(19,), bias=False)
              (1): Conv1d(32, 32, kernel_size=(19,), stride=(1,), padding=(9,), bias=False)
              (2): Conv1d(32, 32, kernel_size=(9,), stride=(1,), padding=(4,), bias=False)
            )
            (maxpool_bottleneck): Sequential(
              (0): MaxPool1d(kernel_size=3, stride=1, padding=1, dilation=1, ceil_mode=False)
              (1): Conv1d(128, 32, kernel_size=(1,), stride=(1,), bias=False)
            )
            (bn_relu): Sequential(
              (0): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
              (1): ReLU()
            )
          )
          (1): Shortcut(
            (act_fn): ReLU(inplace=True)
            (conv): Conv1d(128, 128, kernel_size=(1,), stride=(1,), bias=False)
            (bn): BatchNorm1d(128, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
          )
        )
      )
      (6): AdaptiveConcatPool1d(
        (ap): AdaptiveAvgPool1d(output_size=1)
        (mp): AdaptiveMaxPool1d(output_size=1)
      )
      (7): Flatten(full=False)
      (8): Linear(in_features=256, out_features=6, bias=True)
    )



## Create Learner object

```
# opt_func = partial(Adam, lr=3e-3, wd=0.01)
#Or use Ranger
def opt_func(p, lr=slice(3e-3)): return Lookahead(RAdam(p, lr=lr, mom=0.95, wd=0.01)) 
```

```
#Learner    
loss_func = LabelSmoothingCrossEntropy() 
learn = Learner(dls2, model, opt_func=opt_func, loss_func=loss_func, metrics=accuracy)

print(learn.summary())
```

    Sequential (Input shape: ['64 x 24 x 51'])
    ================================================================
    Layer (type)         Output Shape         Param #    Trainable 
    ================================================================
    Conv1d               64 x 32 x 51         29,952     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         14,592     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         6,912      True      
    ________________________________________________________________
    MaxPool1d            64 x 24 x 51         0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         768        True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,128      True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         39,936     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         19,456     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         9,216      True      
    ________________________________________________________________
    MaxPool1d            64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,096      True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,128      True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         39,936     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         19,456     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         9,216      True      
    ________________________________________________________________
    MaxPool1d            64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,096      True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 128 x 51        16,384     True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,128      True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         39,936     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         19,456     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         9,216      True      
    ________________________________________________________________
    MaxPool1d            64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,096      True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,128      True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         39,936     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         19,456     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         9,216      True      
    ________________________________________________________________
    MaxPool1d            64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,096      True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,128      True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         39,936     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         19,456     True      
    ________________________________________________________________
    Conv1d               64 x 32 x 51         9,216      True      
    ________________________________________________________________
    MaxPool1d            64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 32 x 51         4,096      True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    ReLU                 64 x 128 x 51        0          False     
    ________________________________________________________________
    Conv1d               64 x 128 x 51        16,384     True      
    ________________________________________________________________
    BatchNorm1d          64 x 128 x 51        256        True      
    ________________________________________________________________
    AdaptiveAvgPool1d    64 x 128 x 1         0          False     
    ________________________________________________________________
    AdaptiveMaxPool1d    64 x 128 x 1         0          False     
    ________________________________________________________________
    Flatten              64 x 256             0          False     
    ________________________________________________________________
    Linear               64 x 6               1,542      True      
    ________________________________________________________________
    
    Total params: 472,742
    Total trainable params: 472,742
    Total non-trainable params: 0
    
    Optimizer used: <function opt_func at 0x7f0f6c4d1158>
    Loss function: LabelSmoothingCrossEntropy()
    
    Callbacks:
      - TrainEvalCallback
      - Recorder
      - ProgressCallback


## LR find 

```
lr_min, lr_steep = learn.lr_find()
lr_min, lr_steep
```








    (0.15848932266235352, 0.0006918309954926372)




![svg](docs/images/output_59_2.svg)


## Train

```
#lr_max=1e-3
epochs=30; lr_max=lr_steep;  pct_start=.7; moms=(0.95,0.85,0.95); wd=1e-2
learn.fit_one_cycle(epochs, lr_max=lr_max, pct_start=pct_start,  moms=moms, wd=wd)
# learn.fit_one_cycle(epochs=20, lr_max=lr_steep)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>2.086663</td>
      <td>1.791449</td>
      <td>0.111111</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2.078727</td>
      <td>1.794685</td>
      <td>0.222222</td>
      <td>00:03</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2.059660</td>
      <td>1.801005</td>
      <td>0.277778</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2.032225</td>
      <td>1.810632</td>
      <td>0.208333</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>4</td>
      <td>1.984182</td>
      <td>1.823712</td>
      <td>0.180556</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>5</td>
      <td>1.909669</td>
      <td>1.839020</td>
      <td>0.208333</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>6</td>
      <td>1.833456</td>
      <td>1.846452</td>
      <td>0.194444</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>7</td>
      <td>1.732690</td>
      <td>1.826979</td>
      <td>0.319444</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>8</td>
      <td>1.620669</td>
      <td>1.753253</td>
      <td>0.319444</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>9</td>
      <td>1.518595</td>
      <td>1.438307</td>
      <td>0.430556</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>10</td>
      <td>1.420360</td>
      <td>1.182858</td>
      <td>0.652778</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>11</td>
      <td>1.326363</td>
      <td>0.987260</td>
      <td>0.763889</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>12</td>
      <td>1.245747</td>
      <td>0.782695</td>
      <td>0.777778</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>13</td>
      <td>1.173810</td>
      <td>0.728713</td>
      <td>0.833333</td>
      <td>00:01</td>
    </tr>
    <tr>
      <td>14</td>
      <td>1.109214</td>
      <td>0.704866</td>
      <td>0.819444</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>15</td>
      <td>1.053367</td>
      <td>0.710490</td>
      <td>0.833333</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>16</td>
      <td>1.001073</td>
      <td>0.684964</td>
      <td>0.861111</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>17</td>
      <td>0.955461</td>
      <td>0.678061</td>
      <td>0.902778</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>18</td>
      <td>0.913411</td>
      <td>0.698473</td>
      <td>0.861111</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>19</td>
      <td>0.873898</td>
      <td>0.693197</td>
      <td>0.861111</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>20</td>
      <td>0.837090</td>
      <td>0.650469</td>
      <td>0.888889</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>21</td>
      <td>0.804146</td>
      <td>0.742458</td>
      <td>0.875000</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>22</td>
      <td>0.774491</td>
      <td>0.816232</td>
      <td>0.861111</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>23</td>
      <td>0.747442</td>
      <td>0.665499</td>
      <td>0.861111</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>24</td>
      <td>0.721633</td>
      <td>0.736514</td>
      <td>0.861111</td>
      <td>00:01</td>
    </tr>
    <tr>
      <td>25</td>
      <td>0.697358</td>
      <td>0.567567</td>
      <td>0.972222</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>26</td>
      <td>0.675201</td>
      <td>0.569275</td>
      <td>0.958333</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>27</td>
      <td>0.654944</td>
      <td>0.595504</td>
      <td>0.944444</td>
      <td>00:02</td>
    </tr>
    <tr>
      <td>28</td>
      <td>0.636593</td>
      <td>0.565226</td>
      <td>0.958333</td>
      <td>00:01</td>
    </tr>
    <tr>
      <td>29</td>
      <td>0.620018</td>
      <td>0.560414</td>
      <td>0.972222</td>
      <td>00:01</td>
    </tr>
  </tbody>
</table>


## Plot loss function

```
learn.recorder.plot_loss()
```


![svg](docs/images/output_63_0.svg)


## Show results

```
learn.show_results(max_n=9, chs=range(0,12,3))
```






![svg](docs/images/output_65_1.svg)


```
# #hide
# from nbdev.export import notebook2script
# notebook2script()
# # notebook2script(fname='index.ipynb')
```

```
# #hide
# from nbdev.export2html import _notebook2html
# # notebook2script()
# _notebook2html(fname='index.ipynb')
```

# Fin

<img src="https://github.com/ai-fast-track/timeseries/blob/master/images/tree.jpg?raw=1" width="1440" height="840" alt="">
