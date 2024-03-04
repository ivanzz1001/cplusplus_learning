# BehaviorTree的介绍
BehaviorTree是由一系列具有层级结构的节点形成的树，用于控制task的执行流程。



## 1. 基本概念

* tick信号发送到行为树的根节点，然后由根节点扩散传递到叶子节点。

* 行为树的任何一个节点接收到tick信号，会执行相应的callback。callback的返回值有如下三种：

  - SUCCESS
  - FAILURE
  - RUNNING

* RUNNING状态表示该action需要更多的时间来返回有效结果

* 假如一个TreeNode有1个或多个子节点的话，那么该节点将会负责传播tick。不同的TreeNode在向其子节点传递tick时可能会有不同的处理规则。

* LeafNodes通常用于实际执行命令（比如负责与系统的其他部分进行交互）。```Action```节点是最常见的LeafNodes



## 2. tick是如何工作的？
参看如下示意图：


![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/sequence_animation-4155a892772542caf81fa16c824c91f8.svg)

```Sequence```是最简单的控制节点： 按顺序执行孩子节点，假如所有孩子节点执行成功，其返回```SUCCESS```。

下面描述一下执行过程：

1） 第一个tick会将Sequence节点的状态设置为RUNNING

2) Sequence节点首先将tick传递到其第一个孩子节点```OpenDoor```，该孩子节点最终返回```SUCCESS```

3）之后会执行第二个孩子节点```Walk```以及第三个孩子节点```CloseDoor```

4) 一旦最后一个孩子节点执行完成，则整个Sequence节点的状态就会由```RUNNING```状态变为```SUCCESS```状态

## 3. 结点类型

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/bt_node_type.png)

对于ActionNodes，我们可以进一步将其细化为```synchronous```与```asynchronous```节点。

* sychronous节点会一直阻塞到对应的callback返回```SUCCESS```或者```FAILURE```

* asynchronous节点则可能会返回```RUNING```状态，用于表示该任务当前仍然处于正在被执行状态。此种情况下，父节点将会继续tick该节点，直到最终返回SUCCESS或者FAILURE.

## 4. 实例

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


3）**第二个控制节点：Fallback**

Fallback节点也被称为```selectors```，主要用于实现回退策略。比如，当其一个孩子节点返回Failure，其就会自动tick到下一个孩子节点。

Fallback节点tick孩子节点的顺序如下：

* 假如一个child返回FAILURE，那么其就会tick下一个孩子节点；

* 假如一个孩子接单返回SUCCESS，那么Fallback节点将不会tick下一个孩子节点，并直接返回SUCCESS。

* 假如所有的孩子节点返回FAILURE，那么Fallback节点返回FAILURE

通过如下的示例，你可以看到如何将Sequences及Fallbacks两种控制节点结合起来：

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/FallbackBasic-4d0eb7fe32bdcd1f2ee3d46cd27dc19f.svg)


4）**重看'Fetch me a beer'**

对于上面的```Fetch me a beer```示例，当冰箱中并没有啤酒时，冰箱门仍然会保持打开状态。我们可以对这一问题进行修复。

下图中使用```绿色```来指示对应的节点返回SUCCESS，使用```红色```来指示对应的节点返回FAILUE，```黑色```表示对应的节点并未执行。

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/fetch_me_beer1.svg)

下面我们创建另外一棵树来关闭冰箱门，并且使得能够处理GrabBeer为FAILURE的情形：

![sequence-animation](https://raw.githubusercontent.com/ivanzz1001/cplusplus_learning/main/BehaviorTree/image/FetchBeer-24723226326a39f057f9d9c32250eb95.svg)

上面左右两棵树都能最后将冰箱门关闭，但是：

* 对于左边这棵树，不管冰箱里是否有啤酒，其都会返回SUCCESS

* 对于右边这棵树，假如冰箱中有啤酒的话，将返回SUCCESS，否则返回FAILURE



# 行为树的XML格式

在后面的第一个教程中我们会使用到如下xml:
```
 <root BTCPP_format="4">
     <BehaviorTree ID="MainTree">
        <Sequence name="root_sequence">
            <SaySomething   name="action_hello" message="Hello"/>
            <OpenGripper    name="open_gripper"/>
            <ApproachObject name="approach_object"/>
            <CloseGripper   name="close_gripper"/>
        </Sequence>
     </BehaviorTree>
 </root>
```
你会注意到：

* 行为树的第一个tag是<root>，其可以包含有一个或多个<BehaviorTree>标签

* <BehaviorTree>标签需要有[ID]属性

* <root>标签需要包含有[BTCPP_format]属性

* 每一个TreeNode节点有一个单独的tag来表示。

	- tag的名称由ID指定，该名称用于在factory中注册对应的TreeNode

    - 属性name是可选项

    - ports通过属性来配置。在上面的例子中，SaySomething需要输入port(message)

* 对于一棵BehaviorTree的孩子节点个数：

    - ControlNodes可以包含1~N个孩子节点

    - DecoratorNodes及其子树只能有一个孩子节点

    - ActionNodes及ConditionNodes不能有孩子节点

