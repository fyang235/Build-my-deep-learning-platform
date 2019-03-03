# Build a Deep Learning Platform with Nvidia RTX2060
### RTX2060 + Ubuntu16.04 + Nvidia415.27 + CUDA10.0 + cuDNN7.4 + TensorFlow1.13.1
Everybody knows computation capability is crucial for deep learning.
However not every students can aford the NVIDIA graphic card like **RTX 1080 TI (\$699)**.
Fortunantly, NVIDIA released its new product **RTX 2060 (\$349)** a few month ago,
which shows a high price/performance ratio and is suitable for students.

Not sure about whether RTX2060 is capable of deep learning computation? 
[Here](https://3g.163.com/news/article_cambrian/E8FQBI3V0511DSSR.html?c) gives you the answer:  

# Procedures:
1. Install ubuntu 16.04  
2. Install graphic card dirver
3. install and cuda10 + cudnn7.4
4. Install tensorflow-gpu1.13.1
5. Configure jupyter notebook


## 1. Install ubuntu 16.04  
### Download  ubuntu16.04  
You can downloead ubuntu16.04 [here](http://releases.ubuntu.com/?_ga=2.185751660.475113973.1551573743-1171051096.1551006501.)   
The latest 18.04 version is not recommend for its not mature enough and few resources are availble.  
### Creat bootable USB
Creat a bootable USB with the [POWERISO](https://www.poweriso.com/download.php) software.   
### Install and Update
Then install ubuntu 16.04. after that remember to connect to the internet update your software in Software Updater. For me I use PPPOE and I also started my SSH service (skip this step if you don’t need)     

### Connect Internet and set SSH (skip if not needed)
To Configure pppeo
```shell
sudo pppoeconf  
```
enter username and password. Select yes for the rest.
To configure ssh  
```shell
sudo apt-get install openssh-server openssh-client  
sudo vi /etc/ssh/sshd_config
```
set `PermitRootLogin` to `no`  
start it
```shell 
sudo service ssh restart
```
## 2. Install graphic card dirver 
Before installation, make sure your gcc version is 5.4.0. 
### Shut down the default nouveau driver
Then we need to shut down the default nouveau driver.  
Check the status of nouveau:  
```shell
lsmod | grep nouveau
```

press **ctrl+F1** to switch to console mode and add nouveau to blacklist:  
```shell
sudo vi /etc/modprobe.d/blacklist.conf
```

append `blacklist nouveau`  line to the end  
make it work
```shell
sudo update-initramfs –u
```  
a `/boot/initrd.img-4.15.0-45-generic` file will be generated 
and then we reboot and get into the console mode, retype 
```shell
lsmod | grep nouveau
```
make sure nothing appears.


### Stop lightdm service
Now we need to stop the lightdm service.     
Check the service 
```shell
service --status-all | grep lightdm
```    
stop it
```shell
sudo service lightmd stop
```  
check again, make sure the lightdm is shutdown
```shell
service --status-all| grep lightdm
```


### install RTX2060 driver
now it time to get started with our RTX2060 driver.
**The most important principle to keep in mind is version matching. We need the exactly versions in the title.**
It can be problematic if you use a driver downloaded from the Nvidia web. They will offer you the least version,  which work with the latest version of cuda and no tensorflow version can appropriately work with them. For me, I got nvidia-418 and it requires cuda 10.1 and I did find a suitable version of tensorflow which work with that. What I would recommend is use the nvidia driver repository.  
```shell
sudo add-apt-repository ppa:graphics-drivers/ppa  
sudo apt update  
sudo apt install nvidia-415
```  
the **nvidia-415.27** driver will be installed, test it with `nvidia-smi`, you can see a table if it works. then we reboot.  
Go to the Ubuntu __Software&updates__ window checkout the __Additional Divers__ tab and make sure you are using the nvidia binary driver-415.27.




## 3. install and cuda10 + cudnn7.4
### Download
Download [cuda10.0](https://developer.nvidia.com/cuda-toolkit-archive) runfile   
and [cudnn7.4 runtime lib and dev lib](https://developer.nvidia.com/rdp/cudnn-archive)  
Then you have what you need:  
```shell
cuda_10.0.130_410.48_linux.run  
libcudnn7_7.4.2.24-1+cuda10.0_amd64.deb  
libcudnn7-dev_7.4.2.24-1+cuda10.0_amd64.deb
```
get into the console mode and stop the lightdm service again. 
### Install CUDA
Make all the cuda package executable   
```shell
sudo chmod +x cuda_10.0.130_410.48_linux.run 
sudo chmod +x libcudnn7_7.4.2.24-1+cuda10.0_amd64.deb  
sudo chmod +x libcudnn7-dev_7.4.2.24-1+cuda10.0_amd64.deb
```  
install cuda
```shell
sudo ./cuda_10.0.130_410.48_linux.run 
```   
choose no when asked if you want to install nvidia accelerated driver. And yes for the rest.  
After finishing, start lightdm service
```shell
sudo service lightmd start
```  
add cuda path to the `.bashrc` file. For me it looks like this  
```shell
export PATH=$PATH:/usr/local/cuda-10.0/bin  
export LD_LIBRARY_HOME=$LD_LIBRARY_HOME:/usr/local/cuda-10.0/lib64
```  
and add the cuda lib to the dynamic lib  
```shell
sudo vi /etc/ld.so.conf
```
append
`/usr/local/cuda-10.0/lib64`  
Make it work
```shell
sudo ldconfig
```
### Install cuDNN
then we install cudnn  
```shell
sudo dpkg -i libcudnn7_7.4.2.24-1+cuda10.0_amd64.deb  
sudo dpkg -i libcudnn7-dev_7.4.2.24-1+cuda10.0_amd64.deb  
type nvcc --version to check
```  
## 4. Install tensorflow-gpu1.13.1
### Install anaconda
Download and install  
```shell
sudo wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.0.1-Linux-x86_64.sh   
sudo bash Anaconda3-5.0.1-Linux-x86_64.sh
```
append anaconda path to PATH. It varies for specific PC, for me it’s like  
```shell
vi ./bashrc  
export PATH=$PATH:/home/yang/anaconda3/binxport LD_LIBRARY_HOME=$LD_LIBRARY_HOME:/homeyang/anaconda3/lib  
export PATH=$PATH:/home/yang/anaconda3/binxport LD_LIBRARY_HOME=$LD_LIBRARY_HOME:/homeyang/anaconda3/lib
```
### Build conda environment
```shell
conda create -n py3 python=3  
source activate py3
```  
then install tensorflow 1.13.1 from https://pypi.tuna.tsinghua.edu.cn/simple for its speedy, we choose this tensorflow version because lower versions work with lower versions of cuda.
```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple  tensorflow-gpu==1.13.1
```  
### Test
after installation test it with some cnn program if you get error like   

    UnknownError: Failed to get convolution algorithm. This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above.     
    [[{{node conv1/convolution}}]]     
    [[{{node mul_49}}]]  
Add these lines prior your code  
```python
from tensorflow.compat.v1 import ConfigProto  
from tensorflow.compat.v1 import InteractiveSession  
config = ConfigProto()  
config.gpu_options.allow_growth = True  
session = InteractiveSession(config=config) 
```
try again, it works for me.
## 5. Configure jupyter notebook
Install jupyter notebook
```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple  jupyter
```  
set remote access for jupyter notebook 
type  `ipython` input  
```python
from notebook.auth import passwd  
passwd()
```  
input your password and copy the password token.  
Exit ipython type  
```shell 
jupyter notebook --generate-config
```  
to generate `/home/yang/.jupyter/jupyter_notebook_config.py`. open it, add the following and save.  
```shell
c.NotebookApp.ip = '*'  
c.NotebookApp.open_browser = False  
c.NotebookApp.password='your-password-token'  
```
now open jupyter notebook in terminal and enter **http://your-server-ip:8888/** in your browser  
Enjoy!






