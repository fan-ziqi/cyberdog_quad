# cyberdog_quad

小米铁蛋quad，[quad官方wiki](https://github.com/robomechanics/quad-sdk/wiki/)

## 更新日志

- [x] 调试环境
- [ ] 接入Cyberdog

## 部署

运行环境：Ubuntu-20.04+ROS-Noetic（注意：在此环境下需要拉取noetic_devel分支）

### 安装Ipopt与HSL

下载Ipopt

```bash
# 下载Ipopt并解压
wget https://www.coin-or.org/download/source/Ipopt/Ipopt-3.12.7.zip 
unzip Ipopt-3.12.7.zip 
# 配置
cd Ipopt-3.12.7
./configure
```

去这里[HSL for IPOPT](http://www.hsl.rl.ac.uk/ipopt/)注册并下载coinhsl，放到`Ipopt-3.12.7/Thirdparty/HSL`中

```bash
# 进入HSL文件夹并解压coinhsl
cd Ipopt-3.12.7/ThirdParty/HSL
unzip coinhsl-archive-xxx.zip
# 创建软连接，名称必须为coinhsl，或者把文件夹名直接改成coinhsl
ln -s ./coinhsl-archive-xxx/ coinhsl
# 配置
sudo chmod +x ./configure
./configure --enable-loadable-library
# 编译
make
sudo make install
```

编译Ipopt

```bash
# 先拷贝一个头文件，否则编译出错
cd Ipopt-3.12.7/ThirdParty/HSL/include/coin/ThirdParty
sudo cp -r * /usr/local/include/
# 编译
make
make install
```

复制lib与include

```bash
cd Ipopt-3.12.7/lib
sudo cp -r * /usr/local/lib/ 
cd Ipopt-3.12.7/include
sudo cp -r * /usr/local/include/ 
sudo ldconfig
```

### 编译代码

```bash
mkdir -p quad_ws/src && cd quad_ws/src
# 如果您使用的是Ubuntu20.04和ROS-Noetic，记得拉取noetic_devel分支，官方仓库有个地方写错了，建议拉我的
git clone -b noetic_devel https://github.com/fan-ziqi/quad-sdk.git
# 将coinhsl拷贝过来
cp -r /xxx/Ipopt-3.12.7/ThirdParty/HSL/coinhsl-archive-xxx ./quad-sdk/external/ipopt/coinhsl
# 脚本是Windows编写的，需要删除其中的\r字符
sed -i 's/\r//' quad-sdk/setup.sh quad-sdk/external/setup_deps.sh quad-sdk/quad_simulator/setup_deps.sh
# 给权限运行脚本，请仔细检查脚本的输出，有报错都要解决。
sudo chmod +x quad-sdk/setup.sh 
./quad-sdk/setup.sh 
# 需要拷贝头文件，否则编译会报错
sudo cp -r /usr/local/include/coin/* /usr/local/include
# 安装额外需要的一个依赖
sudo apt-get install libgfortran4
# 编译
cd ..
catkin build
# 设置环境变量
source ./devel/setup.bash
```

## 测试

开启一个终端（不要忘记source环境变量）

```bash
roslaunch quad_utils quad_gazebo.launch
```

其中，`world`为可选参数，比如`world:=rough_25cm`可更改地图。

新开一个终端，发送消息让机器人站立，站立后ctrl+c退出

```bash
rostopic pub /robot_1/control/mode std_msgs/UInt8 "data: 1"
```

运行规划器

```bash
roslaunch quad_utils quad_plan.launch reference:=twist logging:=true
```

新开一个终端，打开遥控器节点：

```bash
rosrun teleop_twist_keyboard teleop_twist_keyboard.py cmd_vel:=/robot_1/cmd_vel
```

全局路点规划，运行后机器人会自动前进到目标位置，并依据地形在线规划全局轨迹和未来多个周期落足与GRF

```bash
roslaunch quad_utils quad_plan.launch reference:=gbpl logging:=true
```

## 其他

MIT 和 unitree的腿编号：
1   0
3   2

quad_sdk的腿编号：
0   2
1   3

## 参考文章

[超越MIT Mini-Cheetah 的四足机器人开源项目（Quad-SDK）](https://zhuanlan.zhihu.com/p/522218252)