# typescript 基础数据类型

* boolean，let isDone:boolean = false;
* 数字，支持10进制，2进制，8进制，16进制；
* 字符串，let name:string = "bob";
    * 可以用双引号或者单引号；name = 'john';
    * 可以使用模板字符串，可以用反引号定义多行文本：
    ```
    let name: string = `Gene`;
    let sentence:string = `hello,my name is ${name}.
    
    I'll be ${ age + 1 } years old next month.`;
    ```
* 数组,有两种方式定义数组：
    * 在元素类型后面接[],let list: number[] = [1,2,3];
    * 使用数组泛型，Array<元素类型>,let list: Array<number> = [1,2,3];
* 元组，允许表示一个已知元素数量和类型的数组，各元素类型不必相同。
    * 例如：
    ```
    let x: [string,number];
    x = ['hello',10]; // OK
    x = [10,'hello']; // Error
    ```
    * 当访问一个已知索引的元素，会得到正常的类型：
    ```
    console.log(x[0].substr(1)); // OK
    console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
    ```
    * 当访问一个越界的元素，会使用联合类型替代：
    ```
    x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
    console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
    x[6] = true; // Error, 布尔不是(string | number)类型
    ```
    * 联合类型是高级主题，以后讨论它。
* 枚举，enum类型是对JavaScript标准数据类型的一个补充。 使用枚举类型可以为一组数值赋予友好的名字。
    ```
    enum Color {Red, Green, Blue}
    let c: Color = Color.Green;
    ```
    * 默认是从0开始为元素编号，也可以手动指定成员的数值。
    ```
    enum Color {Red = 1, Green, Blue}  /*or */ enum Color {Red = 1, Green = 2, Blue = 4}
    let c: Color = Color.Green;
    ```
    * 枚举提供一个便利，是可以由枚举的值得到它的名字：
    ```
    enum Color {Red = 1, Green, Blue}
    let colorName: string = Color[2];
    console.log(colorName);  // 显示'Green'因为上面代码里它的值是2
    ```
* Any，我们可以为编程时未知类型的变量标记为Any：
    ```
    let notSure: any = 4;
    notSure = "maybe a string instead";
    notSure = false; // okay, definitely a boolean
    let list: any[] = [1, true, "free"];
    list[1] = 100;
    ```
* Void，表示没有任何类型（可以理解为与Any相反）：
    ```
    function warnUser(): void {
        console.log("This is my warning message");
    }
    ```
    * 声明一个void类型的变量没有什么大用，因为只能为它赋予undefined 或者 null;
* Null和Undefined，TypeScript里，undefined和null两者各自有自己的类型分别叫做undefined和null。 
    * 和 void相似，它们的本身的类型用处不是很大。
    * 默认情况下null和undefined是所有类型的子类型。 就是说你可以把null和undefined赋值给number类型的变量。
    * 然而，当你指定了--strictNullChecks标记，null和undefined只能赋值给void和它们自己。 这能避免很多常见的问题。 
    * 也许在某处你想传入一个 string或null或undefined，你可以使用联合类型string | null | undefined。 再次说明，稍后我们会介绍联合类型。
    * `注意：`我们鼓励尽可能地使用--strictNullChecks，但在本手册里我们假设这个标记是关闭的。
* Never，表示的是那些永不存在的值的类型。 
    * never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never类型，当它们被永不为真的类型保护所约束时。
    * never类型是任何类型的子类型，也可以赋值给任何类型；
    * 然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 即使 any也不可以赋值给never。
    * 示例：
    ```
    // 返回never的函数必须存在无法达到的终点
    function error(message: string): never {
        throw new Error(message);
    }

    // 推断的返回值类型为never
    function fail() {
        return error("Something failed");
    }

    // 返回never的函数必须存在无法达到的终点
    function infiniteLoop(): never {
        while (true) {
        }
    }
    ```
* Object，表示非原始类型
    ```
    declare function create(o: object | null): void;

    create({ prop: 0 }); // OK
    create(null); // OK

    create(42); // Error
    create("string"); // Error
    create(false); // Error
    create(undefined); // Error
    ```
* 类型断言，类型其他语言中的强制类型转换，但是不进行特殊的数据检查和解构，它没有运行时的影响，只在编译阶段起作用。
    * 类型断言有两种形式，其一是“尖括号”语法：
    ```
    let someValue: any = "this is a string";
    let strLength: number = (<string>someValue).length;
    ```
    * 另一个是as语法：
    ```
    let someValue: any = "this is a string";
    let strLength: number = (someValue as string).length;
    ```
    * 两种形式是等价的。 但当你在TypeScript里使用JSX时，只有as语法断言是被允许的。
* 关于let，你可能已经注意到了，我们使用let关键字来代替大家所熟悉的JavaScript关键字var。 let关键字是JavaScript的一个新概念，TypeScript实现了它。 我们会在以后详细介绍它，很多常见的问题都可以通过使用 let来解决，所以尽可能地使用let来代替var吧。