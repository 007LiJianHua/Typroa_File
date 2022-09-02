[toc]

getpots是Shell[命令行](https://so.csdn.net/so/search?q=命令行&spm=1001.2101.3001.7020)参数解析工具，旨在从Shell Script的命令行当中解析参数。getopts被Shell程序用来分析位置参数，option包含需要被识别的选项字符，**如果这里的字符后面跟着一个冒号，表明该字符选项需要一个参数，其参数需要以空格分隔**。冒号和问号不能被用作选项字符。getopts每次被调用时，它会将下一个选项字符放置到变量中，OPTARG则可以拿到参数值；如果option前面加冒号，则代表忽略错误；

**命令格式：**

```shell
getopts` `optstring name [arg...]
1
```

**命令描述：**
optstring列出了对应的Shell Script可以识别的所有参数。比如：如果 **Shell Script可以识别-a，-f以及-s参数，则optstring就是afs**；如果对应的参数后面还跟随一个值，则在相应的optstring后面加冒号。比如，**a:fs 表示a参数后面会有一个值出现，-a value的形式**。另外，getopts执行匹配到a的时候，会把value存放在一个叫**OPTARG的Shell Variable**当中。如果 optstring是以冒号开头的，命令行当中出现了optstring当中没有的参数将不会提示错误信息。

name表示的是参数的名称，每次执行getopts，会从命令行当中获取下一个参数，然后存放到name当中。如果获取到的参数不在optstring当中列出，则name的值被设置为?。命令行当中的所有参数都有一个index，第一个参数从1开始，依次类推。 另外有一个名为OPTIND的Shell Variable存放下一个要处理的参数的index。

```shell
#!/bin/bash
 
func() {
    echo "Usage:"
    echo "test.sh [-j S_DIR] [-m D_DIR]"
    echo "Description:"
    echo "S_DIR,the path of source."
    echo "D_DIR,the path of destination."
    exit -1
}
 
upload="false"
 
while getopts 'h:j:m:u' OPT; do
    case $OPT in
        j) S_DIR="$OPTARG";;
        m) D_DIR="$OPTARG";;
        u) upload="true";;
        h) func;;
        ?) func;;
    esac
done
 
echo $S_DIR
echo $D_DIR
echo $upload
```

实验1

```shell
sh test.sh -j /data/usw/web -m /opt/data/web
1
```

返回结果

```shell
/data/usw/web
/opt/data/web
false
123
```

实验2

```shell
sh test.sh -j /data/usw/web -m /opt/data/web -u
1
```

返回结果

```shell
/data/usw/web
/opt/data/web
true
123
```

实验3

```shell
sh test.sh j
1
```

返回结果

```shell
false
12
```

实验4

```shell
sh test.sh -h
1
```

返回结果

```shell
test.sh: option requires an argument -- h
Usage:
test.sh [-j S_DIR] [-m D_DIR]
Description:
S_DIR,the path of source.
D_DIR,the path of destination.
123456
```

实验5

```shell
sh test.sh -x
1
```

返回结果

```shell
test.sh: option requires an argument -- x
Usage:
test.sh [-j S_DIR] [-m D_DIR]
Description:
S_DIR,the path of source.
D_DIR,the path of destination.
123456
```

**shell深思考**

1.`getops`只接收带`-`的参数, **实验3**的`j`不带`-`,代码不走`getops`

2.`?)`接收两种情况，第一种情况getops接收的条件不存在，如实验5所示，第二种情况接收的参数后面没有，如实验4所示