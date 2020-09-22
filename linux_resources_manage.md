---
title: Linux 服务器资源管理
created: '2020-09-15T04:52:07.794Z'
modified: '2020-09-15T06:33:01.252Z'
---

# Linux 服务器资源管理
## 一. 查看系统资源占用信息
### <1> 一般信息
#### 1.1 top:
```shell
top
```
+ 进入`top`后, 按下 `1` 可以查看每个 CPU 核心的占用情况, `us`代表用户占用, `sy`代表系统占用, 再按 `1` 可以退出多核心视图
+ 进入`top`后, 按下 大写`P`可以 以 CPU 占用排序, 按下 大写`M`可以 以内存占用排序 
+ 进入`top`后, 按下 `c` 可以查看命令的完整信息
+ 进入`top`后, 按下 `n`, 再输入数字可以只显示 top N 的进程信息
+ 进入`top`后, 按下 `f` 可以对监控的列信息进行过滤/筛选, (具体按`方向键`选择列, 按`空格`选择是否显示, 按`q`退出), 这在显示完整命令信息的时候很有用(否则命令会显示不全)
#### 1.2 glances
> 安装: `pip install glances`

```shell
glances
```
#### 1.3 htop
> 安装: `sudo yum install htop`

```shell
htop
```
#### 1.4 lscpu
```shell
lscpu
```

### <2> GPU 信息
#### 2.1 nvidia-smi
```shell
watch nvidia-smi
```
#### 2.2 gpustat
> 安装: `pip install gpustat`
```shell
gpustat -u -p -i 1
```

## 二. 进程/程序管理

### <1> 查看进程 PID 的具体信息
```shell
ll /proc/进程PID
```
### <2> 强制杀死进程
```shell
kill -9 进程PID
```
### <3> 限制程序的 GPU 占用

```shell
# eg: 限制程序只能访问 0,1,2 三个卡
# 在启动命令时:
CUDA_VISIBLE_DEVICES=0,1,2 命令
```

### <4> 限制程序的 CPU 占用
#### 1. 使用 taskset 绑定进程到固定的 CPU 核心
> 这是限制 CPU 占用的最有效方法

##### eg1: 执行命令 Command,同时限制其只使用 1,2,3,4 核心
```shell
taskset -a -c 1,2,3,4 Command
```
> 注意 1. : `-a` 参数可以限制进程和其启动的所有线程

> 注意 2. : 在限制程序前,最好使用 `top`命令或者`htop`查看现在不同 CPU 核心的占用情况, 尽可能放在占用少的核心上

> 注意 3. : 和 GPU 一样, CPU 核心也是从 0 开始计数

#### eg2: 限制正在执行的命令,只允许其占用 1,2,3,4 核心
首先找到命令的 PID (top 命令)
```shell
taskset -a -c 1,2,3,4 -p 程序PID
```
#### 2. 使用 `cpulimit` 限制进程的 CPU 占用百分比
> 安装: `sudo yum install cpulimit`
```shell
cpulimit --pid 进程PID --limit 百分比
```

##### eg1. 限制命令 x(pid:233) 最多使用 20%的 CPU 资源
```shell
cpulimit --pid 233 --limit 20
```
> 注意 1: 这实际上在约束 CPU 的占用时间, Ref: [linux - How to limit the CPU usage of process? - Stack Overflow](https://stackoverflow.com/questions/41238417/how-to-limit-the-cpu-usage-of-process)

### <5> 设置程序的"优先级"(`Nice Value`, `NI`)
当我们使用 `top` 命令时, 注意到会有 `PR` (`Priority`) 和 `NI` (`Nice Value`) 两列, 这两个都影响了程序的调度优先级, 这两个值的关系一般如下:
```
PR = 20 + NI
```
(Ref: https://askubuntu.com/a/656787)

因此我们可以通过设置 NI 值来影响程序的优先级.

在 Linux 中, NI 值的范围为 [-20,+19], 其中, -20 为优先级最高, +19 的优先级最低, 默认为 0

可以使用 `nice` 和 `renice` 命令 设置程序的优先级
#### eg1: 启动命令 Command, 并设置优先级(NI)为 10
```shell
nice -n 10 Command
```

#### eg2. 调整正在执行的程序 x 的优先级为 10
首先获取程序的 PID
```shell
renice -n 10 -p 程序PID
```
> renice 还可以通过 -u 或者 -g 限制用户或者用户组


