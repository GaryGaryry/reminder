caffe 的安装过程中遇到了诸多的坑，在此加以记录，以备后用：
ps: 目前仅安装好python2.7版本，python3.6失败

动因：公司的face segmentation需要
参考资料：
1. http://caffe.berkeleyvision.org/installation.html
2. http://caffe.berkeleyvision.org/install_osx.html
3. http://sugar918.com/index.php/2018/01/12/mac_shang_caffe_pei_zhi_python3_5_xiang_jie/
4. https://github.com/psi4/psi4/issues/313

python 2.7:
brew install -vd snappy leveldb gflags glog szip lmdb
brew install --build-from-source --with-python -vd protobuf
brew install --build-from-source -vd boost boost-python
brew install openblas



conda create -n caffe python=2.7
source activate caffe
cd ~
git clone https://github.com/BVLC/caffe.git
cd caffe/python
for req in $(cat requirements.txt); do pip install $req; done
conda install opencv ipython

cd ~/caffe
cp Makefile.config.example Makefile.config
vim Makefile.config
CPU_ONLY: = 1
BLAS := open
BLAS_INCLUDE := $(shell brew --prefix openblas)/include
BLAS_LIB := $(shell brew --prefix openblas)/lib

ANACONDA_HOME := $(HOME)/anaconda3
PYTHON_INCLUDE := $(ANACONDA_HOME)/envs/caffe/include \
                  $(ANACONDA_HOME)/envs/caffe/include/python2.7 \
                  $(ANACONDA_HOME)/envs/caffe/lib/python2.7/site-packages/numpy/core/include 
PYTHON_LIB := $(ANACONDA_HOME)/envs/caffe/lib/python2.7
WITH_PYTHON_LAYER := 1
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/local/Cellar
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/local/Cellar
BOOST_LIBRARYDIR := /usr/local/Cellar
INCLUDE_DIRS += $(shell brew --prefix)/include
LIBRARY_DIRS += $(shell brew --prefix)/lib


mkdir ~/caffe/build
cd ~/caffe/build
cmake -DBoost_DEBUG=ON ..
vim CMakeCache.txt
//Boost python library (debug) Boost_PYTHON_LIBRARY_DEBUG:FILEPATH=Boost_PYTH    ON_LIBRARY_DEBUG-NOTFOUND
Boost_PYTHON_LIBRARY_DEBUG:FILEPATH=/usr/local/Cellar/boost-python/1.67.0/lib    /libboost_python27-mt.a
//Boost python library (release) Boost_PYTHON_LIBRARY_RELEASE:FILEPATH=Boost_    PYTHON_LIBRARY_RELEASE-NOTFOUND
Boost_PYTHON_LIBRARY_RELEASE:FILEPATH=/usr/local/Cellar/boost-python/1.67.0/l    ib/libboost_python27-mt.a
//Flags used by the CXX compiler during all build types. CMAKE_CXX_FLAGS:STRI    NG=
CMAKE_CXX_FLAGS:STRING=-std=c++11
//Path to a program.
PYTHON_EXECUTABLE:FILEPATH=/Users/gaoyuan/anaconda3/envs/caffe/bin/python2.7
//Path to a file.
PYTHON_INCLUDE_DIR:PATH=/Users/gaoyuan/anaconda3/envs/caffe/include/python2.7
//Path to a library.
PYTHON_LIBRARY:FILEPATH=/Users/gaoyuan/anaconda3/envs/caffe/lib/libpython2.7.dylib
//Dependencies for the target
//在caffe_LIB_DEPENDS:STATIC最添加
/Users/gaoyuan/anaconda3/envs/caffe/lib/libpython2.7.dylib;general;/usr/local/Cellar/boost-python/1.67.0/lib/libboost_python27-mt.a;
//Build Caffe without CUDA support
CPU_ONLY:BOOL=ON

cmake -DBoost_DEBUG=ON ..
make all -j8
make install 
make runtest
make pycaffe

cp -r ~/caffe/python/caffe /Users/gaoyuan/anaconda3/envs/caffe/lib/python2.7/site-packages
