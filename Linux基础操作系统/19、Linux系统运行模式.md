# Linux系统运行模式

[toc]

## 一、运行模式

### 1、运行模式

> * 0
>   * 关机模式
> * 1
>   * 单用户模式（重置root密码）
> * 2
>   * 字符模式（无网络连接，网卡不能使用）
> * **3**
>   * **完整的字符模式**						//红色为常见的
> * 4
>   * 空白模式（预留出来）
> * **5**
>   * **图形化模式**
> * 6
>   * 重启模式

### 2、查看系统当前运行模式

```bash
[root@localhost ~]# runlevel 		
N 5
```

### 3、切换模式

```bash
//前提是系统必须已经安装了该系统
[root@localhost ~]# init 3 
[root@localhost ~]# init 5		
```

* Centos 7：

切换系统运行模式：

```bash
[root@hosta ~]# systemctl isolate graphical.target 
[root@hosta ~]# systemctl isolate multi-user.target 
```

设置系统默认启动模式：

```bash
[root@hosta ~]# systemctl set-default multi-user.target 
```

## 二、救援模式修复磁盘挂载

1. 先将虚拟机电源设置为“打开电源时自动进入固件”

![img](https://note.youdao.com/yws/public/resource/2afc09e9f36e6d71c131f2f4bcbf5d7d/xmlnote/WEBRESOURCE0c64764ae8748558bf228ce501ae316e/43)

2. 进入光盘的BIOS界面

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/B7C06BF6E5744D188BF1F6FCE103F477/E470FEEA0B634C18BDF4B4B21D89E873/25244)

3. 选择Troubleshoot

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/B7C06BF6E5744D188BF1F6FCE103F477/AFDE9C4FC5C9405E9CE34C3BA445EC60/25246)

4. 选择Rescue a Centos system

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/B7C06BF6E5744D188BF1F6FCE103F477/5985B72AEC53485085CC1E881E233708/25249)

5. ![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/B7C06BF6E5744D188BF1F6FCE103F477/7AF0033BA46C46EA968A40FD42B09747/25251)

6. 进入Rescue Mount，输入相应命令即可进入救援模式

![img](https://note.youdao.com/yws/public/resource/bb531089b1bbce74a52011e49c623597/xmlnote/B7C06BF6E5744D188BF1F6FCE103F477/01064F48B6DF4263BEC9EF47B4AF3325/25253)

7. 救援操作结束后退出
8. 再次进入BIOS界面设置为从硬盘启动