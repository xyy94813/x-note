# 依赖注入

依赖注入（Dependency Injection，DI）是控制反转的一种具体实现方式。

依赖注入指的是将一个对象的依赖关系注入到该对象中，而不是由该对象自己创建或管理依赖对象。
依赖注入通过构造函数注入、属性注入或接口注入等方式实现，让对象之间的依赖关系更加灵活和可配置。

依赖注入适用于需要实现松耦合、可配置和可测试的对象之间的依赖关系管理。

## 代码示例

```js
class WheelHub {
  constructor(size, brand, material) {}
}

class Tire {
  constructor(size, brand, material) {}
}

class Wheel {
  constructor(size, wheelHubBrand, wheelHubMaterial, tireBrand, tireMaterial) {
    this.wheelHub = new WheelHub(size, wheelHubBrand, wheelHubMaterial);
    this.tire = new Tire(size, tireBrand, tireMaterial);
  }
}

// 使用依赖注入，将依赖实例注入
class DIWheel {
  constructor(wheelHub, tire) {
    if (wheelHub.size !== tire.size) {
      throw new Error('轮毂和轮胎尺寸不匹配');
    }
    this.wheelHub = wheelHub;
    this.tire = tire;
  }
}

// 对象之间的依赖关系更加灵活和可配置
const wheelHub1 = new WheelHub(19, '轮毂品牌1', '材料1');
const wheelHub2 = new WheelHub(19, '轮毂品牌2', '材料2');
const tire1 = new Tire(19, '车胎品牌', '光头胎');
const tire2 = new Tire(19, '车胎品牌', '雨胎');

// 灵活的基于组合创建了 4 轮胎实例
const wheel1 = new DIWheel(wheelHub1, tire1);
const wheel2 = new DIWheel(wheelHub1, tire2);
const wheel3 = new DIWheel(wheelHub2, tire1);
const wheel4 = new DIWheel(wheelHub2, tire2);
```

轮毂和车胎的尺寸需要保持一致。
所以，注入的实例对象要做一次校验，这就是相比统一管理时需要额外付出的成本。

## 延展

在 JS/Python 等动态语言下，可以注入一个类而不是实例。
仍由对象自己创建实例，但不进行依赖管理

```js
class WheelHub {
  constructor(size, brand, material) {}
}

class Tire {
  constructor(size, brand, material) {}
}

// 模版类
class ABrandWheelHub extends WheelHub {
  constructor(size) {
    super(size, '轮毂品牌1', '材料1');
  }
}

class ABRandTire extends Tire {
  constructor(size) {
    super(size, '车胎品牌', '雨胎');
  }
}

// 使用依赖注入，将依赖实例注入
class DIWheel {
  constructor(size, wheelHubClz, tireClz) {
    this.wheelHub = new wheelHubClz(size);
    this.tire = new tireClz(size);
  }
}

// JAVA 等静态语言，可以注入工厂实例达到类似目的
new DIWheel(19, ABrandWheelHub, ABRandTire);
```

大多数应用层服务器框架会，DI 配合装饰器语法糖一同使用

```js
@singleton
class APP {}

// 柯里化，装饰
@curry
const injectController = (path, controllerClz) => {
  const app = APP.getInstance();
  app.checkPath(path);

  const registryService = app.getServices();

  const depService = []; // 将注入的服务实例，基于某些规则筛选出来，然后注入进控制器中。

  // 将注册的服务实例注入进控制器
  app.addController(path, new controllerClz(...depService));
};

@singleton
@injectController('path1')
class ControllerA {}

@singleton
@injectController('path2')
class ControllerB {}
```
