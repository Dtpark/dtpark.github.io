---
layout: post
title: Fuzzing： AFL fuzzer 的 Docker API 封装
categories: develop
tags: fuzzing AFL API
---

## 一、背景介绍

最近课题组有个需求，要将目前组内的几个研究课题通过web的方式进行可视化展示。因为涉及模块众多，故决定将各个模块用docker+flask封装为API供web后端请求，这样方便部属、维护和扩展。本文以其中一个模块——AFL-fuzzer为例介绍模块的Docker封装。

![架构](https://i.loli.net/2021/06/06/dcpxrtPDVgYXAas.png)

## 二、AFL简介

### 2.1 概念

> **american fuzzy lop** (**AFL**) is a free software fuzzer that employs genetic algorithms in order to efficiently increase code coverage of the test cases. So far it helped in detection of significant software bugs in dozens of major free software projects, including X.Org Server, PHP, OpenSSL, pngcrush, bash, Firefox, BIND, Qt, and SQLite.
>
> The [source code](https://en.wikipedia.org/wiki/Source_code) of american fuzzy lop is published on GitHub. Its name is a reference to a breed of rabbit, the American Fuzzy Lop.
>
> ——《维基百科》

AFL是一个开源的软件模糊测试工具，它采用了遗传算法，来有效地提高测试案例的代码覆盖率。到目前为止，它帮助检测了几十个主要自由软件项目中的重大BUG，包括X.Org Server、PHP、OpenSSL、pngcrush、bash、Firefox、BIND、Qt和SQLite。

本次封装的AFL仅用到基础功能，即将编译好的插装二进制文件作为输入来启动fuzzing，实时显示fuzzing状态。

### 2.2 目录结构

```
AFL 执行的生成目录
├─in           					输入
│  ├─seed.txt           
├─out        						输出
│  ├─crashes            
│  ├─hangs           		
│  ├─queue           		
│  ├─fuzz_bitmap        
│  ├─fuzzer_status      
│  ├─plot_data          实时状态【unix_time, cycles_done, cur_path, paths_total, pending_total, pending_favs, map_size, unique_crashes, unique_hangs, max_depth, execs_per_sec】
├─plot-out        			
├─dictionaries        	字典
├─[file]        				输入的文件
```

通过编写shell脚本，来启动可以生成上述目录结构的AFL。

### 2.3 shell脚本

```shell
#! /bin/sh
# file1 is the target executable file(e.g. vuln),file2 is the dictionnary file(e.g. webp.dict)
file1=$2 
file2=$3
is_dict=0

# For everytime you run the fuzzer,it will create a new folder while its name is the time you run this script
time1=$1
# The floder name will also be write into the log.txt
# cat >> ./log.txt <<EOF
# $time1
# EOF

mkdir $time1
mv $file1 $time1
cd $time1
mkdir in out dictionaries plot-out

# create a simple seed file
touch seed.txt
cat >> ./seed.txt <<EOF
aaaaaa
EOF
mv seed.txt in

# if $3(the dict file)exists,it will run afl with "-x"
if [ -n "$3" ];then
	mv ../$file2 dictionaries
	is_dict=1
fi

if [ $is_dict -eq 1 ];then
	afl-fuzz -i ./in/ -o out -x dictionaries/$file2 ./$file1
elif [ $is_dict -eq 0 ];then
	afl-fuzz -i ./in/ -o out ./$file1
fi
```

在该shell所在路径下，用户只要输入 `./[shell脚本名] [任务id] [待测文件路径]`即可对待测文件进行fuzzing。

如，`./test.sh abc vuln1`

## 三、AFL的Docker封装

要将AFL封装为Docker API，逻辑上分为两步：

- 将AFL封装为Docker
- 在上一步Docker的基础上，将编写好的Flask一同封装

本部分介绍AFL的Docker封装。

### 3.1 分析依赖

因为Docker中的操作系统仅为基础镜像，很多依赖都需要手动安装，所以需要先梳理依赖。

AFL的版本选用为2.52b，运行其所需要的依赖包括：make、gcc、clang、llvm。

操作系统选用ubuntu 18.04。

### 3.2 编写Dockerfile

在解压好的AFL文件夹——`afl-2.52b`的同级目录新建Dockerfile文件：

```dockerfile
#afl-fuzz Dockerfile
FROM ubuntu:18.04

#换源
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

# 安装依赖
RUN apt-get update \
    && apt-get install -y make \
    && apt-get install -y gcc \
    && apt-get install -y clang \
    && apt-get install -y llvm

# afl搭建
COPY ./afl-2.52b/ /opt/afl-2.52b/
RUN make -C /opt/afl-2.52b&&make -C /opt/afl-2.52b/ install
#RUN echo core >/proc/sys/kernel/core_pattern
```

- FROM 引入基础镜像，本文以ubuntu 18.04作为基础镜像
- 换源：众所周知的原因，国内apt速度慢，换源为阿里云加速包下载速度
- 安装依赖：安装3.1中梳理的依赖【记得不要忘记输入 -y，否则在构建镜像过程中会中断退出】
- afl搭建：
  - 将本机的AFL复制到Docker中的`/opt/afl-2.52b/`目录下
  - 在Docker的`/opt/afl-2.52b`下编译AFL

### 3.3 构建镜像

1. 构建镜像

   在Dockerfile路径下输入命令`docker build -t [镜像名[:版本号]]`，构建镜像。如`docker build -t afl-fuzz:0.0.1`，构建一个名为afl-fuzz，版本号为0.0.1的镜像。

2.  查看镜像

   通过命令`docker images`查看镜像是否构建成功，成功会有如下效果：

   ```bash
   $ docker images
   REPOSITORY                TAG       IMAGE ID       CREATED       SIZE
   afl-fuzz                  0.0.1     06422a40a483   6 hours ago   859MB
   ```

3.  验证AFL镜像是否可用

   - 生成容器。通过命令`docker run -idt --name [容器名] [镜像名]`生成容器。如`docker run -idt --name afl-test afl-fuzz:0.0.1`以2中镜像为模板生成名为afl-test的容器。

   - 查看是否生成成功。通过命令`docker ps`可查看正在运行的容器

     ```bash
     $ docker ps                                         
     CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS       PORTS                    NAMES
     462ba5f2701d   afl-fuzz:0.0.1   "gunicorn main:app -…"   6 hours ago   Up 6 hours   0.0.0.0:8080->5901/tcp   ecstatic_thompson
     ```

   - 进入容器。通过命令`docker exec -it [容器id] /bin/sh`进入容器：

      ```bash
      docker exec -it 462ba5f2701d  /bin/sh
      # 
      ```

   - 测试AFL。通过命令`afl-fuzz`测试是否部署成功，出现下面效果就是部署成功。

      ```bash
      # afl-fuzz
      afl-fuzz 2.52b by <lcamtuf@google.com>
      
      afl-fuzz [ options ] -- /path/to/fuzzed_app [ ... ]
      
      Required parameters:
      
        -i dir        - input directory with test cases
        -o dir        - output directory for fuzzer findings
      
      Execution control settings:
      
        -f file       - location read by the fuzzed program (stdin)
        -t msec       - timeout for each run (auto-scaled, 50-1000 ms)
        -m megs       - memory limit for child process (50 MB)
        -Q            - use binary-only instrumentation (QEMU mode)
      
      Fuzzing behavior settings:
      
        -d            - quick & dirty mode (skips deterministic steps)
        -n            - fuzz without instrumentation (dumb mode)
        -x dir        - optional fuzzer dictionary (see README)
      
      Other stuff:
      
        -T text       - text banner to show on the screen
        -M / -S id    - distributed mode (see parallel_fuzzing.txt)
        -C            - crash exploration mode (the peruvian rabbit thing)
      
      For additional tips, please consult /usr/local/share/doc/afl/README.
      ```

   至此，AFL用Docker封装成功。

## 四、API实现

上一部分中，我们完成了AFL基础镜像的封装。要实现fuzzing模块的API功能，还需要web服务来调用AFL。笔者选择了python的flask框架来实现API功能。

> 本部分代码因为没有通用性，故注说明不会很详细，只有代码；
>
> 本部分及下部分在安装有AFL的虚拟机上开发。

### 4.1 启动脚本的实现

```python
# test.py
import os
import requests
import hashlib

def cmd(command):
    pid = os.popen("nohup "+command+" >test.txt 2>&1 & echo $!")
    return pid.read().strip()


# start fuzzing
def start(file_url):
    # get uuid
    hl = hashlib.md5()
    hl.update(file_url.encode(encoding="utf-8"))
    uuid = hl.hexdigest()

    # get file_name
    file_name = os.path.basename(file_url)

    # download the file
    result = down_load(file_url, file_name)

    if result['code'] != 20000:
        return result

    # chmod the file
    cmd("chmod -R 777 "+file_name)
    # start fuzzing
    pid = cmd("./test.sh " +uuid+" "+ file_name)
    return {
        'code': 20000,
        'message': 'start success',
        'data': {
            'pid': pid
        }
    }


# download the file
def down_load(file_url, file_name):
    try:
        # download the file
        file_stream = requests.get(file_url, stream=True)

        # download success
        if file_stream.status_code == 200:
            with open(file_name, 'wb') as f:
                content = file_stream.content
                f.write(content)
                res = {
                    'code': 20000,
                    'message': 'download success'
                }
        else:
            res = {
                'code': file_stream.status_code,
                'message': 'file download failed'
            }

    except Exception as e:
        res = {
            'code': 30000,
            'message': str(e)
        }

    return res
```

> AFL的启动API接收的待分析文件是腾讯云的文件链接，需要下载到本地再进行分析，uuid（任务id）约定为文件链接的md5值。
>
> 因目前只写了启动fuzzing的脚本，只贴出来启动的代码。启动成功后会返回进程id

### 4.2 API实现

```python
#main.py
from flask import Flask, request
import json

import test

app = Flask(__name__)


@app.route("/api/v1/start", methods=["GET"])
def start():
    try:
        file_url = request.args.get('file_url')
        # type = request.args.get('')
        # start fuzzing
        res = test.start(file_url)
    except Exception as e:
        res = {
            'code': 30000,
            'message': str(e)
        }

    return json.dumps(res)


if __name__ == '__main__':
    app.run(debug=True)
```

启动flask后，在浏览器测试成功，下部分介绍将flask封装为Docker。

## 五、Flask构建Gunicorn

> 参考文献 https://blog.csdn.net/qiulin_wu/article/details/105507785
>
> 本部分仅介绍实现步骤，原理详见上述链接

### 5.1 构建依赖文件

在flask根目录下新建文件`requirements.txt`，本项目仅用到flask、requests、gevent和gunicorn，故文件依赖内容如下：

```
flask
requests
gevent
gunicorn==18.0
```

> **由于本Docker的Python环境是2.7，所以在安装Gunicron的时候pip它默认会安装最新版本>=3.4，所以2.7环境使用指定版本18.0**

### 5.2 构建Gunicorn配置文件

Flask应用是一个符合WSGI规范的Python应用，它不能独立运行（类似run的方式只适合开发模式），需要依赖其他组件提供服务器功能。所以上面依赖包选择了Gunicorn+Gevent的超级组合，开始构建Gunicorn配置文件（下面workers的功能可以实际根据你的项目需求来定制化）。

在flask根目录新建文件`gunicorn.conf.py`，内容如下：

```python
workers = 5
worker_class = "gevent"
bind = "0.0.0.0:5901"
```

### 5.3 测试

使用Gunicorn提供的命令测试一下服务是否可以正确的运行

```bash
$ gunicorn main:app -c gunicorn.conf.py
```

在浏览器访问 `localhost:5901`，出现404页面，测试成功。

## 六、API的Docker封装

因为Docker构建的层数应尽量少，且笔者在开发时将第二部分中的AFL镜像作为基础镜像构建镜像时pip一直安装不上，故将二中的Dockerfile改写进行API部分的封装。

```dockerfile
#afl-fuzz Dockerfile
FROM ubuntu:18.04

#换源
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

# 安装依赖
RUN apt-get update \
    && apt-get install -y make \
    && apt-get install -y gcc \
    && apt-get install -y clang \
    && apt-get install -y llvm \
    && apt-get install -y python-pip \
    && pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple

# afl搭建
COPY ./afl-2.52b/ /opt/afl-2.52b/
#COPY ./test.sh /opt
#COPY ./vuln1 /opt
# WORKDIR /opt
#RUN chmod -R 777 ./test.sh
#RUN chmod -R 777 ./vuln1
RUN make -C /opt/afl-2.52b&&make -C /opt/afl-2.52b/ install
#RUN echo core >/proc/sys/kernel/core_pattern

# api构建
COPY ./api/py /usr/src/Project
RUN chmod -R 777 /usr/src/Project/test.sh \
    && pip install -r /usr/src/Project/requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
WORKDIR /usr/src/Project
CMD ["gunicorn", "main:app", "-c", "./gunicorn.conf.py"]

EXPOSE 5901
```

构建镜像、生成容器与二中相似，就不再赘述。

至此，afl-fuzz API封装完成。