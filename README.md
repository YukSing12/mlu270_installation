# mlu270_installation
Installation instructions of mlu270

Change current working directory to MLU270-v1.4.0-X86安装环境/270-v1.4.0/
```
cd MLU270-v1.4.0-X86安装环境/270-v1.4.0/
```

## **Install driver**
```
sudo apt install driver/neuware-mlu270-driver-dkms_4.2.0_all.deb
```
## **Load docker image**
```
# Load images
sudo docker load < mlu270-docker-images-x86/cambricon-test-mlu270-docker6.0.tar
# Check images
sudo docker images
```

## **Prepare environment**
### **Prepare models and datasets**
```
mkdir /opt/shared/
cp datasets/* /opt/shared/ 
cp models/* /opt/shared/ 
cd /opt/shared/
```
### **Extract .tar file**
```
cat MLU270_datasets.tar.gza* > MLU270_datasets.tar
tar -zxvf MLU270_datasets.tar.gz
tar -xvf Cambricon-MLU270-models-caffe.tar
tar -xvf Cambricon-MLU270-models-pytorch.tar
tar -zxvf Cambricon-MLU270-models-tensorflow.tar.gz
```

## **Run docker**
### **Publish a container's specific port(s) to the Docker host**
```
#/bin/bash
export MY_CONTAINER="Cambricon-MLU270-pytorch"
num=`sudo docker ps -a|grep "$MY_CONTAINER"|wc -l`
echo $num
echo $MY_CONTAINER
if [ 0 -eq $num ];then
sudo xhost local:root
sudo docker run -e DISPLAY=unix$DISPLAY --device /dev/cambricon_c10Dev* -v /tmp/.X11-unix:/tmp/.X11-unix -it -p 40000:22 --privileged --name $MY_CONTAINER \
-v $PWD/Cambricon-MLU270-pytorch/:/home/Cambricon-Test \
-v /opt/shared/models:/home/Cambricon-Test/models \
-v /opt/shared/datasets:/home/Cambricon-Test/datasets \
-v /opt/shared/neuware:/home/Cambricon-Test/neuware cambricon/test/ubuntu:v6.0 /bin/bash
else
sudo docker start $MY_CONTAINER
# sudo docker exec -ti $MY_CONTAINER /bin/bash
sudo docker exec $MY_CONTAINER /usr/sbin/sshd -D &
fi
```

### **Run docker**
```
bash ./run-cambricon-docker-pytorch.sh
```
### Set password
```
passwd
```
### **Install and start openssh-server**
```
apt-get update
apt-get install openssh-server
mkdir -p /var/run/sshd
/usr/sbin/sshd -D &
```
### **Check sshd**
```
apt-get install net-tools
netstat -apn | grep ssh
```
### Generate ssh key
```
ssh-keygen -t rsa
```

### **Permit Root Login **
```
vim /etc/ssh/sshd_config 
```
sshd_config:
```
line28: PermitRootLogin yes #prohibit-password
```

### **Restart ssh**
```
ps -aux | grep ssh
kill -9 pid_of_ssh
/usr/sbin/sshd -D &
```

## **Complie cambricon pytorch with python3.5**
### **Switch to python3.5**
```
update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2
```

### **Install cambricon sdk toolkits**
```
cd /home/Cambricon-Test/neuware
dpkg -i neuware-mlu270-1.4.0-1_Ubuntu16.04_amd64.deb
cd /var/neuware-mlu270-1.4.0/
dpkg -i *.deb
cat /usr/local/neuware/version.txt
```
Successfully install cambricon sdk toolkits if it show 'Neuware Version 1.4.0'.

### **Install requirements**
```
cd /home/Cambricon-Test/
apt update
apt install python3-pip
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple/ pybind11
```

### **Extract source file**
```
tar -zxf neuware/Cambricon-MLU270-pytorch.tar.gz -C /home/Cambricon-Test/
```

### **Modify shell script**
```
vim pytorch/src/catch/script/build_catch.sh
```
#### Commend line 130 to line 136 as:
```
# echo "====================create pip.conffile==============================="
# if [ ! -e ${HOME}/.pip/pip.conf ]; then
# mkdir -p ${HOME}/.pip
# echo "[global]" >> ${HOME}/.pip/pip.conf
# echo "index-url = http://mirrors.xxxxx.com/pypi/web/simple" >>${HOME}/.pip/pip.conf
# echo "trusted-host = mirrors.xxxxx.com" >> ${HOME}/.pip/pip.conf
# fi
```

#### Modify line 140 as:
```
virtualenv -p /usr/bin/python3.5 --no-site-packages ${PYTORCH_VENV}
```
**Note**: If your pip was broken with:
```
File "/usr/lib/python3.5/site-packages/pip/_internal/cli/main.py", line 60
    sys.stderr.write(f"ERROR: {exc}")
                                   ^
SyntaxError: invalid syntax
```
Add --no-setuptools --no-pip option to virtualenv in line 140 as:
```
virtualenv -p /usr/bin/python3.5 --no-site-packages --no-setuptools --no-pip ${PYTORCH_VENV}
```
And add command in build_catch.sh to install pip3 :
```
source ${PYTORCH_VENV}/bin/activate
wget https://bootstrap.pypa.io/pip/3.5/get-pip.py
python3 get-pip.py
```

#### Replace pip with pip3 in line 153, line 154, and line 164
```
pip3 install -r requirements.txt
pip3 install ninja
pip3 install dist/*.whl
```

### **Complie**
```
source env_pytorch.sh
./configure_pytorch.sh 0 0
```

**Note**: If you met error: **g++: internal compiler error: Killed (program cc1plus), it means g++ complier run out of memory.**

you should increase the memory of docker. Here we assign swap-memory in host and docker.

Create a swap memory of bs * count (4k * 8192000)
```
mkdir /swap
dd if=/dev/zero of=/swap/swapfile bs=4k count=8192000
mkswap /swap/swapfile
```

Run or stop swapfile by cammand:
```
swapon /swap/swapfile 
swapoff /swap/swapfile
```

If you want to keep swapfile on, you can add cammand in file /etc/fstab:
```
/swap/swapfile          swap               swap         defaults           0 0
```

### **Check**
```
cd pytorch/src/catch/examples/online
source ../../venv/pytorch/bin/activate
python
import torch
print(torch.__version__)
```
Successfully install cambricon sdk toolkits if it show '1.3.0a0'.
