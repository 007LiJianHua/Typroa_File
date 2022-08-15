# 划分VLAN

# <kbd>听课</kbd><kbd>理解</kbd><kbd>操作</kbd><kbd>记笔记</kbd>

[Typroa使用技巧网址]([https://blog.csdn.net/qq_41261251/article/details/102817673?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163938647016780265468606%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163938647016780265468606&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-102817673.first_rank_v2_pc_rank_v29&utm_term=typora%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B&spm=1018.2226.3001.4187](https://blog.csdn.net/qq_41261251/article/details/102817673?ops_request_misc=%7B%22request%5Fid%22%3A%22163938647016780265468606%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=163938647016780265468606&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-102817673.first_rank_v2_pc_rank_v29&utm_term=typora使用教程&spm=1018.2226.3001.4187))

## 一、交换机模式：

1、每个模式中有各自的命令，可以通过Tab键将命令补全，或者通过？键查看帮助

2、不全原则：该命令在当前模式下唯一时，按Tab键可补齐，或者回车

3、如果出错报错请按：ctrl+shift+6

4、修改交换机名字：hostname SW1



SW1>								用户模式，用于基础命令的查询

SW1>enable					进入特权模式

SW1#								用于简单配置和大多数的模式查询

SW1# configure terminal     	进入全局配置模式命令

SW1(config)#						全局模式，用于配置交换机或路由器的主要功能

SW1(config)# exit    			退出该模式，回到上模式

SW1(config)# end/ctrl+C    直接退到特权模式

## 二、配置VLAN：

### 	1、创建VLAN：

> 方法一：
> sw1>enable 
> sw1#conf terminal 
> sw1(config)#vlan 10      创建vlan10
> sw1(config-vlan)#vlan 20
> sw1(config-vlan)#vlan 30

> 方法二：
> sw1#vlan database   进入vlan数据库设置
> sw1(vlan)#vlan 40     创建vlan40
> sw1(vlan)#vlan 40 name haha   修改描述
> sw1(vlan)#exit   退出vlan数据库配置

### 	2、查看VLAN：

> sw1(config-vlan)#end
> sw1#show vlan
>
> sw1(config-vlan)#do show vlan

### 	3、删除VLAN：

> sw1(config)#no vlan 500  删除vlan500
> sw1(config)#exit 退出当前模式
> sw1#show vlan  查看vlan
>
> sw1#conf terminal 
> sw1(config)#no vlan 400 
> sw1(config)#do  show vlan     do命令可以不用退回特权模式并查询

### 4、将VLAN接口加入/划分到VLAN里：

> sw1(config)#<font color="red">interface fastEthernet 0/1</font>   			进入f0/1接口
> sw1(config-if)#switchport mode access   			将接口模式设置为access，交换机连接pc机时，一般都设置为access，只允许一个vlan通过
> sw1(config-if)#switchport access vlan 10   将该接口加入至vlan10中
>
> sw1(config-if)#do show vlan
>

# 三、实验拓扑

| VLAN 10 | PC8  | PC9  |
| ------- | ---- | ---- |
| VLAN 20 | PC7  | PC10 |

![划分VLAN](D:\桌面\12月末北京培训\锦程长期班.md笔记\.md图片\划分VLAN.png)







