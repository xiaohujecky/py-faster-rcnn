# Faster-RCNN安装记录
按照目录下的[README.md](https://github.com/xiaohujecky/py-faster-rcnn/blob/master/README.md)指示来安装，Follow TIPS.   
1. git clone --recursive https://github.com/rbgirshick/py-faster-rcnn.git .   

2. Build the Cython modules:
      ```Shell
    cd $FRCN_ROOT/lib
    make
    ```   
       
3. Build Caffe and pycaffe 
    ```
    make -j8 && make pycaffe
    ```
  不出意料，在这里一般会遇到问题，以下是我遇到的问题。      
  
# 遇到的问题：
1. 在编译caffe-fast-rcnn时，配置Make.config时注意：
    # USE_CUDNN := 1 
    我当时打开了这个选项，实际上我的机器上没有安装cudnn的库，当然会报错啦，具体安装可以参见[How to setup Caffe to run Deep Neural Network](http://corpocrat.com/2014/11/03/how-to-setup-caffe-to-run-deep-neural-network/).
2. 在选择GPU安装时，[Nivida](https://developer.nvidia.com/cudnn)驱动安装在 /opt/cuda/下，而非：CUDA_DIR := /usr/local/cuda，因此，在此修改为
    CUDA_DIR := /opt/cuda  
3. Python的很多库都可以由[Anaconda](https://www.continuum.io/downloads)来安装，独立管理一份python依赖库，非常方便，有个[30分钟教学资料](http://conda.pydata.org/docs/test-drive.html#managing-conda).
     如果安装了Anaconda可以打开选项：
     ```
     ANACONDA_HOME := $(HOME)/anaconda2
     PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
		 $(ANACONDA_HOME)/include/python2.7 \
		 $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include
     ```
    否则，自己指定PYTHON_INCLUDE目录。我在这里踩的坑是，当我使用Anaconda安装的numpy，和scipy库时，编译报错，原因不详，大概是依赖库安装不全，为了节省时间，我直接使用系统原有的python版本（gentoo系统，python2.7.10，scipy，numpy等常用库都已安装）。     
	 
4. python版opencv安装，可以使用[anaconda安装](http://stackoverflow.com/questions/23119413/how-to-install-python-opencv-through-conda)   
     ```
    conda install -c https://conda.binstar.org/menpo opencv
    ```
   因为有opencv2.4.10的安装包，因此直接使用安装包安装：
   在opencv安装目录opencv-2.4.10下，新建python_opencv_install.sh，内容如下：
   ```
    PYTHON_PREFIX=/usr
    
    cmake ..  -DCMAKE_BUILD_TYPE=RELEASE \
    -DWITH_EIGEN=ON \
    -DBUILD_NEW_PYTHON_SUPPORT=ON \
    -DPYTHON_EXECUTABLE=$PYTHON_PREFIX/bin/python2.7 \
    -DPYTHON_INCLUDE_DIR=$PYTHON_PREFIX/include/python2.7/ \
    -DPYTHON_LIBRARY=$PYTHON_PREFIX/lib/libpython2.7.so.1.0 \
    -DPYTHON_NUMPY_INCLUDE_DIR=$PYTHON_PREFIX/lib/python2.7/site-packages/numpy/core/include/ \
    -DPYTHON_PACKAGES_PATH=$PYTHON_PREFIX/lib/python2.7/site-packages/ \
    -DBUILD_PYTHON_SUPPORT=ON
    ```   
    
    ```
    mkdir build
    cd build
    sh ../python_opencv_install.sh 
    ```
    等待安装完毕即可。
5. Boost_python库安装，使用
    sudo pip install boost_python
   可以完成安装，但是在make pycaffe时仍旧提示：Can‘t Find -lboost_python.o
    其实，这个时候/usr/lib/下有 libboost_python-2.7-mt.so、libboost_python-2.7.so、libboost_python-2.7.so.1.55.0, 所以，找不到libboost_python.so*,建个软链似乎能解决问题：
    ln -s libboost_python-2.7.so.1.55.0 libboost_python.so.1.55.0 (未经测试)
    
    一开始，没有意识到这个，去[python_boost安装简介](http://edyfox.codecarver.org/html/boost_python.html)和[Python.Boost](http://www.boost.org/doc/libs/1_61_0/more/getting_started/unix-variants.html)官网下了安装包，从源码[Boost](https://svn.boost.org/trac/boost/wiki/TryModBoost)安装的:
    出现错误：
     ```
    ./b2 headers
    /home/wichtounet/src/modular-boost/tools/build/src/build/feature.jam:493: in feature.validate-value-string from module feature
    error: "none" is not a known value of feature <optimization>
    error: legal values: "off" "speed" "space"
    ......
    ```
    运行./b2时，使用如下命令[问题解决](http://stackoverflow.com/questions/23013433/how-to-install-modular-boost)：
    ./b2 --ignore-site-config 
    
    
     [Anaconda boost python](https://anaconda.org/meznom/boost-python)没有安装成功，未果。
    
6. 其他依赖库：$FRCN_ROOT/caffe-fast-rcnn/python/requirements.txt:
    ```
    pip install -r requirements.txt
    ```
    或者写个脚本install_req.sh：
    ```
    !#/usr/bin/env bash
    for req in $(cat requirements.txt); do pip install $req; done
    ```
    ./install_req.sh
    
    我在这一步，scikit-image没有安装成功，导致import caffe加载失败，原因是direct、blas、Atlas没有安装python版本，问题如同[installing-scipy-and-numpy-using-pip](http://stackoverflow.com/questions/11114225/installing-scipy-and-numpy-using-pip)，scipy版本错误，使用
    ```
    sudo pip install atlas scipy
    ```
    scipy安装不成成功，最后使用 [scipy](http://www.scipy.org/install.html)成功安装，再使用
    ```
    sudo pip install scikit-image
    ```
    
7. 最后，环境变量问题，在~/.bashrc中添加：
  ```
  # added by Anaconda2 4.0.0 installer
  export PATH="/usr/bin:/home/usrname/anaconda2/bin:$PATH"
  export PATH="/data/usrname/software/boost_1_61_0/stage/lib:$PATH"

  export PYTHONPATH=/data/usrname/program/blue_rects/faster-rcnn/caffe-fast-rcnn/distribute/python:$PYTHONPATH
  export LD_LIBRARY_PATH=/data/usrname/program/blue_rects/faster-rcnn/caffe-fast-rcnn/build/lib:/usr/local/lib:$LD_LIBRARY_PATH
  ```
  注意⚠：PYTHONPATH=/data/usrname/program/blue_rects/faster-rcnn/caffe-fast-rcnn/distribute/python,而非/data/usrname/program/blue_rects/faster-rcnn/caffe-fast-rcnn/distribute/python/caffe
  
  
    
    

    
  
