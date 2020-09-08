

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

首次登陆

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

# 可以本地测试一下
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
# 此时 nvcc 命令可以调用
# nvidia-smi 命令也可以显示 GPU 信息

systemctl set-default graphical.target

# cuDNN加速 根据自己的需求安装
# anaconda 根据自己的需求安装
```

**NOTE**: 最后在 openstack 管理界面重启一下实例，重新登陆后一切正常。

