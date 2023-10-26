## 一次搞定观察者模式与订阅发布模式

### 定义：

观察者模式（observer pattern）是一种行为设计模式，在多个对象中定义一种一对多的依赖，这样当一个对象状态发生改变时，可以通知到该对象状态的依赖方并更新。

### 问题&解决方案

#### 问题：

假设我们现在要在前端页面绘制饼图，柱状图和线图，所需要的数据都在一个数据对象中，这些需要被绘制的图像与主题对象之间是没有联系的，而主题对象发生改变时，我们需要重新绘制表格图。我们可以在这些表格图中写一个轮询，每当30ms就查询主题对象是否发生改变，如果改变则进行重新绘制，这无疑是对资源的一种浪费，因为轮询会一直进行但数据可能在生命周期内都不会发生改变。我们也可以让主题对象直接进行通知，每次发生改变时，就通知到这三种图像，而由于其中依赖数据存在不同，可能会导致有些通知是不必要的。这时我们考虑使用订阅机制，建立起主题与被观察者之间的关系并在合适的时机去更新对应的图。

![dependency](/Users/linghankong/Desktop/BlogLHK/assets/dependency.jpg)

#### 解决方案：

通过上面的问题描述，我们要做的是建立依赖数据与观察者之间的关系，在数据发生改变时，通知到对应的观察者并更新。**将依赖数据和观察者对象代码解耦**， 同时使代码做到了复用与可维护性。

#### 设计的对象：

通常观察者模式会涉及到两个对象：

- Subject （主题）: 用来维护多个依赖对象，提供新增和删除某个依赖对象并且当状态发生改变时，通知依赖对象。

  ```typescript
  interface Subject {
  	subscribe: Function,
  	unSubscribe: Function,
  	Notify: Function
  }
  ```

  

- Observer(观察者): 表示对某些状态有依赖的对象。

### 订阅发布模式 & 观察者模式

引用《设计模式》这本书中在介绍 `observer pattern`的一句话， `This kind of interaction is also known as publish-subscribe` ，这句话翻译过来也就是这种两者之间关系也被称为发布者-订阅者。笔者认为，我们在面试中被问到这两个模式（订阅发布模式和观察者模式），其实本质上都是用来建立起主体与观察者之间的关系，处理两者通信的设计模式，而两者在具体实现上有着本质的区别。接下来就让我们一起来看一下二者的区别与具体实现：

##### 观察者模式（Observer Pattern）：

实现一个简单的观察者模式并不困难，网上也有很多例子， 我们首先实现一个Subject用来添加，删除和通知观察者：

```javascript
class Subject {
	constructor() {
		this.state = null; // 数据
		this.observers = [];
	}
	
	subscribe(func) {
		this.observers.push(func);
	}
	
	unSubscribe(func) {
		this.observers.filter(updateFunc => updateFunc !== func);
	}
	
	notify() {
        for (const ob of this.observers) {
            ob(this.state);
        }
	}

    setState(newState) {
        this.state = newState;
    }
}

const observableSubject = new Subject();

const observerOne = function(data) {
    console.log(`current data is: ${data}`);
}

const observerTwo = function(data) {
    console.log(`Date is ${Date.now()}, the data is ${data}`)
}

observableSubject.subscribe(observerOne);
observableSubject.subscribe(observerTwo);

observableSubject.setState(1); // 设置数据为 1

observableSubject.notify(); // 通知观察者

observableSubject.unSubscribe(observerTwo);
observableSubject.notify();
```

1. **主题-观察者的关系**：存在一个主题（Subject）和多个观察者（Observer）。主题维护一组观察者，提供新增和删除某个依赖对象并且当状态发生改变时，通知观察者。
2. **状态**：在观察者模式中，观察者主动注册，在状态发生改变时，主动接收通知
3. **耦合程度**：观察者通常会直接调用主题的方法来注册或取消注册，主题与观察者之间存在一定的耦合。



##### 订阅发布模式 （Publish-Subscribe Pattern）：

在实现原理上，订阅发布模式在会新增一个事件通道作为调度中心，管理事件的订阅与发布，解耦了上述主题和观察者的耦合关系。

![pubsub](/Users/linghankong/Desktop/BlogLHK/assets/pubsub.jpg)

```javascript
class PubSub {
    constructor() {
        this.events = {};
    }

    subscribe(topic, callback) {
        if (!this.events[topic]) {
            this.events[topic] = [];
        }
        this.events[topic].push(callback);
    }

    unSubscribe(topic, callback) {
        if (!this.events[topic]) {
            console.error('topic does not exist');
        }

        this.events[topic] = this.events[topic].filter((cb) => cb !== callback);
    }

    publish(topic, data) {
        if (this.events[topic]) {
            this.events[topic].forEach((callback => callback(data)));
        }
    }
}

const pubsub = new PubSub();
pubsub.subscribe('email/Login', (email) => {
    console.log("Subscriber 1, email login: ", email)
});

pubsub.subscribe('email/Login', (email) => {
    console.log("Subscriber 2, email login: ", email)
});

function loginEmail(email) {
    pubsub.publish('email/Login', email);
}

loginEmail('myEmail@123.com');
```

1. **发布者-订阅者关系**：在订阅发布模式中，有一个中心的事件调度器，用来当发布者，其他对象则是订阅者。发布者和订阅者之间是一对多的。
2. **事件的中心化管理**：事件的发布和订阅是中心化管理的，发布者将事件发送到事件调度器，然后由事件调度器将事件广播给所有订阅者, 详细可见 `publish` 函数。
3. **松散耦合**：对比观察者模式，订阅发布模式会减少对象之间的依赖性，使系统中的各个部分相对独立，从而实现了松散耦合。

**区别**：

1. **中心化 vs. 分散化**：
   - 订阅发布模式是中心化的，事件的发布和订阅由一个事件调度器管理。
   - 观察者模式是分散化的，每个观察者独立注册和接收通知，主题不直接参与通知的传递。
3. **耦合程度**：
   - 订阅发布模式通常实现了松散耦合，发布者和订阅者之间的依赖性较低。
   - 观察者需要了解主题的接口，并注册和取消注册，所以观察者模式主题与观察者对象间存在较高的耦合。

选择使用哪种模式取决于项目的需求和设计目标。订阅发布模式适用于需要将事件中心化管理的情况，而观察者模式适用于需要将通知分散到多个观察者的情况。



##### last words

**Redux**： 其实redux在核心（redux-core）也有 `subcribe` 函数通过观察者模式来订阅每次`store` 修改的情况, 具体应用价值则可以参考 `react-redux` 中的 `connect` 函数。这里需要主要的是，Redux的store更新并不是观察者模式方式而是单向数据流修改，希望大家不要弄混两个概念。

希望本文能对你学习有帮助，如果你认为有用的话，也麻烦一键三连，有任何问题也希望能够沟通讨论～
