# typescript 变量声明

* let 和 const 是JavaScript里相对较新的变量声明方式。let在很多方面与var是相似的，但是可以帮助大家避免在JavaScript里常见一些问题。 const是对let的一个增强，它能阻止对一个变量再次赋值。
* 因为TypeScript是JavaScript的超集,所以它本身就支持let和const。

## 为什么推荐let代替var

### var声明
* var 的作用域：可以在包含它的函数，模块，命名空间或全局作用域内部任何位置被访问；多次声明同一个变量并不会报错，在代码审查时漏掉，引发无穷的麻烦。
* var 捕获变量问题。

### let声明
* let的作用域：块作用域
* let的重定义及屏蔽：
    * 在相同作用域里，变量只能被定义1次；
    * 在一个嵌套作用域里引入一个新名字的行为称做屏蔽；两层嵌套的for变量都可以定义为i，内层i屏蔽了外层的i；尽量避免使用屏蔽，因为可读性差；

### const声明 
* 它拥有与 let相同的作用域规则，但是不能对它们重新赋值。
* 实际上const变量的内部状态是可修改的。 幸运的是，TypeScript允许你将对象的成员设置成只读的。 接口一章有详细说明。

## 解构
* 解构数组
    * 数组解构很简单
    ```
    let input = [1, 2];
    let [first, second] = input;
    console.log(first); // outputs 1
    console.log(second); // outputs 2
    ```
    * 解构作用于已声明的变量会更好：
    ```
    // swap variables
    [first, second] = [second, first];
    ```
    * 作用于函数参数：
    ```
    function f([first, second]: [number, number]) {
        console.log(first);
        console.log(second);
    }
    f(input);
    ```
    * 你可以在数组里使用...语法创建剩余变量：
    ```
    let [first, ...rest] = [1, 2, 3, 4];
    console.log(first); // outputs 1
    console.log(rest); // outputs [ 2, 3, 4 ]
    ```
    * 当然，由于是JavaScript, 你可以忽略你不关心的尾随元素：
    ```
    let [first] = [1, 2, 3, 4];
    console.log(first); // outputs 1
    ```
    * 或其它元素：
    ```
    let [, second, , fourth] = [1, 2, 3, 4];
    ```
* 解构对象
    * 解构对象：
    ```
    let o = {
        a: "foo",
        b: 12,
        c: "bar"
    };
    let { a, b } = o;
    这通过 o.a and o.b 创建了 a 和 b 。 注意，如果你不需要 c 你可以忽略它。
    ```
    * 就像数组解构，你可以用没有声明的赋值：
    ```
    ({ a, b } = { a: "baz", b: 101 });
    注意，我们需要用括号将它括起来，因为Javascript通常会将以 { 起始的语句解析为一个块。
    ```
    * 你可以在对象里使用...语法创建剩余变量：
    ```
    let { a, ...passthrough } = o;
    let total = passthrough.b + passthrough.c.length;
    ```
    * 属性重命名,你也可以给属性以不同的名字：
    ```
    let { a: newName1, b: newName2 } = o;
    这里的语法开始变得混乱。 你可以将 a: newName1 读做 "a 作为 newName1"。 
    令人困惑的是，这里的冒号不是指示类型的。 如果你想指定它的类型， 仍然需要在其后写上完整的模式。
    let {a, b}: {a: string, b: number} = o;
    ```
    * 默认值，默认值可以让你在属性为 undefined 时使用缺省值：
    ```
    function keepWholeObject(wholeObject: { a: string, b?: number }) {
        let { a, b = 1001 } = wholeObject;
    }
    现在，即使 b 为 undefined ， keepWholeObject 函数的变量 wholeObject 的属性 a 和 b 都会有值。
    ```
* 解构函数声明
    * 解构也能用于函数声明。 看以下简单的情况：
    ```
    type C = { a: string, b?: number }
    function f({ a, b }: C): void {
        // ...
    }
    ```
    * 但是，通常情况下更多的是指定默认值，解构默认值有些棘手。 首先，你需要在默认值之前设置其格式。
    ```
    function f({ a="", b=0 } = {}): void {
        // ...
    }
    f();
    上面的代码是一个类型推断的例子，将在本手册后文介绍。
    ```
    * 其次，你需要知道在解构属性上给予一个默认或可选的属性用来替换主初始化列表。 要知道 C 的定义有一个 b 可选属性：
    ```
    function f({ a, b = 0 } = { a: "" }): void {
        // ...
    }
    f({ a: "yes" }); // ok, default b = 0
    f(); // ok, default to {a: ""}, which then defaults b = 0
    f({}); // error, 'a' is required if you supply an argument
    ```
    * 要小心使用解构。 从前面的例子可以看出，就算是最简单的解构表达式也是难以理解的。 尤其当存在深层嵌套解构的时候，就算这时没有堆叠在一起的重命名，默认值和类型注解，也是令人难以理解的。 解构表达式要尽量保持小而简单。 你自己也可以直接使用解构将会生成的赋值表达式。

## 展开
* 展开操作符正与解构相反。 
    * 它允许你将一个数组展开为另一个数组，或将一个对象展开为另一个对象。 例如：
    ```
    let first = [1, 2];
    let second = [3, 4];
    let bothPlus = [0, ...first, ...second, 5];
    这会令bothPlus的值为[0, 1, 2, 3, 4, 5]。 展开操作创建了 first和second的一份浅拷贝。 它们不会被展开操作所改变。
    ```
    * 你还可以展开对象：
    ```
    let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
    let search = { ...defaults, food: "rich" };
    search的值为{ food: "rich", price: "$$", ambiance: "noisy" }。 对象的展开比数组的展开要复杂的多。 像数组展开一样，它是从左至右进行处理，但结果仍为对象。 这就意味着出现在展开对象后面的属性会覆盖前面的属性。 因此，如果我们修改上面的例子，在结尾处进行展开的话：

    let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
    let search = { food: "rich", ...defaults };
    那么，defaults里的food属性会重写food: "rich"，在这里这并不是我们想要的结果。
    ```
    * 对象展开还有其它一些意想不到的限制。 首先，它仅包含对象 自身的可枚举属性。 大体上是说当你展开一个对象实例时，你会丢失其方法：
    ```
    class C {
      p = 12;
      m() {
      }
    }
    let c = new C();
    let clone = { ...c };
    clone.p; // ok
    clone.m(); // error!
    ```
    * 其次，TypeScript编译器不允许展开泛型函数上的类型参数。 这个特性会在TypeScript的未来版本中考虑实现。