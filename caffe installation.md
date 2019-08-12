

# caffe model
https://github.com/soeaver/caffe-model/tree/master/cls

## Install caffe on Ubuntu 16.04 with gpu

#### useful links:
https://github.com/BVLC/caffe/wiki/Ubuntu-16.04-or-15.10-Installation-Guide
https://blog.csdn.net/wuzuyu365/article/details/52430657


## Install thirdparty packages
### 1. install packages
<pre> 
	sudo apt update
	sudo apt upgrade
	sudo apt install -y build-essential cmake git pkg-config
	sudo apt install -y libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler
	sudo apt install -y libatlas-base-dev 
	sudo apt install -y --no-install-recommends libboost-all-dev
	sudo apt install -y libgflags-dev libgoogle-glog-dev liblmdb-dev
</pre>


### 2. install python
### 3. install cuda (9.0), cudnn (7.0), opencv (3.4.0)

3.1 安裝驅動

	主要步骤：

	1.卸载系统里的Nvidia低版本显卡驱动

		sudo apt-get purge nvidia*

	2.把显卡驱动加入PPA

		sudo add-apt-repository ppa:graphics-drivers
		sudo apt-get update

	3.查找安裝显卡驱动最新的版本号
	查找并安装最新驱动
		sudo apt-cache search nvidia
	或可使用终端命令查看Ubuntu推荐的驱动版本：
		ubuntu-drivers devices
	采用apt-get命令在终端安装：
		sudo apt-get install nvidia-430 nvidia-settings nvidia-prime
	4.重啓驗證
		sudo reboot
		nvidia-smi
3.2 install cuda (9.0)

https://blog.csdn.net/Angela_happy/article/details/80977265
	
	error
	<pre>
		Missing recommended library: libGLU.so
		Missing recommended library: libXi.so
		Missing recommended library: libXmu.so
	</pre>
	solve:
	<pre>
		sudo apt-get install libglu1-mesa libxi-dev libxmu-dev libglu1-mesa-dev
	</pre>
3.3 install cudnn (7.0)

	cudnn的安装非常简单 
https://blog.csdn.net/wanzhen4330/article/details/81699769

	（1）下载安装文件

	按需求下载cudnn的安装文件：
https://developer.nvidia.com/rdp/cudnn-archive

	（2）安装cudnn
	
	解压下载的文件，可以看到cuda文件夹，在当前目录打开终端，执行如下命令：
	
		tar -zxvf cudnn-8.0-linux-x64-v6.0.tgz 

		sudo cp cuda/include/cudnn.h /usr/local/cuda/include/

		sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/ -d

		sudo chmod a+r /usr/local/cuda/include/cudnn.h

		sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
		
	（3）查看cudnn版本
	 
	在终端输入

		cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2

	如果出现以下信息，说明安装成功。 
	
		#define CUDNN_MAJOR 7
		#define CUDNN_MINOR 0
		#define CUDNN_PATCHLEVEL 5
			--
		#define CUDNN_VERSION    (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

		#include "driver_types.h"

	安装完成后可用 nvcc -V 命令验证是否安装成功，若出现以下信息则表示安装成功：

		yhao@yhao-X550VB:~$ nvcc -V
		nvcc: NVIDIA (R) Cuda compiler driver
		Copyright (c) 2005-2016 NVIDIA Corporation
		Built on Tue_Jan_10_13:22:03_CST_2017
		Cuda compilation tools, release 8.0, V8.0.61
3.4 install opencv (3.4.0)

https://blog.csdn.net/xd1723138323/article/details/80498381

### Adapt caffe config files
1. download caffe: [github link](https://github.com/BVLC/caffe)

2. If download caffe from [github link](https://github.com/BVLC/caffe), the version should be 1.0. Go to caffe folder, edit _/src/caffe/util/blocking_queue.cpp_, after line 89, add new line: _template class BlockingQueue<Datum*>;_

3. Modify Makefile.config
+ 3.1 cp Makefile.config.example Makefile.config
+ 3.2 edit Makefile.config, find and modify the following lines:
<pre>
	PYTHON_INCLUDE := /usr/include/python2.7 /usr/local/lib/python2.7/dist-packages/numpy/core/include  
	WITH_PYTHON_LAYER := 1  
	INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial  
	LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial  
</pre>
+ 3.3 since cuda 8.0, compute capability 2.0 and 2.1 are discarded, so delete the following two lines:
<pre>
	-gencode arch=compute_20,code=sm_20
	-gencode arch=compute_20,code=sm_21
</pre>

4. fix libhdf5 links
<pre>
	find . -type f -exec sed -i -e 's^"hdf5.h"^"hdf5/serial/hdf5.h"^g' -e 's^"hdf5_hl.h"^"hdf5/serial/hdf5_hl.h"^g' '{}' \;
	cd /usr/lib/x86_64-linux-gnu
	sudo ln -s libhdf5_serial.so.10.1.0 libhdf5.so
	sudo ln -s libhdf5_serial_hl.so.10.0.2 libhdf5_hl.so
</pre>

5. install python packages
<pre>
	cd python
	for req in $(cat requirements.txt); do sudo -H pip install $req --upgrade; done
</pre>

6. Modify Makefile
replace this line:
<pre>
	NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
</pre>
with the following line:
<pre>
	NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
</pre>

7. Modify CMakeLists.txt
add the following line:
<pre>
	# ---[ Includes
	set(${CMAKE_CXX_FLAGS} "-D_FORCE_INLINES ${CMAKE_CXX_FLAGS}")
</pre>


### Build and Compile caffe
<pre>
	make all -j8
	make test -j8
	sudo make runtest -j8
	sudo make pycaffe -j8 (add "export PYTHONPATH=/home/caffe/python:$PYTHONPATH" to "~/.bashrc")
</pre>


### Debug
1. 
error at make all: 
<pre>
	caffe.cpp:(.text+0x15eb): undefined reference to `caffe::Net<float>::CopyTrainedLayersFrom(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)` collect2: error: ld returned 1 exit status
</pre>
solve:
<pre>
	delete libcaffe.so present under /usr/local/lib
</pre>
links:
	https://github.com/rbgirshick/py-faster-rcnn/issues/477
	https://github.com/BVLC/caffe/issues/3396

2.
error:
<pre>
	.build_release/tools/caffe: error while loading shared libraries: libcudart.so.9.0: cannot open shared object file: No such file or directory
</pre>
solve:
<pre>
	in ~/.bashrc, should have the two lines:
		export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
		export PATH=/usr/local/cuda/bin:$PATH
	sudo ldconfig /usr/local/cuda/lib64
</pre>
links:
	https://github.com/BVLC/caffe/issues/4944

3.
error at make pycaffe:
<pre>
	python/caffe/_caffe.cpp:10:31: fatal error: numpy/arrayobject.h: No such file or directory
</pre>
solve:
<pre>
	sudo apt install python-numpy
</pre>
links:
	https://blog.csdn.net/wuzuyu365/article/details/52430657

4.
error at _import caffe_ in python:
<pre>
	ImportError: No module named skimage.io
</pre>
solve:
<pre>
	pip install scikit-image
	sudo apt-get install python-matplotlib python-numpy python-pil python-scipy
	sudo apt-get install build-essential cython
	sudo apt-get install python-skimage
</pre>
links:
	https://github.com/yahoo/open_nsfw/issues/13
	https://github.com/BVLC/caffe/issues/50

5.
pip错误 ImportError: No module named _internal

error at _import caffe_ in python:
<pre>
	ImportError: No module named google.protobuf.internal
</pre>
solve:
<pre>
	sudo apt install python-protobuf
</pre>
links:
	https://stackoverflow.com/questions/37666241/importing-caffe-results-in-importerror-no-module-named-google-protobuf-interna
	https://stackoverflow.com/questions/37666241/importing-caffe-results-in-importerror-no-module-named-google-protobuf-interna/37905483
	
6.
error pip错误 ImportError: No module named _internal
<pre>
	Traceback (most recent call last):
	File "/home/ubuntu/.local/bin/pip", line 7, in <module>
	 
	from pip._internal import main
	 
	ImportError: No module named _internal
</pre>
solve:
<pre>
	强制重新安装pip3
	wget https://bootstrap.pypa.io/get-pip.py  --no-check-certificate
	 
	sudo python3 get-pip.py --force-reinstall
</pre>

7.
<pre>
	安装 requirements.txt列表
	pip install -r requirements.txt -i --user
	
	
	通过cd指令转到caffe下的python目录中运行下面指令

	for req in $(cat requirements.txt); do pip install $req; done

	如果提示权限不够的话就是

	for req in $(cat requirements.txt); do sudo pip install $req; done
</pre>

8.
error：
<pre>
	from pip import __main__
	ImportError: No module named 'pip'
</pre>
solve:
<pre>
	wget https://bootstrap.pypa.io/get-pip.py  --no-check-certificate
	sudo python get-pip.py
</pre>

9.
error：
<pre>
	g++: internal compiler error: Killed (program cc1plus)
	Please submit a full bug report,
</pre>
solve:
<pre>
	主要原因大体上是因为内存不足,有点坑 临时使用交换分区来解决吧

 

	sudo dd if=/dev/zero of=/swapfile bs=64M count=16
	sudo mkswap /swapfile
	sudo swapon /swapfile

	After compiling, you may wish to


	Code:
	sudo swapoff /swapfile
	sudo rm /swapfile
</pre>

10.
install python3.5

【说明】

	  正常情况下你的ubuntu系统是已经自带了python的，不过自带的版本是2.7的，而现在的Python3.5和2.7其实已经非常不同了，作为开发学习的话还是新版本的Python3.5吧。

【安装】

	  1、首先python不在ubuntu的软件仓库，所以我们需要去PPA上找软件源，打开终端，输入下面的命令：sudo add-apt-repository ppa:fkrull/deadsnakes  

	  2、fkrull/deadsnakes就是ubuntu提供的python的repository。接下来接着输入：sudo apt-get update  

	  3、sudo apt-get install python3.5 

【修改配置】

	  1、安装完成之后，你在终端中输入python，输出的信息里面会提示是2.7版本的，也就是说默认打开的并不是刚才安装好的3.5，所以还需要我们重新修改一下链接。方法如下：第一步：先备份原来的链接（在对系统文件执行删除之前进行备份是个好习惯）

	  sudo cp /usr/bin/python /usr/bin/python_bak
	  2、第二步：删除原来的指向2.7版本的默认链接：sudo rm /usr/bin/python  

	  3、第三步：重新指定链接指向3.5版本：sudo ln -s /usr/bin/python3.5 /usr/bin/python  

	  4、大功告成，此时在终端再输入python，输出的信息就是3.5版本了。
<pre>

caffe+python3.5的配置

	打开Makefile.config
	去掉下面的注释：

	PYTHON_LIBRARIES := boost_python3 python3.5m
	PYTHON_INCLUDE := /usr/include/python3.5m \
                                            /usr/lib/python3.5/dist-packages/numpy/core/include

	然后重新编译。

	如果编译过程中遇到cannot find -lboost_python3     

	查看/usr/lib/x86_64-linux-gnu下只有：libboost_python-py35.so

 	问题解决办法，将boost_python3改成boost_python-py35

	PYTHON_LIBRARIES := boost_python-py35 python3.5m      


</pre>

https://github.com/guonannan123456/restnet50/blob/master/caffe%20installation.md

