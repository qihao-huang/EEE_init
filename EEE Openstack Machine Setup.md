

# EEE Openstack Machine Setup

[TOC]

## 申请实例

登陆 http://portal.eeeos/ 后

### 创建实例

第一步：点击创建实例

![Screenshot 2020-09-08 at 15.34.29](Screenshot%202020-09-08%20at%2015.34.29.png)

第二步：在弹出的窗口中输入实例名称，任意均可。

![Screenshot 2020-09-08 at 15.35.26](Screenshot%202020-09-08%20at%2015.35.26.png)



第三步：上一个页面中的 “下一步” ，输入分配的空间大小。其中 “删除实例时删除卷” 默认为 ”不“，建议选 ”是”。最后，在下方的可用配额中选择 Ubuntu 的箭头，选择后，Ubuntu 会显示已分配。

![Screenshot 2020-09-08 at 15.36.09](Screenshot%202020-09-08%20at%2015.36.09.png)

第四步：根据需求选择实例配置，点击向上的箭头。

![Screenshot 2020-09-08 at 15.40.23](Screenshot%202020-09-08%20at%2015.40.23.png)



第五步：直接跳到 “密钥对”，如果在 openstack 主页中已经创建了密钥对，那么直接在可用配额中选择即可。因为有些是个人的密钥，建议创建一个项目的密钥对。

![Screenshot 2020-09-08 at 15.41.39](Screenshot%202020-09-08%20at%2015.41.39.png)

第六步：点击 “创建密钥对”，再点击 ”把私钥复制到剪贴板“。把私钥保存到本地文件。

![Screenshot 2020-09-08 at 15.43.29](Screenshot%202020-09-08%20at%2015.43.29.png)

## ![Screenshot 2020-09-08 at 15.45.42](Screenshot%202020-09-08%20at%2015.45.42.png)

第七步：点击创建实例，等待若干分钟。如果状态显示为 ”运行“，即分配实例成功，拷贝其 IP 地址。

 ![Screenshot 2020-09-08 at 15.47.13](Screenshot%202020-09-08%20at%2015.47.13.png)

## 环境配置

### 首次登陆

```shell
chmod 700 ~/.ssh
cd ~/.ssh
chmod 600 private_key_name

ssh -i private_key_name ubuntu@192.168.xxx.xxx
```

### 设置密码

```shell
sudo su
sudo passwd ubuntu
# 设置机器密码，注意此时 ubuntu 是 admin 权限
exit
```

### 开启 ssh 密码登陆

```shell
# sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_bak 
# 可以选择备份一下
sudo vim /etc/ssh/sshd_config
# change "PasswordAuthentication no" to "PasswordAuthentication yes"

# 退出 vim 后
systemctl restart sshd

# 之后再次登陆，输入 IP 和 密码即可，不再需要 private_key
# 注意保管好 private_key。public_key 可以在 openstack 页面查看
ssh ubuntu@192.168.xxx.xxx
```

### GPU 设置

```shell
sudo apt-get update
sudo apt-get install -y gcc
sudo apt-get install -y build-essential

cd ~

# 这里以 CUDA 10.2 为例
curl -O "http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run"

sudo chmod +x ~/cuda_10.2.89_440.33.01_linux.run

sudo tee /etc/modprobe.d/nouveau.conf <<!
blacklist nouveau
options nouveau modeset=0
!

sudo mkdir /opt/NVIDIA

sudo ~/cuda_10.2.89_440.33.01_linux.run --silent --driver --toolkit --toolkitpath=/opt/NVIDIA/cuda-10.2

# 在 ~/.bashrc 中添加
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

# 此时 nvcc 命令可以调用
# nvidia-smi 命令也可以显示 GPU 信息

systemctl set-default graphical.target
```

### 查看设备

```shell
# 查看 硬盘空间
df -hl

# 查看 CPU
cat /proc/cpuinfo

# 查看 内存
free -m
```

### cuDNN

```shell
# 从 Nvidia 下载 CUDA 对应版本的 cuDNN
tar -zxvf cudnn-10.2-linux-x64-v8.0.3.33.tgz
# sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/include/* /usr/local/cuda/include

sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
cd /usr/local/cuda/lib64/

ls -lha libcudnn*

# 创建软链接
sudo rm -rf libcudnn.so libcudnn.so.8
sudo ln -s libcudnn.so.8.0.3 libcudnn.so.8
sudo ln -s libcudnn.so.8 libcudnn.so

sudo rm -rf libcudnn_adv_infer.so libcudnn_adv_infer.so.8
sudo ln -s libcudnn_adv_infer.so.8.0.3 libcudnn_adv_infer.so.8
sudo ln -s libcudnn_adv_infer.so.8 libcudnn_adv_infer.so

sudo rm -rf libcudnn_adv_train.so libcudnn_adv_train.so.8
sudo ln -s libcudnn_adv_train.so.8.0.3 libcudnn_adv_train.so.8
sudo ln -s libcudnn_adv_train.so.8 libcudnn_adv_train.so

sudo rm -rf libcudnn_cnn_infer.so libcudnn_cnn_infer.so.8
sudo ln -s libcudnn_cnn_infer.so.8.0.3 libcudnn_cnn_infer.so.8
sudo ln -s libcudnn_cnn_infer.so.8 libcudnn_cnn_infer.so

sudo rm -rf libcudnn_cnn_train.so libcudnn_cnn_train.so.8
sudo ln -s libcudnn_cnn_train.so.8.0.3 libcudnn_cnn_train.so.8
sudo ln -s libcudnn_cnn_train.so.8 libcudnn_cnn_train.so

sudo rm -rf libcudnn_ops_infer.so libcudnn_ops_infer.so.8
sudo ln -s libcudnn_ops_infer.so.8.0.3 libcudnn_ops_infer.so.8
sudo ln -s libcudnn_ops_infer.so.8 libcudnn_ops_infer.so

sudo rm -rf libcudnn_ops_train.so libcudnn_ops_train.so.8
sudo ln -s libcudnn_ops_train.so.8.0.3 libcudnn_ops_train.so.8
sudo ln -s libcudnn_ops_train.so.8 libcudnn_ops_train.so

ls -lha libcudnn*
sudo ldconfig

# 从 Nvidia 下载下面三个文件，和 cuDNN 文件一起下载
sudo dpkg -i libcudnn8_8.0.3.33-1+cuda10.2_amd64.deb
sudo dpkg -i libcudnn8-dev_8.0.3.33-1+cuda10.2_amd64.deb
sudo dpkg -i libcudnn8-samples_8.0.3.33-1+cuda10.2_amd64.deb

# 测试 cuDNN 安装
cp -r /usr/src/cudnn_samples_v8/ $HOME
cd ~/cudnn_samples_v8/mnistCUDNN
make clean && make
./mnistCUDNN
# 显示 Test passed 即成功
```

### Anaconda

```shell
./Anaconda3-2020.07-Linux-x86_64.sh
# 在 ~/.bashrc 中添加
export PATH=/home/ubuntu/anaconda3/bin:$PATH
```

**NOTE**: 最后在 openstack 管理界面重启一下实例，重新登陆后一切正常。

