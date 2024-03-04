# BehaviorTree的使用

本章介绍一下BehaviorTree CPP的安装及使用。主要参考如下：

1. [BehaviorTree GitHub](https://github.com/BehaviorTree/BehaviorTree.CPP/tree/master)

2. [BehaviorTree官网文档](https://www.behaviortree.dev/docs/tutorial-basics/tutorial_01_first_tree)


## 1. 源代码的编译

这里我们下载的版本是```4.5.1```:

1) 下载源代码

<pre>
# wget https://github.com/BehaviorTree/BehaviorTree.CPP/archive/refs/tags/4.5.1.tar.gz
# tar -zxvf 4.5.1.tar.gz
</pre>

2）安装编译工具conan

<pre>
# yum install python3 python3-pip
# pip3 install conan
# conan profile detect --force

# cat ~/.conan2/profiles/default 
[settings]
arch=x86_64
build_type=Release
compiler=gcc
compiler.cppstd=gnu98
compiler.libcxx=libstdc++
compiler.version=4.8
os=Linux
</pre>
> ps: 参看[conan - c++包管理器](https://zhuanlan.zhihu.com/p/627358880)

上面需要将conan默认配置cppstd改为```gnu11```

3) 编译BehaviorTree.CPP
<pre>
# mkdir build
# ls
4.5.1.tar.gz  BehaviorTree.CPP-4.5.1  build

# cd build
# conan install ../BehaviorTree.CPP --output-folder=. --build=missing
# cmake ../BehaviorTree.CPP -DCMAKE_TOOLCHAIN_FILE="conan_toolchain.cmake"
# cmake --build . --parallel
</pre>



