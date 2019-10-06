## Ubnutu16.04下caffe的安装

#### 本文写于2019-9-30

​	首先感谢CSDN用户王英豪的文章[Ubuntu16.04 Caffe 安装步骤记录（超详尽）](https://blog.csdn.net/yhaolpz/article/details/71375762)，用户ChenShuiBiao的[相关文章](https://blog.csdn.net/chenshuibiao/article/details/78734957)以及github用户ghost与wenwei202对protobuf相关的处理方案。本文将对上述方案进行整理改进，以备后用。

---

1. 相关环境安装

   Caffe的安装需要配置很多相关库，所以先使用apt安装库：

   ```bash
   sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
   
   sudo apt-get install --no-install-recommends libboost-all-dev
   
   sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev
   
   sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
   
   sudo apt-get install git cmake build-essential
   ```

   确认安装完毕后即可进行下一步

2. 禁用nouveau

   安装好依赖包后需要禁用 nouveau，只有在禁用掉 nouveau 后才能顺利安装 NVIDIA 显卡驱动，禁用方法就是在 /etc/modprobe.d/blacklist-nouveau.conf 文件中添加一条禁用命令，首先需要打开该文件，通过以下命令打开：

   ```bash
   vim /etc/modprobe.d/blacklist-nouveau.conf
   ```

   如果文件中没有这条指令，添加之：

   ```
   blacklist nouveau option nouveau modeset=0
   ```

   保存后退出，执行下面指令使之生效：

   ```bash
   sudo update-initramfs -u
   ```

   

3. 配置环境变量（可选）

   由于可能在运行中的服务器上搭建caffe，所以服务器并不一定可以重启以配置环境变量，故此步骤可选，如不能重启，可以考虑添加临时环境变量。

   打开`bashrc` ：

   ```bash
   vim ~/.bashrc
   ```

   在文件尾添加：

   ```bash
   export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
   export LD_LIBRARY_PATH=/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH 
   ```

   保存后退出即可。

4. 安装Cuda（可选）

   由于一般服务器在安装caffe之前可能进行过其他机器学习任务，所以Cuda可能已经安装完毕，具体需要询问相关负责人，这里简述一下安装方式：

   首先从Nvidia官网下载Cuda显卡驱动，为了方便查找，我们将Cuda的安装文件放置在`/home` 目录下，然后通过`Ctrl + Alt + F1` 进入文本模式，登录后关闭桌面服务：

   ```bash
   sudo service lightdm stop
   ```

   然后通过 `Ctrl + Alt + F7`  发现已无法成功返回图形化模式，说明桌面服务已成功关闭，注意此步对接下来的 nvidia 驱动安装尤为重要，必需确保桌面服务已关闭。

   `Ctrl + Alt + F1`  进入文本模式，然后运行 CUDA 安装文件进行安装，之前我们已经把 CUDA 安装文件移动至 `/home`，直接通过 sh 命令运行安装文件即可：

   ```bash
   sudo sh cuda_8.0.61_375.26_linux.run --no-opengl-libs
   ```

   其中 `cuda_8.0.61_375.26_linux.run`  是本文使用的CUDA 安装文件名，需替换为自己的 CUDA 安装文件名，若此时忘记可直接通过 ls 文件查看文件名，这也是建议把 CUDA 安装文件移动到`/home` 下的另一个原因。

   执行此命令约1分钟后会出现 0%信息，此时长按回车键让此百分比增长，直到100%，然后按照提示操作即可，先输入 accept ，然后让选择是否安装 nvidia 驱动，这里的选择对应第5步开头，若未安装则输入 “y”，若确保已安装正确驱动则输入“n”。

   剩下的选择则都输入“y”确认安装或确认默认路径安装，开始安装，此时若出现安装失败提示则可能为未关闭桌面服务或在已安装 nvidia 驱动的情况下重复再次安装 nvidia 驱动，安装完成后输入重启命令重启

   ```bash
   reboot
   ```

   然后参考上一步添加环境变量：

   ```bash
   vim ~/.bashrc
   ```

   ```bash
   export PATH=/usr/local/cuda-8.0/bin:$PATH
   export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
   ```

   最后使之生效：

   ```bash
   source ~/.bashrc
   ```

   重启后执行下列指令确认Cuda是否安装完毕：

   ```
   cd /usr/local/cuda-8.0/samples/1_Utilities/deviceQuery
   
   sudo make
   
   ./deviceQuery
   ```

   如果有类似信息，即安装成功：

   ```bash
   ./deviceQuery Starting...
   
    CUDA Device Query (Runtime API) version (CUDART static linking)
   
   Detected 1 CUDA Capable device(s)
   
   Device 0: "GeForce GT 740M"
     CUDA Driver Version / Runtime Version          8.0 / 8.0
     CUDA Capability Major/Minor version number:    3.5
     Total amount of global memory:                 2004 MBytes (2100953088 bytes)
     ( 2) Multiprocessors, (192) CUDA Cores/MP:     384 CUDA Cores
     GPU Max Clock rate:                            1032 MHz (1.03 GHz)
     Memory Clock rate:                             800 Mhz
     Memory Bus Width:                              64-bit
     L2 Cache Size:                                 524288 bytes
     Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
     Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
     Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
     Total amount of constant memory:               65536 bytes
     Total amount of shared memory per block:       49152 bytes
     Total number of registers available per block: 65536
     Warp size:                                     32
     Maximum number of threads per multiprocessor:  2048
     Maximum number of threads per block:           1024
     Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
     Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
     Maximum memory pitch:                          2147483647 bytes
     Texture alignment:                             512 bytes
     Concurrent copy and kernel execution:          Yes with 1 copy engine(s)
     Run time limit on kernels:                     No
     Integrated GPU sharing Host Memory:            No
     Support host page-locked memory mapping:       Yes
     Alignment requirement for Surfaces:            Yes
     Device has ECC support:                        Disabled
     Device supports Unified Addressing (UVA):      Yes
     Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
     Compute Mode:
        < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >
   
   deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 8.0, CUDA Runtime Version = 8.0, NumDevs = 1, Device0 = GeForce GT 740M
   Result = PASS
   ```


5.  安装CUDNN

   首先下载对应版本的CUDNN，官网下载CUDNN需要注册账号。

   下载后解压压缩包，进入`cudn/include` 文件夹，进行下列操作：

   ```bash
   sudo cp cudnn.h /usr/local/cuda/include/
   cd ../lib64
   sudo cp lib* /usr/local/cuda/lib64/
   cd /usr/local/cuda/lib64/sudo rm -rf libcudnn.so libcudnn.so.5
   sudo ln -s libcudnn.so.5.1.10 libcudnn.so.5
   sudo ln -s libcudnn.so.5 libcudnn.so
   ```

   对于倒数第二条指令，需要根据具体cudnn的版本确定：

   ```bash
   locate libcudnn.so
   ```

   观察显示结果中的cudnn版本号确定倒数第二句的链接库

   安装结束可以使用`nvcc -V`验证是否安装成功，如果可以正常显示版本信息就是安装成功了。
   
6.  安装OpenCV

    进入官网下载OpenCV，并解压，本文中使用的是OpenCV3.1，然后执行以下指令：

    ```bash
    mkdir build
    cd build
    cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..
    make
    ```

    如果在`make` 过程中出现`**** has not been declared` 错误，这主要是由OpenCV与CUDA8.0不兼容导致的，可以按如下方法解决：

    ```bash
    vim /opencv-3.1.0/modules/cudalegacy/src/graphcuts.cpp
    ```

    这里的路径可以根据实际情况进行更改

    进入`vim` 后将对应行进行如下修改

    ```C
    //#if !defined (HAVE_CUDA) || defined (CUDA_DISABLER)
    #if !defined (HAVE_CUDA) || defined (CUDA_DISABLER) ||(CUDART_VERSION>=8000)
    ```

    编译后进行安装：

    ```bash
    make install
    ```

    最后可以按如下指令检查OpenCV是否安装成功：

    ```bash
    pkg-config --modelversion opencv
    ```

7.  安装caffe

    首先从caffe的github中clone一份代码：

    ```bash
    git clone https://github.com/BVLC/caffe.git
    ```

    然后复制`Makefile.config.example` 的一份副本至`Makefile.config` ，然后编辑：

    ```bash
    sudo cp Makefile.config.example Makefile.config
    vim Makefile.config
    ```

    修改如下几行：

    ```makefile
    #OLD
    #USE_CUDNN := 1
    #NEW
    USE_CUDNN := 1												
    
    #OLD
    #OPENCV_VERSION := 3										
    #NEW
    OPENCV_VERSION :=3											
    
    #OLD
    #WITH_PYTHON_LAYER := 1
    #NEW
    WITH_PYTHON_LAYER := 1										
    
    #OLD
    INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include		
    LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib		
    
    #NEW									           
    INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
    LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial
    ```

    然后修改`/caffe` 目录下的`Makefile` 文件：

    ```makefile
    #OLD
    NVCCFLAGS +=-ccbin=$(CXX) -Xcompiler-fPIC $(COMMON_FLAGS)						      
    #NEW
    NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)	   
    
    #OLD
    LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_hl hdf5	
    #NEW
    LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial
    																				
    #OLD
    #error-- unsupported GNU version! gcc versions later than 4.9 are not supported!
    #NEW
    //#error-- unsupported GNU version! gcc versions later than 4.9 are not supported!
    ```

    然后回到caffe根目录，执行：

    ```bash
    make all
    ```

    在执行过程中会遇到一系列错误，大部分问题可以通过搜索引擎完美解决[笑]

    但是其中有一个疑难问题，在本文中需要特殊提及：

    在`make` 指令中会出现类似如下错误：

    ```bash
    .build_release/lib/libcaffe.so：对‘google::protobuf::Message::GetTypeName() const’未定义的引用
    .build_release/lib/libcaffe.so：对‘google::protobuf::Message::DebugString() const’未定义的引用
    .build_release/lib/libcaffe.so：对‘google::protobuf::internal::empty_string_’未定义的引用
    .build_release/lib/libcaffe.so：对‘google::protobuf::MessageLite::ParseFromString(std::string const&)’未定义的引用
    .build_release/lib/libcaffe.so：对‘google::protobuf::internal::NameOfEnum(google::protobuf::EnumDescriptor const*, int)’未定义的引用
    ```

    造成这个问题的原因有两个：第一，`caffe` 使用的`protobuf` 版本是比较老旧的`protobuf-2.5.*` 或`protobuf-2.6.0` ，而目前最新的`protobuf` 是`3.*` 以上的版本，所以我们需要将`protobuf` 的版本转成较低的版本。第二，Ubuntu使用的默认GCC版本是 `GCC 4.9.*` 而编译使用的GCC一般是`GCC 5.3.*` (此处仍需讨论)，所以我们需要把默认的GCC转换成高版本的GCC。

    由于`protobuf` 的低版本需要手动进行编译，所以GCC的版本也会影响编译的结果，所以先更改GCC版本：

    ```bash
    ll /usr/bin/gcc*
    which gcc
    ll /usr/bin/gcc
    
    sudo ln -s /usr/bin/gcc-5* /usr/bin/gcc -f
    sudo ln -s /usr/bin/gcc++c-5 /usr/bin/gcc -f
    sudo ln -s /usr/bin/g++-5 /usr/bin/g++ -f
    
    gcc -v
    ```

    如果输出的gcc版本是5.*，就说明已经成功了。

    卸载已有`protobuf` ：

    ```bash
    sudo apt-get clean
    sudo apt-get autoclea
    sudo apt-get autoremove libprotobuf-dev protobuf-compiler
    ```

    下载protobuf2.6.0安装包(网上有资源)，然后安装：

    ```bash
    tar zxvf protobuf-2.6.1.tar.gz
    cd protobuf-2.6.1
    ./configure --prefix=/usr/local/protoc/ CC=/usr/bin/gcc
    make install
    ```

    `--prefix=/usr/local`是目标安装目录，可以按照需求修改

    安装成功后，需要将对应库文件部署到对应文件夹：

    ```bash
    cd /usr/local/protoc
    mv -r bin/* /usr/bin/
    mv lib/lib* /usr/lib/
    mv lib/pkgconfig/* /usr/lib/pkgconfig/
    mv include/* /usr/include
    ```

    再次回到caffe目录，执行`make` 即可。

