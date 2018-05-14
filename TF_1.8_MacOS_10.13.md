# Compile TensorFlow 1.8 on macOS 10.13.4 with GPU acceleration and optionally eGPU support.
I compiled TensorFlow for 10.13.4 after upgrading from 10.11.6, making sure all Homebrew packages were up-to-date first, eliminating the need to rool back to Command-Line Tools 8.2.3.

## Requirements

* Disable SIP on mac
* NVIDIA Web-Drivers
* CUDA-Drivers
* CUDA 9.1 Toolkit
* cuDNN 7.0.5
* Python 3.6
* Apple Command-Line-Tools 8.3.2
* bazel


## Preparations
#### NVIDIA Web-Drivers
Important for 10.13. Download and install the latest NVIDIA graphics drivers before installing CUDA drivers.

https://www.insanelymac.com/forum/topic/324195-nvidia-web-driver-updates-for-macos-high-sierra-update-04252018/

#### Homebrew
If not already installed, Homebrew will also install the latest Apple Command-Line-Tools
`$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

#### Python dependencies
`$ pip install six numpy wheel`

#### coreutils
`$ brew install coreutils`

#### bazel 
'brew install bazel'


#### Downgrad Command-Line-Tools to Version 8.3.2
If you have a more recent version of CLT, dowload 8.3.2 and downgrade.

```
$ sudo mv /Library/Developer/CommandLineTools /Library/Developer/CommandLineTools_backup
$ sudo xcode-select --switch /Library/Developer/CommandLineTools
```

### Install CUDA Toolkit 9.1 with Cuda Drivers
[Download CUDA-9.1](https://developer.nvidia.com/cuda-downloads?target_os=MacOSX&target_arch=x86_64&target_version=1013&target_type=dmglocal)
```

### Install NVIDIA cuDNN
Download [cuDNN 7.0.5](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v7.0.5/prod/9.1_20171129/cudnn-9.1-osx-x64-v7-ga)[^1]


Change into the directory you unzipped cuDNN to.
```
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib/libcudnn_static.a /usr/local/cuda/lib
sudo cp cuda/lib/libcudnn.7.dylib /usr/local/cuda/lib
sudo ln -s /usr/local/cuda/lib/libcudnn.7.dylib /usr/local/cuda/lib/libcudnn.dylib
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib/libcudnn*
```

### Clone TensorFlow from Repository
```
$ cd ~/temp
$ git clone https://github.com/tensorflow/tensorflow
$ cd tensorflow
$ git checkout v1.8
```

#### Patch files
Copy the files in the patch folder to 

`/tensorflow/tensorflow/core/kernels`

#### Add the following to you .Bash_profile
```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export CUDA_HOME=/usr/local/cuda
export DYLD_LIBRARY_PATH=/usr/local/cuda/lib:/usr/local/cuda/extras/CUPTI/lib\
                         ${DYLD_LIBRARY_PATH:+:${DYLD_LIBRARY_PATH}}
export LD_LIBRARY_PATH=$DYLD_LIBRARY_PATH
```


## Prepare Build

````
$ bazel clean --expunge
$ ./configure

```
You have bazel 0.13 installed.
Please specify the location of python. [Default is /Users/user/.pyenv/versions/tensorflow-gpu/bin/python]: 


Found possible Python library paths:
  /Users/user/.pyenv/versions/tensorflow-gpu/lib/python3.6/site-packages
Please input the desired Python library path to use.  Default is [/Users/user/.pyenv/versions/tensorflow-gpu/lib/python3.6/site-packages]

Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: n
No Google Cloud Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
No Hadoop File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
No Amazon S3 File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Apache Kafka Platform support? [y/N]: n
No Apache Kafka Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [y/N]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with GDR support? [y/N]: n
No GDR support will be enabled for TensorFlow.

Do you wish to build TensorFlow with VERBS support? [y/N]: n
No VERBS support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 9.0]: 9.1


Please specify the location where CUDA 9.1 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 


Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7.0]: 


Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:


Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 3.5,5.2]6.1


Do you want to use clang as CUDA compiler? [y/N]: n
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: 


Do you wish to build TensorFlow with MPI support? [y/N]: 
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: 
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
Configuration finished
```

#### Set Environment Variables
```
export CUDA_HOME=/usr/local/cuda
export DYLD_LIBRARY_PATH=/usr/local/cuda/lib:/usr/local/cuda/extras/CUPTI/lib
export LD_LIBRARY_PATH=$DYLD_LIBRARY_PATH
export PATH=$PATH:$DYLD_LIBRARY_PATH
```

## Build Process

`$ bazel build --config=cuda --config=opt --action_env PATH --action_env LD_LIBRARY_PATH --action_env DYLD_LIBRARY_PATH //tensorflow/tools/pip_package:build_pip_package`

#### Create wheel file and install it
```
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
$ cd ~
$ pip install /tmp/tensorflow_pkg/tensorflow-1.8-cp36-cp36m-macosx_10_13_x86_64.whl
```

