# 一、行为树基本概念

## 1. BehaviorTree的介绍
BehaviorTree是由一系列具有层级结构的节点形成的树，用于控制task的执行流程。

### 1.1 基本概念

* tick信号发送到行为树的根节点，然后由根节点扩散传递到叶子节点。

* 行为树的任何一个节点接收到tick信号，会执行相应的callback。callback的返回值有如下三种：

  - SUCCESS
  - FAILURE
  - RUNNING

* RUNNING状态表示该action需要更多的时间来返回有效结果

* 假如一个TreeNode有1个或多个子节点的话，那么该节点将会负责传播tick。不同的TreeNode在向其子节点传递tick时可能会有不同的处理规则。

* LeafNodes通常用于实际执行命令（比如负责与系统的其他部分进行交互）。```Action```节点是最常见的LeafNodes

### 1.2 tick是如何工作的？
参看如下示意图：


![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/sequence_animation-4155a892772542caf81fa16c824c91f8.svg)

```Sequence```是最简单的控制节点： 按顺序执行孩子节点，假如所有孩子节点执行成功，其返回```SUCCESS```。

下面描述一下执行过程：

1） 第一个tick会将Sequence节点的状态设置为RUNNING

2) Sequence节点首先将tick传递到其第一个孩子节点```OpenDoor```，该孩子节点最终返回```SUCCESS```

3）之后会执行第二个孩子节点```Walk```以及第三个孩子节点```CloseDoor```

4) 一旦最后一个孩子节点执行完成，则整个Sequence节点的状态就会由```RUNNING```状态变为```SUCCESS```状态

### 1.3 结点类型

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/bt_node_type.png)

对于ActionNodes，我们可以进一步将其细化为```synchronous```与```asynchronous```节点。

* sychronous节点会一直阻塞到对应的callback返回```SUCCESS```或者```FAILURE```

* asynchronous节点则可能会返回```RUNING```状态，用于表示该任务当前仍然处于正在被执行状态。此种情况下，父节点将会继续tick该节点，直到最终返回SUCCESS或者FAILURE.

### 1.4 实例

为了更好的理解行为树，如下我们给出一些实例。为简单起见，当返回RUNNING状态时我们暂不考虑会发生哪些动作。

此外，这里我们假定每一个Action都是自动、sychronous的执行。

1） **第一个控制节点: Sequence**

如下我们演示最基本、最常用的控制节点```Sequence```节点的使用。

ControlNode的孩子节点都是具有顺序的。在下图的展示中，执行顺序是从左到右。

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/sequence_node.svg)

执行过程简述如下：

* 假如一个孩子节点返回SUCCESS，那么继续执行该控制节点的下一个孩子节点；

* 假如一个孩子节点返回FAILURE，那么将不再tick其他的孩子节点，然后整个Sequence节点返回FAILURE;

* 假如所有的孩子节点返回SUCCESS，则整个Sequence节点返回SUCCESS

2） **Decorators**

根据装饰节点的不同类型，该节点可实现如下一个目标：

* 对从child接收到的结果进行转化

* 暂停其孩子节点的执行

* 根据decorator的类型，重复tick孩子节点

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/DecoratorEnterRoom-f844716da2812873bdbb3f21448419c7.svg)

 
上图中```Inverter```节点是一个Decorators类型的节点，其用于翻转孩子节点返回的结果。```Invertor```的孩子节点是```IsDoorOpen```，因此其等价于：
<pre>
"Is the door closed"?
</pre>

```Retry```节点是另一个Decorators节点，假如其孩子节点返回FAILURE的话，其将会重复tick孩子，直到num_attempts次。

显然上图左分支的含义是：
<pre>
If the door is closed, then try to open it.
Try up to 5 times, otherwise give up and return FAILURE.
</pre>


