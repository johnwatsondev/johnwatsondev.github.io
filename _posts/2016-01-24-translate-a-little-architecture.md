---
title: (译) 关于架构的一点想法
layout: post
guid: urn:uuid:4e4878de-1757-44bc-91b8-f00e4366d9b6
comments: true
tags:
  - architecture
---

原文：[A Little Architecture](http://blog.cleancoder.com/uncle-bob/2016/01/04/ALittleArchitecture.html)  
译者：[JohnWatsonDev](http://johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

我想成为一名软件架构师。
> 作为一名年轻软件开发者，这是很好的目标。:)

我想领导一个团队，作一些重要的决定，比如，用什么数据库，框架和 web 服务器之类的事情。
> 噢。好吧。你终究还是不想做一名软件架构师。

我当然想！我想成为作出所有重要决定的人。
> 那很好，但你没有指出重要的决定，而是一些无关的事情。

什么意思？采用什么数据库不是重要的决定？你知道我们在它上面花了多少钱吗？
> 也许很多。那个当然不是重要的；采用何种数据库不是最重要的决定之一。

你怎么能这么说？数据库可是系统的核心！它包含了所有有条理、可操作、可排序和可索引的数据，没有它就没有系统！
> 数据库只不过是 IO 设备。它不过是提供了一些可排序，可查询的有用工具而已。但它们对于系统架构是无关紧要的。

无关紧要的？这太疯狂了。
> 是的，无关紧要的。系统的业务逻辑可能会利用这些工具；但是它们并不是业务逻辑内在需要的。如果有必要，你可以把它们替换成别的工具；业务逻辑还是没有变化的。

好吧。但是由于它们使用了原先数据库中的工具，我不得不重新编写它们。（译者注：它们指业务逻辑代码）
> 嗯，这就是你的问题所在。

什么意思？
> 你的问题是，你认为业务逻辑依赖于数据库的工具。其实并不是这样。或者说，如果你提供了好的架构，业务逻辑不应该依赖它们。

胡扯。我该如何创建业务逻辑，而且不使用它们必须使用的工具呢？
> 我不是说它们不需要数据库的工具；而是不应该依赖它们。*业务逻辑不应该知道你用了何种特定的数据库。*

你如何在不知道使用什么工具的情况下，让业务逻辑使用这些工具呢？
> **依赖反转**。你可以让数据库依赖业务逻辑。请确保业务逻辑不依赖数据库。

简直胡扯。
> 相反，我在谈论软件架构的事情。这是**依赖反转**原则。*低等级别策略应该依赖高等级别策略。*

更是胡扯！高等级别策略（我假设是你说的业务逻辑）调用低等级别策略（我假定是数据库）。所以高等级别策略依赖低等级别策略，也就是调用者依赖被调用者。所有人都知道这个！
> 在运行阶段，这是对的。但在编译阶段，我们想让依赖反转。高等级别策略的代码不应该包含低等级别策略的代码。

噢，拜托！你不能调用不依赖的代码。
> 你当然可以。那就是面向对象的全部。

面向对象是创建现实世界的模型，把数据和功能封装到对象中，代码组织成直观的结构。
> 谁告诉你的？

每个人都知道。这明显是对的。
> 毫无疑问。但是，利用面向对象原则确实可以调用不依赖的代码。

好吧。如何做呢？
> 你晓得在面向对象中，对象可以互相发消息吗？

当然。
> 消息发送者不知晓接收者类型，这个也知道吧。

这要分语言。在 Java 中，发送者起码知道接收者的基本类型；在 Ruby 中，发送者至少知道接收者能够处理接收到的消息。
> 对。在其中任何一种情况下，消息发送者不知道接收者的确切类型。

是的，当然。
> 因此，消息发送者可以调用接收者中的一个函数，而并不需要知道接收者的具体类型。

对。我知道了。但是消息发送者依然依赖接收者呀。
> 在运行时是这样。但不是在编译时期。发送者的代码中没有提到，也没有依赖接收者的代码。实际上，接收者的代码依赖发送者的代码。

发送者仍然依赖接收者类。
> 也许用代码可以解释清楚。我将用 Java 写一下。首先 *sender* 包：

```java
package sender;

public class Sender {
  private Receiver receiver;

  public Sender(Receiver r) {
    receiver = r;
  }

  public void doSomething() {
    receiver.receiveThis();
  }

  public interface Receiver {
    void receiveThis();
  }
}
```

> 接着 *receiver* 包：

```java
package receiver;

import sender.Sender;

public class SpecificReceiver implements Sender.Receiver {
  public void receiveThis() {
    //do something interesting.
  }
}
```

> 看到 *receiver* 包依赖 *sender* 包了吧。*SpecificReceiver* 还依赖 *Sender* 。而且 *sender* 包中没有依赖任何 *receiver* 包中的东西。

嗯，但是你作弊了。你把接收者的接口放在了发送者类中。
> 你才开始理解，孩子。

理解什么？
> 当然是架构的原则。发送者拥有接收者必须实现的接口。

是不是我必须用嵌套类声明？
> 嵌套类只是实现目标的一种方式而已。还有别的。

嗯，等等。这和数据库有什么关系呢？回到我们开始的话题。
> 让我们看看更多的代码吧。首先是一个简单的业务逻辑：

```java
package businessRules;

import entities.Something;

public class BusinessRule {
  private BusinessRuleGateway gateway;

  public BusinessRule(BusinessRuleGateway gateway) {
    this.gateway = gateway;
  }

  public void execute(String id) {
    gateway.startTransaction();
    Something thing = gateway.getSomething(id);
    thing.makeChanges();
    gateway.saveSomething(thing);
    gateway.endTransaction();
  }
}
```

业务逻辑不够全面。
> 这只是个列子。你可以有许多类似这样的类，实现不同的业务逻辑。

好，`Gateway` 是啥意思？
> 它提供业务逻辑需要的所有数据处理方法。实现如下：

```java
package businessRules;

import entities.Something;

public interface BusinessRuleGateway {
  Something getSomething(String id);
  void startTransaction();
  void saveSomething(Something thing);
  void endTransaction();
}
```

> 注意它在 *businessRules* 包下。

好吧。*Something* 类是干嘛用的？
> 它代表一个简单的业务对象。我把它放在 *entities* 包中。

```java
package entities;

public class Something {
  public void makeChanges() {
    //...
  }
}
```

> 最后是 *BusinessRuleGateway* 的实现。这个类才知道具体的数据库：

```java
package database;

import businessRules.BusinessRuleGateway;
import entities.Something;

public class MySqlBusinessRuleGateway implements BusinessRuleGateway {
  public Something getSomething(String id) {
    // use MySql to get a thing.
  }

  public void startTransaction() {
    // start MySql transaction
  }

  public void saveSomething(Something thing) {
    // save thing in MySql
  }

  public void endTransaction() {
    // end MySql transaction
  }
}
```

> 请再次注意，业务逻辑在运行时调用了数据库；但是在编译时期，*database* 包依赖了 *businessRules* 包。

嗯，我想我明白了。你在业务逻辑中使用多态隐藏了数据库的实现细节。但是，你仍然有一个接口为业务逻辑提供所有的数据库工具呀。
> 不，完全不是。我们并没有尝试提供给业务逻辑所有的数据库工具。而是让业务逻辑通过接口只创建自己需要的方法。接口的实现者可以调用合适的工具。

嗯，如果所有的业务逻辑需要所有的工具，那你是否必须把所有的工具放在 *gateway* 中。
> 呃，你还是有点不明白。

明白什么？我好像很清楚了。
> 每个业务逻辑通过接口定义它仅仅需要的数据操作方法。

等等。什么？
> 这叫接口分离原则。每个业务逻辑仅仅使用数据库的一部分工具。所以，每个业务逻辑提供的接口仅仅能访问相应的一些方法。

但是，这意味着将有许多接口，还有许多实现类调用其他数据库类。
> 呃，不错。你已经开始理解了。

但这会弄的乱七八糟，而且浪费时间！我为什么要这样？
> 你那样做将会使代码很清晰，而且节约时间。

拜托。那仅仅让代码变多了而已。
> 相反的，这些就是最重要的架构决定，它们允许你推迟无关紧要的决定。

啥意思？
> 还记得开始，你想成为软件架构师吗？你想作出真正重要的决定？

是，那就是我要的。
> 在那些决定中，你想决定用何种数据库，何种服务器和框架。

是，而且你说那些不是重要的决定。它们是无关的。
> 对，它们是无关的。软件架构师最重要的决定就是让你**不必**决定采用何种数据库、服务器和框架。

但你必须首先作出这些决定！
> 不，的确不需要。你应该在开发周期的后面（当拥有更多信息时）作这些决定。

> 当架构师过早地决定采用何种数据库，之后发现采用文件就足够时，这是悲哀的。

> 当架构师过早地决定采用何种服务器，之后发现整个团队确实只采用 Socket 就足够时，这是悲哀的。

> 当架构师过早地让团队强制使用某个框架，之后发现很多功能用不到，反而带来许多束缚时，这是悲哀的。

> 当团队中的架构师推迟那些无关紧要的决定，而在后期有足够多信息再决定时，这是幸福的。

> 当团队中的架构师把访问很慢，消耗资源的 IO 设备和框架同业务隔离开来，反而可以创建快速、轻量的测试环境时，这是幸福的。

> 当团队中的架构师能决定真正重要的事情，推迟不重要的决定时，这是幸福的。

胡说，我一点都不明白。
> 好吧，假如你现在还没有进行管理，也许将在十年内或更久之后才明白。
