# 个人环境
系统：win11  
CUDA：12.1  
VS版本：2022  
VS installer：安装win的C++桌面开发相关内容  
gcc编译器：MinGW  
```
C:\Users\Administrator>gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=d:/mingw/bin/../libexec/gcc/mingw32/6.3.0/lto-wrapper.exe
Target: mingw32
Configured with: ../src/gcc-6.3.0/configure --build=x86_64-pc-linux-gnu --host=mingw32 --target=mingw32 --with-gmp=/mingw --with-mpfr --with-mpc=/mingw --with-isl=/mingw --prefix=/mingw --disable-win32-registry --with-arch=i586 --with-tune=generic --enable-languages=c,c++,objc,obj-c++,fortran,ada --with-pkgversion='MinGW.org GCC-6.3.0-1' --enable-static --enable-shared --enable-threads --with-dwarf2 --disable-sjlj-exceptions --enable-version-specific-runtime-libs --with-libiconv-prefix=/mingw --with-libintl-prefix=/mingw --enable-libstdcxx-debug --enable-libgomp --disable-libvtv --enable-nls
Thread model: win32
gcc version 6.3.0 (MinGW.org GCC-6.3.0-1)
```
python版本：3.10
python库版本：  
```
pytorch3d               0.7.4
torch                   2.1.0
torchaudio              2.1.0+cu121
torchvision             0.16.0+cu12
```

注意：由于我没有把VS装在C盘，所以替换了程序内的默认路径，你可以用VS Code全局搜索下`Microsoft Visual Studio\\*\\%s\\VC\\Tools\\MSVC\\*\\bin\\Host`  然后你就会看见我的注释了，然后你就把你的路径恢复上去就行。  
```
# 我自己安装的路径，看情况适配
paths = sorted(glob.glob(r"D:\\VS2022\\%s\\VC\\Tools\\MSVC\\*\\bin\\Hostx64\\x64" % edition), reverse=True)
# 默认路径
# paths = sorted(glob.glob(r"C:\\Program Files (x86)\\Microsoft Visual Studio\\*\\%s\\VC\\Tools\\MSVC\\*\\bin\\Hostx64\\x64" % edition), reverse=True)
```

替换了c++17为c++14  
```
['/O2', '/std:c++17'] -> ['/O2', '/std:c++14']
```

torch库内  
```
# 修改
cflags = common_cflags + COMMON_MSVC_FLAGS + ['/std:c++14'] + extra_cflags
# cflags = common_cflags + COMMON_MSVC_FLAGS + ['/std:c++17'] + extra_cflags
```

# API
ws接口：`ws://127.0.0.1:10002`  
传参为json数据  

http接口(http_api.py)：`http://127.0.0.1:8800/audio_to_video?file_path=音频文件路径`  


# 虚拟人说话头生成(NeRF虚拟人实时驱动)--尽情打造自己的call annie吧
![](/img/example.gif)

xuniren windows安装教程：[一步步教学在 Windows 下面安装 pytorch3d 来部署 xuniren 这个项目 - 坤坤 - 博客园 (cnblogs.com)](https://www.cnblogs.com/dm521/p/17469967.html)

模型训练教程：[(278条消息) xuniren（Fay数字人开源社区项目）NeRF模型训练教程_郭泽斌之心的博客-CSDN博客](https://blog.csdn.net/aa84758481/article/details/131135823)



# Get Started

## Installation

Tested on Ubuntu 22.04, Pytorch 1.12 and CUDA 11.6，or  Pytorch 1.12 and CUDA 11.3

```python
git clone https://github.com/waityousea/xuniren.git
cd xuniren
```

### Install dependency

```python
# for ubuntu, portaudio is needed for pyaudio to work.
sudo apt install portaudio19-dev

pip install -r requirements.txt
or
## environment.yml中的pytorch使用的1.12和cuda 11.3
conda env create -f environment.yml 
## install pytorch3d
#ubuntu/mac
pip install "git+https://github.com/facebookresearch/pytorch3d.git"
```

**windows安装pytorch3d**

- gcc & g++ ≥ 4.9

在windows中，需要安装gcc编译器，可以根据需求自行安装，例如采用MinGW

以下安装步骤来自于[pytorch3d](https://github.com/facebookresearch/pytorch3d/blob/main/INSTALL.md)官方, 可以根据需求进行选择。

```python
conda create -n pytorch3d python=3.9
conda activate pytorch3d
conda install pytorch=1.13.0 torchvision pytorch-cuda=11.6 -c pytorch -c nvidia
conda install -c fvcore -c iopath -c conda-forge fvcore iopath
```

对于 CUB 构建时间依赖项，仅当您的 CUDA 早于 11.7 时才需要，如果您使用的是 conda，则可以继续

```
conda install -c bottler nvidiacub
```

```
# Demos and examples
conda install jupyter
pip install scikit-image matplotlib imageio plotly opencv-python

# Tests/Linting
pip install black usort flake8 flake8-bugbear flake8-comprehensions
```

任何必要的补丁后，你可以去“x64 Native Tools Command Prompt for VS 2019”编译安装

```
git clone https://github.com/facebookresearch/pytorch3d.git
cd pytorch3d
python setup.py install
```

### Build extension 

By default, we use [`load`](https://pytorch.org/docs/stable/cpp_extension.html#torch.utils.cpp_extension.load) to build the extension at runtime. However, this may be inconvenient sometimes. Therefore, we also provide the `setup.py` to build each extension:

```
# install all extension modules
# notice: 该模块必须安装。
# 在windows下，建议采用vs2019的x64 Native Tools Command Prompt for VS 2019命令窗口安装
bash scripts/install_ext.sh(建议复制出来单独安装)
```

### **start(独立运行)**

环境配置完成后，启动虚拟人生成器：

```python
python app.py
```
### **start（对接fay，在ubuntu 20.04及windows10下完成测试）**
环境配置完成后，启动fay对接脚本(无须启动app.py)
```python
python fay_connect.py
```
![](img/weplay.png)

扫码支助开源开发工作，凭支付单号入qq交流群



接口的输入与输出信息 [Websoket.md](https://github.com/waityousea/xuniren/blob/main/WebSocket.md)

虚拟人生成的核心文件

```python
## 注意，核心文件需要单独训练
.
├── data
│   ├── kf.json			
│   ├── pretrained
│   └── └── ngp_kg.pth

```

### Inference Speed

在台式机RTX A4000或笔记本RTX 3080ti的显卡（显存16G）上进行视频推理时，1s可以推理35~43帧，假如1s视频25帧，则1s可推理约1.5s视频。

# Acknowledgement

- The data pre-processing part is adapted from [AD-NeRF](https://github.com/YudongGuo/AD-NeRF).
- The NeRF framework is based on [torch-ngp](https://github.com/ashawkey/torch-ngp).
- The algorithm core come from  [RAD-NeRF](https://github.com/ashawkey/RAD-NeRF).
- Usage example [Fay](https://github.com/TheRamU/Fay).

学术交流可发邮件到邮箱：waityousea@126.com
