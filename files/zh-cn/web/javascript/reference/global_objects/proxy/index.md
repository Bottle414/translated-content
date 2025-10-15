---
title: Proxy
slug: Web/JavaScript/Reference/Global_Objects/Proxy
---

**Proxy** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义。

## 描述

`Proxy` 对象允许你创建用于替代原对象的对象，这可能会重定义基础的 `Object` 操作（比如获取、设置和属性定义）。代理对象通常用于记录属性访问、验证、格式化或清理输入等操作。

你可以通过两个参数创建一个 `Proxy`：

- `target`: 你想代理的原对象
- `handler`: 一个定义了哪些操作将被捕获，并且如何重定义被捕获的操作的对象。

例如，这段代码为 `target` 对象创建了一个代理。

```js
const target = {
  message1: "你好",
  message2: "所有人",
};

const handler1 = {};

const proxy1 = new Proxy(target, handler1);
```

由于处理器为空，因此此代理的行为与原始目标一样：

```js
console.log(proxy1.message1); // 你好
console.log(proxy1.message2); // 所有人
```

要自定义代理，我们在处理器对象定义函数：

```js
const target = {
  message1: "你好",
  message2: "所有人",
};

const handler2 = {
  get(target, prop, receiver) {
    return "世界";
  },
};

const proxy2 = new Proxy(target, handler2);
```

在这里，我们提供了一个 {{jsxref("Proxy/Proxy/get", "get()")}} 处理程序的实现，它会拦截对目标属性的访问尝试。

处理器函数有时被称为 _劫持_，大概是因为它们会拦截对目标对象的调用。上面 `handler2` 中的劫持重新定义了所有属性访问器：

```js
console.log(proxy2.message1); // 世界
console.log(proxy2.message2); // 世界
```

代理通常与 {{jsxref("Reflect")}} 对象一起使用，该对象提供了一些与 `Proxy` 陷阱同名的方法。`Reflect` 方法提供了调用对应 [对象内部方法](#对象内部方法) 的反射语义。例如，如果我们不希望重定义对象的行为，可以调用 `Reflect.get`：

```js
const target = {
  message1: "你好",
  message2: "所有人",
};

const handler3 = {
  get(target, prop, receiver) {
    if (prop === "message2") {
      return "世界";
    }
    return Reflect.get(...arguments);
  },
};

const proxy3 = new Proxy(target, handler3);

console.log(proxy3.message1); // 你好
console.log(proxy3.message2); // 世界
```

`Reflect` 方法仍然通过对象的内部方法与对象进行交互——如果在代理对象上调用它，它不会“去代理化”该代理。如果你在代理陷阱中使用 `Reflect` 方法，而 `Reflect` 方法调用再次被该陷阱拦截，可能会导致无限递归。

### 术语

谈论代理功能时使用以下术语。

- [handler](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy#处理器函数)
  - : 该对象作为 `Proxy` 构造函数的第二个参数传递。它包含重定义代理行为的劫持。
- trap
  - : 定义了相应[对象内部方法](#对象内部方法)的函数。（这类似于操作系统中的 _陷阱_ 概念）
- target
  - : 代理虚拟化的对象。它通常用作代理的存储后端。关于对象不可扩展性或不可配置属性的不变性（语义保持不变）会针对目标进行验证。
- {{Glossary("invariant", "invariants")}}
  - : 实现自定义操作时保持不变的语义。如果你的劫持实现违反了处理程序的不变性，将抛出 {{jsxref("TypeError")}}。

### 对象内部方法

[对象](/zh-CN/docs/Web/JavaScript/Guide/Data_structures#object)是属性的集合。然而，该语言并不提供任何机制来_直接_操作存储在对象中的数据——而是对象定义了一些内部方法，说明如何与其进行交互。例如，当你读取 `obj.x`，你可能会期望发生以下情况：

- `x` 属性沿着[原型链](/zh-CN/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)向上查找，直到被找到。
- 如果 `x` 是一个数据属性，则属性描述符的 `value` 属性将被返回。
- 如果 `x` 是一个访问器属性，则会调用 getter，并返回 getter 的返回值。

在这门语言中，这个过程没有什么特别之处——只是因为普通对象默认具有一个 `[[Get]]` 内部方法，而该方法定义了这种行为。`obj.x` 属性访问语法只是调用对象的 `[[Get]]` 方法，而对象使用其自身的内部方法实现来确定返回什么。

再举一个例子，[数组](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array) 与普通对象不同，因为它们具有一个神奇的 [`length`](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length) 属性，修改该属性时会自动分配空位或从数组中删除元素。同样，添加数组元素也会自动更改 `length` 属性。这是因为数组具有一个 `[[DefineOwnProperty]]` 内部方法，该方法知道在写入整数索引时更新 `length`，或在写入 `length` 时更新数组内容。这种内部方法实现与普通对象不同的对象被称为_特殊对象_。`Proxy` 使开发者能够完全自定义特殊对象。

所有对象都具有以下内部方法：

| 内部方法         | 对应的劫持                                                               |
| ----------------------- | -------------------------------------------------------------------------------- |
| `[[GetPrototypeOf]]`    | {{jsxref("Proxy/Proxy/getPrototypeOf", "getPrototypeOf()")}}                     |
| `[[SetPrototypeOf]]`    | {{jsxref("Proxy/Proxy/setPrototypeOf", "setPrototypeOf()")}}                     |
| `[[IsExtensible]]`      | {{jsxref("Proxy/Proxy/isExtensible", "isExtensible()")}}                         |
| `[[PreventExtensions]]` | {{jsxref("Proxy/Proxy/preventExtensions", "preventExtensions()")}}               |
| `[[GetOwnProperty]]`    | {{jsxref("Proxy/Proxy/getOwnPropertyDescriptor", "getOwnPropertyDescriptor()")}} |
| `[[DefineOwnProperty]]` | {{jsxref("Proxy/Proxy/defineProperty", "defineProperty()")}}                     |
| `[[HasProperty]]`       | {{jsxref("Proxy/Proxy/has", "has()")}}                                           |
| `[[Get]]`               | {{jsxref("Proxy/Proxy/get", "get()")}}                                           |
| `[[Set]]`               | {{jsxref("Proxy/Proxy/set", "set()")}}                                           |
| `[[Delete]]`            | {{jsxref("Proxy/Proxy/deleteProperty", "deleteProperty()")}}                     |
| `[[OwnPropertyKeys]]`   | {{jsxref("Proxy/Proxy/ownKeys", "ownKeys()")}}                                   |

函数对象还具有以下内部方法：

| 内部方法 | 对应的劫持                                 |
| --------------- | -------------------------------------------------- |
| `[[Call]]`      | {{jsxref("Proxy/Proxy/apply", "apply()")}}         |
| `[[Construct]]` | {{jsxref("Proxy/Proxy/construct", "construct()")}} |

重要的是要意识到，与对象的所有交互最终都归结为调用这些内部方法之一，并且它们都可以通过代理进行自定义。这意味着该语言几乎没有保证任何行为（除了某些关键的不变量）——一切都由对象本身定义。当你运行 [`delete obj.x`](/zh-CN/docs/Web/JavaScript/Reference/Operators/delete) 时，并不能保证 [`"x" in obj`](/zh-CN/docs/Web/JavaScript/Reference/Operators/in) 之后会返回 `false`——这取决于对象对 `[[Delete]]` 和 `[[HasProperty]]` 的实现。`delete obj.x` 可能会将内容记录到控制台、修改某些全局状态，甚至定义新属性而不是删除现有属性，尽管这些语义应该在你的代码中避免。

所有内部方法均由语言本身调用，JavaScript 代码无法直接访问。{{jsxref("Reflect")}} 命名空间提供的方法除了调用内部方法外，还进行了一些输入规范化/验证。在每个劫持的页面中，我们列出了调用劫持的几种典型情况，但这些内部方法在很多地方都会被调用。例如，数组方法通过这些内部方法读取和写入数组，因此像 [`push()`](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push) 这样的方法也会调用 `get()` 和 `set()` 劫持。

大多数内部方法的功能都很简单。只有两个可能容易混淆的是 `[[Set]]` 和 `[[DefineOwnProperty]]`。对于普通对象，前者会调用 setter 方法；而后者则不会。（如果不存在现有属性或该属性是数据属性，`[[Set]]` 会在内部调用 `[[DefineOwnProperty]]`。）虽然你可能知道 `obj.x = 1` 语法使用了 `[[Set]]`，而 {{jsxref("Object.defineProperty()")}} 使用了 `[[DefineOwnProperty]]`，但其他内置方法和语法使用的语义并不明显。例如，[类字段](/zh-CN/docs/Web/JavaScript/Reference/Classes/Public_class_fields) 使用 `[[DefineOwnProperty]]` 语义，这就是为什么在派生类上声明字段时不会调用超类中定义的 setter。

## 构造函数

- {{jsxref("Proxy/Proxy", "Proxy()")}}
  - : 创建一个新的 `Proxy` 对象。

> [!NOTE]
> 没有 `Proxy.prototype` 属性，因此 `Proxy` 实例没有任何特殊的属性或方法。

## 静态方法

- {{jsxref("Proxy.revocable()")}}
  - : 创建一个可撤销的 `Proxy` 对象。

## 示例

### 基础示例

在这个例子中，当对象中不存在该属性名称时，数字 `37` 会作为默认值返回。它使用了 {{jsxref("Proxy/Proxy/get", "get()")}} 处理程序。

```js
const handler = {
  get(obj, prop) {
    return prop in obj ? obj[prop] : 37;
  },
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b); // 1, undefined

console.log("c" in p, p.c); // false, 37
```

### 无操作转发代理

在以下例子中，我们使用了一个原生 JavaScript 对象，代理会将所有应用到它的操作转发到这个对象上。

```js
const target = {};
const p = new Proxy(target, {});

p.a = 37; // 操作转发到目标

console.log(target.a); // 37 (操作已经被正确地转发!)
```

### 不支持私有字段转发

A proxy is still another object with a different identity — it's a _proxy_ that operates between the wrapped object and the outside. As such, the proxy does not have direct access to the original object's [private elements](/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements).
代理仍然是另一个具有不同身份的对象——它是一个在被包装对象和外部之间进行操作的代理。因此，代理无法直接访问原始对象的[私有元素](/zh-CN/docs/Web/JavaScript/Reference/Classes/Private_elements)。

```js
class Secret {
  #secret;
  constructor(secret) {
    this.#secret = secret;
  }
  get secret() {
    return this.#secret.replace(/\d+/, "[REDACTED]");
  }
}

const secret = new Secret("123456");
console.log(secret.secret); // [REDACTED]
// 看起来像是无操作转发...
const proxy = new Proxy(secret, {});
console.log(proxy.secret); // TypeError: Cannot read private member #secret from an object whose class did not declare it
```

这是因为当代理的 `get` 陷阱被调用时， `this` 的值是 `proxy`，而不是原始的 `secret`，因此 `#secret` 无法访问。要解决这个问题，请将原始的 `secret` 替换为 `this`：

```js
const proxy = new Proxy(secret, {
  get(target, prop, receiver) {
    // 默认情况下，它看起来像 Reflect.get(target, prop, receiver)
    // w它具有不同的 `this` 值
    return target[prop];
  },
});
console.log(proxy.secret);
```

对于方法，这意味着你还必须将方法的 `this` 值重定向到原始对象：

```js
class Secret {
  #x = 1;
  x() {
    return this.#x;
  }
}

const secret = new Secret();
const proxy = new Proxy(secret, {
  get(target, prop, receiver) {
    const value = target[prop];
    if (value instanceof Function) {
      return function (...args) {
        return value.apply(this === receiver ? target : this, args);
      };
    }
    return value;
  },
});
console.log(proxy.x());
```

一些原生 JavaScript 对象具有称为 _[内部槽](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-object-internal-methods-and-internal-slots)_ 的属性，这些属性无法通过 JavaScript 代码访问。例如，[`Map`](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map) 对象有一个名为 `[[MapData]]` 的内部槽，用于存储 Map 的键值对。因此，你无法轻松地为 Map 创建转发代理：

```js
const proxy = new Proxy(new Map(), {});
console.log(proxy.size); // TypeError: get size method called on incompatible Proxy
```

你必须使用上面说明的 “`this`-recovering” 代理来解决这个问题。

### 验证

通过 `Proxy`，你可以轻松校验传给对象的值。这个示例使用了 {{jsxref("Proxy/Proxy/set", "set()")}} 处理器。

```js
const validator = {
  set(obj, prop, value) {
    if (prop === "age") {
      if (!Number.isInteger(value)) {
        throw new TypeError("年龄不是整数");
      }
      if (value > 200) {
        throw new RangeError("无效年龄");
      }
    }

    // The default behavior to store the value
    // 存储该值的默认行为
    obj[prop] = value;

    // Indicate success
    // 表示成功
    return true;
  },
};

const person = new Proxy({}, validator);

person.age = 100;
console.log(person.age); // 100
person.age = "young"; // 抛出异常
person.age = 300; // 抛出异常
```

### 操作 DOM 节点

在这个例子中，我们使用 `Proxy` 来切换两个不同元素的属性：因此，当我们在一个元素上设置该属性时，另一个元素上的属性会被取消。

我们创建了一个 `view` 对象，它是具有 `selected` 属性对象的代理。代理处理器定义了 {{jsxref("Proxy/Proxy/set", "set()")}} 处理器。

当我们将一个 HTML 元素分配给 `view.selected` 时，该元素的 `'aria-selected'` 属性会被设置为 `true`。如果我们随后将另一个元素分配给 `view.selected`，这个元素的 `'aria-selected'` 属性会被设置为 `true`，而之前的元素的 `'aria-selected'` 属性会自动被设置为 `false`。

```js
const view = new Proxy(
  {
    selected: null,
  },
  {
    set(obj, prop, newVal) {
      const oldVal = obj[prop];

      if (prop === "selected") {
        if (oldVal) {
          oldVal.setAttribute("aria-selected", "false");
        }
        if (newVal) {
          newVal.setAttribute("aria-selected", "true");
        }
      }

      // 存储该值的默认行为
      obj[prop] = newVal;

      // 表示成功
      return true;
    },
  },
);

const item1 = document.getElementById("item-1");
const item2 = document.getElementById("item-2");

// 选择 item1:
view.selected = item1;

console.log(`item1: ${item1.getAttribute("aria-selected")}`);
// item1: true

// 选择 item2 会取消选择 item1:
view.selected = item2;

console.log(`item1: ${item1.getAttribute("aria-selected")}`);
// item1: false

console.log(`item2: ${item2.getAttribute("aria-selected")}`);
// item2: true
```

### 数值修正和额外属性

`products` 代理对象会评估传入的值，并在需要时将其转换为数组。该对象还支持一个作为 getter 和 setter 的额外的属性 `latestBrowser`。

```js
const products = new Proxy(
  {
    browsers: ["Firefox", "Chrome"],
  },
  {
    get(obj, prop) {
      // 额外属性
      if (prop === "latestBrowser") {
        return obj.browsers[obj.browsers.length - 1];
      }

      // 返回该值的默认行为
      return obj[prop];
    },
    set(obj, prop, value) {
      // 额外属性
      if (prop === "latestBrowser") {
        obj.browsers.push(value);
        return true;
      }

      // 如果该值不是数组，则进行转换
      if (typeof value === "string") {
        value = [value];
      }

      // 存储该值的默认行为
      obj[prop] = value;

      // 表示成功
      return true;
    },
  },
);

console.log(products.browsers);
//  ['Firefox', 'Chrome']

products.browsers = "Safari";
//  传递一个字符串（错误地）

console.log(products.browsers);
//  ['Safari'] <- 没问题，该值是一个数组

products.latestBrowser = "Edge";

console.log(products.browsers);
//  ['Safari', 'Edge']

console.log(products.latestBrowser);
//  'Edge'
```

## 规范

{{Specifications}}

## 浏览器兼容性

{{Compat}}

## 参见

- [Proxies are awesome](https://youtu.be/sClk6aB_CPk) presentation by Brendan Eich at JSConf (2014)
