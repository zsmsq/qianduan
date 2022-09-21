## 一、通过React构建页面

### 1-1 基本使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!-- react核心库，提供 React对象 -->
    <script src="../lib/react.development.js"></script>

    <!-- react操作dom的库，提供ReactDOM ReactDOM 库依赖于react库，所以引入要有先后顺序-->
    <script src="../lib/react-dom.development.js"></script>
</head>
<body>
    <!-- 指定react容器（“根” DOM 节点）用户操作界面【react的话剧舞台】 -->
    <div id="root"></div>
    <script>
        console.log(React,ReactDOM)
        // 使用 react-dom.development库提供的 ReactDOM 对象 创建容器
        const root = ReactDOM.createRoot(document.querySelector('#root'))
        // 可以通过 root.render('渲染的内容') 进行页面渲染
        root.render('hello react!')
    </script>

</body>

</html>
```

### 1-2 注意事项

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="../lib/react.development.js"></script>
    <script src="../lib/react-dom.development.js"></script>
</head>
<body>
    <div id="root"></div>
    <div class="box"></div>
    <script>
        
        {
            // 1. 可以在同一个页面指定多个容器
            // const root = ReactDOM.createRoot(document.querySelector('#root'))
            // root.render('字符串')
            // const box = ReactDOM.createRoot(document.querySelector('.box'))
            // box.render('box舞台')
        }
        {
            // 2. 同一个容器可以进行多次渲染，后面的渲染结果，会覆盖前面的
            // const root = ReactDOM.createRoot(document.querySelector('#root'))
            // root.render('字符串')
            // root.render('二次渲染')
        }
        {
            // 3. 同一个容器，不允许被多次指定为react舞台,没有报错，弹的警告，不建议这样做
            // const root1 = ReactDOM.createRoot(document.querySelector('#root'))
            // const root2 = ReactDOM.createRoot(document.querySelector('#root'))
            // root1.render('root1')
            // root2.render('root2')
        }
        {
            // 4. 不用将html、body 设置为react容器
            // html
            // const root = ReactDOM.createRoot(document.documentElement)
            // root.render('html')

            // body
            // const root = ReactDOM.createRoot(document.body)
            // root.render('body')
        }
        {
            // 5. 可以支持链式调用。
            ReactDOM.createRoot(document.querySelector('#root')).render('链式调用')
        }
    </script>
</body>
</html>
```

### 1-3 挂载与卸载

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="../lib/react.development.js"></script>
    <script src="../lib/react-dom.development.js"></script>
</head>
<body>
    <div id="root"></div>
    <button>点击</button>
    <button>卸载</button>
    <script>
    
        const root = ReactDOM.createRoot(document.querySelector('#root'))
        const btns = document.querySelectorAll('button')
        let count = 1

        root.render(count) // render可以渲染变量
        btns[0].onclick = function(){
            count++
            root.render(count)
        }

        btns[1].onclick = function(){
            // 卸载：root容器经不被react管理了
            root.unmount()
        }

    </script>
</body>
</html>
```



## 二、虚拟DOM

### 2-1 什么是虚拟DOM？

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>

    <script>
        /**
         *  react vue 用到的页面渲染优化方案。面试的时候也经常问
         *  1- 页面是如何加载渲染的？
         *      输入网址按下回车键，会发送请求，请求之后会得到相应的相应结果
         *      响应一个html，浏览器的html解析器就会对html进行解析，并生成 DOM树
         *      响应的是css， 浏览器的css解析器就会将css 解析生成一个css的 规则树
         *      响应的js，    js会操作css以及DOM，往往会造成多次的重排与重绘
         * 
         *  2- 什么是重排与重绘？
         *      css中的一些 几何属性（位置、大小），一旦发生变化，浏览器就会将几何属性重新计算
         *      因为几何属性不光影响自己，还影响其他周围的元素（导致周围元素的几何属性也要重新计算）
         *      计算的过程称为重排。重排之后对其进行重新渲染，渲染的过程称为重绘
         *      所以，重排一定会导致重绘
         *  3- 传统DOM操作（jquery）的缺陷与弊端？
         *      频繁进行dom操作，导致大量的重排与重绘。效率性能都比较差
         *      this.style.height = 100px;
         *      this.style.height = 300px;
         *  4- React vue性能更好，为啥呢？
         *      引入虚拟DOM，react操作的是虚拟dom，不是真实的DOM。不会导致大量重排和重绘
         * 
         *  5- 虚拟DOM是什么？
         *      1- 虚拟DOM就是一个对象，该对象的属性与真实DOM的属性是一一对应的
         *      2- 虚拟DOM最终会通过 render方法，渲染为真实DOM
         *      3- 虚拟DOM是轻量级，属性比真实DOM要少很多。只拥有他需要的指定属性
         *      4- 虚拟DOM在React中，我们称之为 React元素
         *      5- 虚拟DOM在react中的创建有两种方式
         *          5-1 通过 React.createElement() 来创建
         *          5-2 通过 jsx 创建
         * 
         */
    </script>
</body>
</html>
```

### 2-2 ```React.createElement```(了解即可)

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="../lib/react.development.js"></script>
    <script src="../lib/react-dom.development.js"></script>
</head>

<body>
    <div class="testClass">testClass</div>
    <div id="root"></div>
    <script>
        const root = ReactDOM.createRoot(document.querySelector('#root'))
        // render方法：把div当做字符串，并没有对<div> 进行解析。那我如果想要渲染html 标签，怎么办？
        root.render('<div>我是div</div>');

        {
            // 创建虚拟dom
            /**
             * 
             *  React.createElement(ele,props,children)
             *  第一个参数：是标签名
             * 
             */
            // const oDiv = React.createElement('div')
            // const realDom = document.createElement('div')
            // console.log(oDiv);
            // console.dir(realDom)
            // // 将虚拟dom渲染到页面
            // root.render(oDiv)
        }
        {
            // 第二个参数是一个对象，用来设置虚拟dom 的属性
            // const oDiv = React.createElement('div',{id:'box',school:'atguigu'})
            // console.log(oDiv)
            // root.render(oDiv)//<div id="box" school="atguigu"></div>

        }
        {
            // 特殊属性class. 设置时需要用属性名 className 来设置
            // 为什么？因为真实DOM 用的就是className
            // 为什么真实DOM，要用className呢？而不用class呢？
            // 因为
            // const oDiv = React.createElement('div',{id:'box',className:'c1'})
            // console.log(oDiv)
            // root.render(oDiv) // <div id="box" class="c1"></div>

            // // 看下真实DOM的 class
            // const testDiv = document.querySelector('.testClass')
            // console.dir(testDiv)

            // // class 是js的关键字，为了避免冲突和误解，所以DOM属性的class ===》 className
            // class Person{

            // }
            // 虚拟DOM和 真实DOM属性是一一对应的。
        }
        {
            // 第三个参数开始都是：标签元素的 子节点。 值会被设定到虚拟dom的 children属性中
            const oDiv = React.createElement('div',{id:'box',className:'c1'},'我是div')
            console.log(oDiv)
            root.render(oDiv)
        }
        {
            // 多个元素子节点，怎么处理？
            /**
             *   <div class="c1">
             *      <h3>标题</h3>
             *      <p>正文</p>
             *  </div>
             *
             */
            // 如果标签没有任何属性，也需要写 null或{} 占位
            // const oH3 = React.createElement('h3',{},'标题')
            // const oP = React.createElement('p',null,'正文')
            // 多个子节点写法一：                                  从第三个参数开始，都是子节点
            // const oDiv = React.createElement('div',{className:'c1'},oH3,oP)
            // 多个子节点写法二：       
            //       如果是数组，属性需要加key 并且值不能重复，该处涉及到diff算法，后面详解
            //   key属性是一个特殊属性，不会渲染到真实的dom中
            const oH3 = React.createElement('h3',{key:1},'标题')
            const oP = React.createElement('p',{key:2},'正文')                      
            oDiv = React.createElement('div',{className:'c1'},[oH3,oP])
            root.render(oDiv)
            
        }


    </script>
</body>

</html>
```

### 2-3 jsx基本语法

> jsx ===> js javascript  x=> xml
>
> 用来快速生成react元素。与 React.createElement 作用一致
>
> 1. jsx 不能使用非 html 标签。
> 2. jsx创建的虚拟dom 必须有且只有为一个根标签
> 3. 可以通过 React.Fragment 来包裹过个标签。
>    1. <React.Fragment>xxxxx </React.Fragment>
>    2. <></>

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="../lib/react.development.js"></script>
    <script src="../lib/react-dom.development.js"></script>
    <!-- 浏览器不支持jsx主要用babel转化，所以引入 -->
    <script src="../lib/babel.min.js"></script>
    <style>
        .c1 {
            background-color: aquamarine;
        }
    </style>
</head>

<body>
    <div id="root"></div>
    <script type="text/babel">
        /**
         *  jsx:  js==>javascript   x==> xml
         *  xml:是一种数据格式,在json数据格式出现之前，应用最广范的数据格式
         *      1. const xhr = new XMLHttpRequest()
         * 
         *  jsx 是react特有的语法。快速创建react的虚拟dom===》 react元素
         *  <div></div>
         * 
         * jsx 浏览器不支持，要想支持需要满足一下两点
         *      1. 引入 babel进行转化
         *      2. 在有jsx语法的代码段 script 标签上需要增加类型 type="text/babel"
         * 
         */
        const root = ReactDOM.createRoot(document.querySelector('#root'))

        {
            // 以下两句等价：jsx创造的，也是 react元素，即react虚拟dom
            root.render(<div className="c1">123</div>)  // jsx
            root.render(React.createElement('div', { className: 'c1' }, '123'))
        }
        {
            // jsx 只能写 html的标签，不能写html没有的标签，因为jsx创建的react元素，最终也是为了转化为真实的dom元素。
            // 因此必须是html标签名的jsx
            root.render(<username>123</username>) //The tag <username> is unrecognized in this browser
            // 浏览器不认识 username标签，会报错
        }
        {
            //    vdom virtual dom   virtual 虚拟
            //可以将jsx创建的 虚拟dom 存入变量
            const vdom = <p>我是p标签</p>
            root.render(vdom)
        }

        {
            // 虚拟dom 结构比较复杂的时候，我们为了方便的格式化和便于阅读，会进行换行。那建议在虚拟dom外围加一个 （）
            const vdom = (
                <div>
                    <h3>我是标题</h3>
                    <p>我是正文</p>
                </div>
            )
            root.render(vdom)
        }
        {
            // 虚拟dom 只能有一个根节点
            // const vdom = (
            //     <h3>我是标题</h3>
            //     <p>我是正文</p>
            // )
            // 报错:Adjacent JSX elements must be wrapped in an enclosing tag
            //       相邻的  jsx 元素必须被包裹在一个封闭的标签里


            // 解决根节点问题：创建一个文档碎片
            // 解决方案一： React.Fragment 文档碎片，创建了一个容器，用来包裹内部的jsx，但是在页面渲染的时候，不会渲染到真实dom中去
            // let vdom = (
            //     <React.Fragment>
            //         <h3>我是标题</h3>
            //         <p>我是正文</p>
            //     </React.Fragment>
            // )

            // 解决方案二：
            // const {Fragment} = React
            // let vdom = (
            //     <Fragment>
            //         <h3>我是标题</h3>
            //         <p>我是正文</p>
            //     </Fragment>
            // )

            // 解决方案三：简写
            let vdom = (
                <>
                    <h3>我是标题</h3>
                    <p>我是正文</p>
                </>
            )
            root.render(vdom)
        }

    </script>
</body>

</html>
```

### 2-4 配置代码片段

> vscode 配置 react代码片段
>
> 1. 文件===》 首选项===》 配置用户代码片段==》 新代码片段==》 复制粘贴以下内容
> 2. 使用： !react + 回车

```js
{
	// Place your snippets for html here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }

	"react模板":{
		"prefix": "!react",
		"body": [
			"<!DOCTYPE html>",
			"<html lang=\"en\">",
			"<head>",
				"\t<meta charset=\"UTF-8\">",
				"\t<title>Title</title>",
				"\t<script src=\"../lib/react.development.js\"></script>",
				"\t<script src=\"../lib/react-dom.development.js\"></script>",
				"\t<script src=\"../lib/babel.min.js\"></script>",
			"</head>",
			"<body>",
			"\t<div id=\"root\"></div>",
			"</body>",
			"<script type=\"text/babel\">",
			"\tconst root = ReactDOM.createRoot(document.querySelector(\"#root\"));",
			"\troot.render((",
				"\t\t<div></div>",
			"\t))",
			"</script>",
			"</html>"
		],
		"description": "快速构建react模板页页面"
	}
}
```







### 3-3 jsx中的插值表达式

```js
/**
     *  JS 代码由两部分组成: 1. js 语句 2.js表达式
     *  JS 语句：
     *  赋值语句： let a = 1
     *  条件语句： if(){}
     *  循环语句： for(){}  while(){}
     *  选择语句： switch(){}
     *  
     *  JS表达式：是一种特殊语句，必须要有返回值的
     * 
     *  变量  usonb  you so niu b
     *      u:undefined
     *      s:string  symbol
     *      o:object       
     *          1. 对象： {} Objects are not valid as a React child ,  {}  无法渲染，报错
     *          2. 数组： [1,2,3,4] 将数组中的每一个元素进行渲染
     *      n:number  null
     *      b:boolean
     *          1. true    渲染结果为空
     *          2. false   渲染结果为空
     *         
     *  常量
     *  三元表达式
     *  函数() 
     * 
     *  JSX 中的插值表达式：
     *  语法： {表达式}
     * 
     *  总结：插值表达式中如果是 布尔值、null、undefined不会输出任何内容
     *        插值表达式不支持 对象 {}
     *        如果是数组： 将数组中的每一个元素展开，进行渲染
     * 
     */
    const root = ReactDOM.createRoot(document.querySelector("#root"));
    {
        // 插值表达式基本使用

        let str = '我是插值div'
        // vdom 本身也是一个变量，变量的值是 一个react元素【虚拟dom】
        const vdom = (
            <div>{str}</div>
        )
        root.render((
            <div>
                {vdom}
            </div>
        ))
    }
    {
        let num = 0
        let str = '我是字符串'
        let b1 = true
        let b2 = false
        let obj = {
            name: 'atguigu'
        }
        let arr = [1, 2, 3, 4]
        let fn = function () {
            // return {}  对象就会报错，处理不了
            return [22,33,44]

        }
        root.render((
            <div>
                <p>{num}</p>
                <p>{str}</p>
                <p>{b1}</p>
                <p>{b2}</p>
                <p>{null}</p>
                <p>{undefined}</p>
                {/*多行注释<p>{obj}</p>*/}
                {
                    // 单行注释
                    //sdffasdf
                }
                <p>{arr}</p>
                <p>{fn()}</p>
                
            </div>
        ))
    }
```

### 3-4 插值表达式条件渲染

```js
const root = ReactDOM.createRoot(document.querySelector("#root"));
    /**
     * 如果性别是男，那么显示男同学
     * 性别是女，显示女同学
     * 
     * 
     */
    const sex = 1 // 1 表示男 0表示女
    // const vdom = <div>男同学</div>
    // const vdom2 = <div>女同学</div>

    const vdom = (
        sex === 0 ? <div>男</div> : <div>女</div>
    )
    const isShow = true
    /**
     *  && ： 表达式 = 表达式1 && 表达式2 
     *        如果表达式1的布尔值为真，那么整个表达式的值就是表达式2的值
     *        如果表达式1的布尔值为假，那么整个表达式的值就是表达式1的值
     * 
     * || ：  表达式 = 表达式1 || 表达式2 
     *        如果表达式1的布尔值为真，那么整个表达式的值就是表达式1的值
     *        如果表达式1的布尔值为假，那么整个表达式的值就是表达式2的值
     * 
     */
    console.log(5&&8)
    console.log(false && 10)
    console.log(0 || false)
    console.log(1 || 100)

    function fn(){
        return '我是函数返回值'
    }

    /**
     *  90分以上，优秀毕业生
     *  75分以上，学的还不错
     *  60分以上，及格
     *  40分以上，仍需努力，还有救
     *  0分以上， 洗洗睡吧！
     * 
     */
    function getStudent(score){
        if(score>=90) return '优秀毕业生'
        if(score>=75) return '学的还不错'
        if(score>=60) return '及格'
        if(score>=45) return '仍需努力，还有救'
        if(score>=0)  return '洗洗睡吧！'
    }
    root.render((
        <div>
            <div>{vdom}</div>
            <div>{sex === 1 ? '男' : '女'}</div>
            <div>{isShow && 'loading.....'}</div>
            <div>{isShow || '或运算'}</div>
            <div>{fn && fn()}</div>
            <div>{getStudent && getStudent(31)}</div>
        </div>
    ))
```

### 3-5 插值表达式属性赋值

```js
const root = ReactDOM.createRoot(document.querySelector("#root"));

    let width = 500; // 属性如果有单位，并且单位是px，那么px可以省略，只写数字即可
    let height = 200;
    let style = {
        width:700,
        height:700,
        border:2
    }
    root.render((
        <div>
            {
                // 属性赋值，解析变量，外层没有引号
                //<img src="{imgStr}" />
            }
            
            <table width={width} height={height}  border="1">
                <tr>
                    <td>11</td>
                </tr>
            </table>

            <table width={style.width} height={style.height} border={style.border}>
                <tr>
                    <td>222</td>
                </tr>
            </table>

            <table {...style}>
                <tr>
                    <td>333</td>
                </tr>
            </table>
            
            <div width={width}>999</div>

        </div>
    ))
```

### 3-6 style 样式

> - 样式属性名使用小驼峰命名法
> - 如果样式是数值，可以省略单位

```js
const root = ReactDOM.createRoot(document.querySelector("#root"));
    /**
     *  style：设置元素的行内样式，值必须是一个对象.  style={{width}}
     *         外层的花括号，表示插值语法，内层的花括号是对象定义的语法
     *  有一些样式是用 - 分割的 比如 font-size ===> 驼峰的写法 fontSize
     * 
     */
    root.render((
        <div>
            <div style={{width:100,height:100,background:'red',fontSize:60}}>1111</div>
        </div>
    ))
```

### 3-7 className 设置样式

> 必须用className, 不能用class

```js
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../lib/react.development.js"></script>
    <script src="../lib/react-dom.development.js"></script>
    <script src="../lib/babel.min.js"></script>
    <style>
        .c1 {
            color: aqua
        }

        .bg1 {
            background-color: cornflowerblue;
        }
    </style>
</head>

<body>
    <div id="root"></div>
</body>
<script type="text/babel">
    const class1 = 'c1'
    const class2 = 'c1 bg1'
    const arrStyle = ['c1','bg1']// 数组插值表达式渲染，如果是在页面里，那么 数组中的每一项都会被渲染，并且中间没有 ","
    // 但是如果是在属性中，使用插值表达式渲染，那么中间有 ","
    const root = ReactDOM.createRoot(document.querySelector("#root"));
    root.render((
        <div>
            <div className="c1">李思卓</div>
            <div className={class1}>杜虎林</div>
            <div className={class2}>刘斌峰</div>
            <div className={arrStyle}>王丹丹</div>
            <div>{arrStyle}</div>
            <div className={arrStyle.join(' ')}>杨萌</div>
        </div>
    ))
</script>

</html>
```



### 3-8 列表渲染（重中之重）

> 必须给列表项添加唯一的 key 属性, 推荐使用id作为key, 尽量不要用index作为key

```js
const root = ReactDOM.createRoot(document.querySelector("#root"));
    {
        // 1 - {} 可以将数组元素进行展开渲染
        let arr = [1, 2, 3, 4]
        // root.render((
        //     <div>{arr}</div>
        // ))
    }
    {
        // 2- 数组中的元素，可以是 react元素【虚拟dom】
        // let arr = [<p>1</p>, <p>2</p>, <p>3</p>]
        // root.render((
        //     <div>{arr}</div>
        // ))
    }
    {
        // 3- 本来是普通数组，但可以对其进行转化，转化为 react元素的数组
        let arr = [1, 2, 3, 4, 5, 6, 7, 8]
        // 3-1 for循环
        // for (let i = 0; i < arr.length; i++) {
        //     arr[i] = <p>{arr[i]}</p>
        // }
        // 普通数组，变成react元素数组
        // console.log(arr);
        // root.render((
        //     <div>{arr}</div>
        // ))

    }
    {
        // 3-2 将普通数组转化为react元素数组   forEach
        // let arr = [1, 2, 3, 4, 5, 6, 7, 8]

        // arr.forEach((item, index) => {
        //     arr[index] = <p>{item}</p>
        // })
        // root.render((
        //     <div>{arr}</div>
        // ))
    }
    {
        // 3-3 将普通数组转化为react元素数组   map
        let arr = [1, 2, 3, 4, 5, 6]
        // arr = arr.map(item=><p>{item}</p>) //如果函数体内语句只有一句，那么{}可以省略。如果返回值就是该唯一语句，那么return也可以省略。这句写全了，就是下面的代码
        // arr = arr.map(item=>{
        //     return <p>{item}</p>
        // })

        // arr = arr.map(item => (
        //     <p>
        //         <h1>123</h1>
        //     </p>)) // 如果map回调函数，外有 {},那么必须有return语句。
        // () 将包含的内容进行返回

        // console.log(arr);
        // root.render((
        //     <div>{arr}</div>
        // ))
    }
    {
        // 3-4 map 转化后直接渲染，这种方式必须掌握
        // const arr = [1, 2, 3, 4]
        // root.render((
        //     <div>
        //         {arr.map(item => <p>{item}</p>)}
        //     </div>
        // ))
    }

    {
        // 4- map返回值一般而言 react元素。要求：返回的react元素，必须只能有一个根节点
        // let arr = [1, 2, 3, 4]
        // root.render((
        //     <div>
        //         {/*
        //             {arr.map(item=>{
        //                 return (
        //                     <h3>我是标题</h3>
        //                     <p>我是内容</p>
        //                 )
        //             })}

        //             返回多个同级元素标签，会报错，必须有一个唯一的元素标签，进行包裹，返回才可以
        //         */}

        //         {arr.map(item => {
        //             return (
        //                 <>
        //                     <h3>我是标题</h3>
        //                     <p>我是内容</p>
        //                 </>
        //             )
        //         })}
        //     </div>
        // ))
    }

    {
        let data = [
            {
                id: 1,
                title: 'estar pro 与重庆狼队会师总决赛',
                href: 'http://baidu.com'
            },
            {
                id: 2,
                title: '唐山打人事件，详细视频曝光',
                href: 'http://qq.com'
            },
            {
                id: 3,
                title: '西安疫情稳定，无社会面新增',
                href: 'atguigu.com'
            }
        ]
        // <ul>
        //     <li>
        //         <h3>标题</h3>
        //         <p>链接</p>
        //     </li>
        //     <li>
        //         <h3>标题</h3>
        //         <p>链接</p>
        //     </li>
        // </ul>

        /**
         *  渲染列表时，每一个循环的列表元素，身上都要有一个key属性，并且值是唯一不重复的。
         *  当列表渲染到页面，key属性并不会出现在真实dom上。key属性的作用，是为了diff算法。后面详解
         *  key的值，我们一般而言使用数据的唯一标识 id.
         *
         */
        // root.render((
        //     <ul>
        //         {data.map(item=>(
        //             <li key={item.id}>
        //                 <h3>{item.title}</h3>
        //                 <p>{item.href}</p>
        //             </li>
        //         ))}
        //     </ul>
        // ))

        // <div>
        //     <h3></h3>
        //     <p></p>
        //     <h3></h3>
        //     <p></p>
        // </div>

        /**
         *  下面的事例，不可以用文档碎片的简写方式 <> ，因为文档碎片作为被循环的元素，需要有key，但<key={item.id}>，存在歧义。解析器认为，你要渲染一个 <key></key> 标签，所以会报错
         *  
         * 此处，只能使用完整写法的 文档碎片作为外层循环元素
         * 
         */
        root.render((
            <div>
                {data.map(item => (
                    <React.Fragment key={item.id}>
                        <h3>{item.title}</h3>
                        <p>{item.href}</p>
                    </React.Fragment>
                ))}
            </div>
        ))

    }
```

### 3-9 原生DOM事件

>事件名： on+事件  onclick oninput onchange ......
>
>e：事件对象：
>
>e.target: 事件源对象【事件是在谁身上最先触发的】
>
>e.preventDefault(): 阻止元素的默认行为
>
>e.stopPropagation(): 阻止事件冒泡
>
>oninput 事件：文本框输入触发
>
>onchange事件：文本框值有变化，失去焦点触发

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <a href="http://baidu.com">百度</a>
    <button >按钮1</button>
    <button onclick="btn2Fn(event,this)">按钮2</button>
    <button >按钮3</button>
    <input type="text" name="" id="">
    <input type="text" name="" id="">
    <script>
        function btn2Fn(e,_this){
            console.log('按钮2')
            // 事件对象
            console.log(e);
            // this 指向window
            console.log(this);
            // _this 指向按钮
            console.log(_this)
        }
        const btns = document.querySelectorAll('button')
        const a = document.querySelector('a')
        a.onclick = function(e){
            alert('a')
            // 阻止默认跳转行为
            // 方式一：return false
            // return false;
            // 方式二：
            e.preventDefault();
            console.log(e);
        }

        btns[0].onclick = function(e){
            console.log(e);
            console.log(this);
        }


        function btn3Fn(event, _this){
            // _this 指向按钮
            console.log(event, _this)
            // this 指向window
            console.log(this);
        }
        btns[2].onclick = function(event){
            btn3Fn(event,this)
        }

        // dom事件，绑定事件时，都是用 on+事件名的形式绑定
        // oninput : 文本框输入时触发
        // onchange：文本框内容发生变化，且失去焦点时触发

        const oInputs = document.querySelectorAll('input')
        // 有输入就一直触发
        oInputs[0].oninput = function(e){
            // e.target 获取事件源对象，也就是当前的文本框
            console.log(e.target.value)
        }

        // onchange：失去焦点触发
        oInputs[1].onchange = function(e){
            console.log(e.target.value);
        }
    </script>
</body>
</html>
```

### 3-10 react jsx事件

> 1. jsx中事件对象是包装过的
> 2. 获取原生事件对象 nativeEvent属性获取
> 3. onChange事件，实际是原生DOM事件中的 oninput
> 4. 事件都是驼峰命名法
> 5. onClick={函数名}
> 6. 阻止默认行为使用 e.preventDefault() 。 return false 不管用

```js
/**
     *  原生DOM事件： on + 事件名 作为属性，全部是小写的。属性的值是一个字符串。当点击时，会将字符串当做js代码执行
     *  jsx事件：    on + 事件名  作为属性，采用驼峰命名法。属性值是一个函数
     *               jsx中的onchange事件，跟原生DOM是不同的。它实际是 原生dom中的oninput事件
     *               jsx中的事件对象，已经被处理和包装过了，不是原生的事件对象。
     *               在jsx事件对象中，nativeEvent 是原生的事件对象
     *               使用我们使用jsx中的事件对象更好，因为常用的属性也提供了，够用。第二jsx中的事件对象做了兼容性的处理。没有兼容性问题。
     *          
     */
    const root = ReactDOM.createRoot(document.querySelector("#root"));
    function fn() {
        // 使用严格模式
        console.log('fn', this)
    }
    root.render((
        <div>
            <button onClick={fn}>点击</button>
            <button onClick={() => {
                console.log('匿名箭头函数,', this)
            }}>点我2</button>
            <input type="text" name="" id="" onChange={(e)=>{
                console.log('change');
                console.log(e.target.value);
                //jsx 中的事件对象
                console.log(e)
            }}/>

			<a href="http://baidu.com" onClick={(e)=>{
                alert('百度')
                // 阻止默认行为
                // e.preventDefault();
                
                // 在jsx中阻止默认行为 只能用 e.preventDefault() 。 return false 不管用
                //return false
            }}>跳转去百度</a>

        </div>
    ))
```

### 3-11 综合练习

```html
需求：实现评论列表功能

- 如果有评论数据，就展示列表结构 li（ 列表渲染 ）要包含a标签
  - name 表示评论人，渲染 h3
  - content 表示评论内容，渲染 p
- 如果没有评论数据，就展示一个 h1 标签，内容为： 暂无评论！
- 用户名的字体25px, 内容的字体20px
- 点击内容区域提示它发表的时间
```

```js
const list = [
  { id: 1, name: 'jack', content: 'rose, you jump i jump', time: '03:21' },
  { id: 2, name: 'rose', content: 'jack, you see you, one day day', time: '03:22' },
  { id: 3, name: 'tom', content: 'jack,。。。。。', time: '03:23' }
]
```

```js
const list = [
        { id: 1, name: 'jack', content: 'rose, you jump i jump', time: '03:21' },
        { id: 2, name: 'rose', content: 'jack, you see you, one day day', time: '03:22' },
        { id: 3, name: 'tom', content: 'jack,。。。。。', time: '03:23' }
    ]
    const root = ReactDOM.createRoot(document.querySelector("#root"));

    let vdom = (
        list.length === 0 ? <h1>暂无评论</h1> : (
            <ul>
                {list.map(item => (
                    <li key={item.id}>
                        <a href="" onClick={(e)=>{
                            e.preventDefault();
                        }}>
                            <h3 className="title">{item.name}</h3>
                            <p style={{ fontSize: 20 }} onClick={() => {
                                alert(item.time)
                            }}>{item.content}</p>
                        </a>
                    </li>
                ))}
            </ul>
        )
    )
    root.render((
        <div>
            {vdom}
        </div>
    ))
```

### 3-12 关于注释

> 多行注释
>
> 单行注释
>
> 行内属性注释

```js
 const root = ReactDOM.createRoot(document.querySelector("#root"));
    root.render((
        <div>
            {/*多行注释*/}
            {/*
                中间都是注释的内容
            
            
            */}
            {
                // 单行注释
                // 单行注释
            }

            {
                // 以下是在行内注释属性
            }
            <button className="c1" 
                /*id="box"*/ type="button"
            >按钮</button>
        
        </div>
    ))
```

## 三 react-cli 脚手架

> React脚手架：用来帮助程序员快速创建一个基于React库的模板项目。
>
> 脚手架是开发现代Web应用必备利器
>
> 充分利用了webpack、Babel、EsLint辅助开发工具
>
> 零配置、无需手动配置繁琐的工具即可使用
>
> 让程序员可以关注业务开发、而不是工具配置
>
> 公司大神==》 构架你们公司自己的脚手架

### 使用

1. 全局安装：

```shell
npm i create-react-app -g #装好后会获得一个 create-react-app 命令
```

2. 创建项目

```shell
create-react-app 项目名 # 用npm 创建的项目
# 例如：create-react-app nproject123
yarn create react-app 项目名 # 用yarn创建的项目
# 例如： yarn create react-app yproject999
```

3. 启动项目

```shell
# 1. 进入项目目录
cd 项目名
# 2. 启动
# 如果是 npm 创建的项目
npm start
# 如果是 yarn 创建的项目
yarn start
```

4. 如何区分一个项目是yarn创建的还是npm创建的？

1. npm创建的项目：有特有文件 package-lock.json

2. yarn创建的项目：有特有文件 yarn.lock 

   lock文件是干什么用的？ 锁定版本的。因为git pull到项目后，是没有 node_modules目录的，所以需要本地执行 ```npm i ``` 或 ```yarn```进行依赖包的安装。lock文件中有详细的版本信息，已经下载地址。确保版本一致性

### yarn 常用命令

```shell
# 初始化项目
yarn init -y
# 使用yarn 安装
yarn add 包名
# 启动项目
yarn start
# pull 到项目后，根据package.json 安装依赖创建 node_modules
yarn

```

![image-20220829165716326](05-React%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0.assets/image-20220829165716326.png)

## 四、组件

> 组件的组成
>
> html + css + js 的一个混合体。具有相对独立的功能。
>
> 自定义组件：
>
> jsx语法规则： 当遇到 <> 时，首字母大写的会被认为是组件，进行渲染。首字母小写的作为普通标签(标签必须得被浏览器识别)
>
> <User ></User>   ==》 render方法
>
> 当遇到 {} 时，会把{} 中的内容当做 js 表达式来处理
>
> 在React中最小的单位是 React元素，组件是由若干个React元素组成的，页面是由若干个组件构成的。
>
> 在React应用中一般会设置一个 主组件，一般而言名字叫做 App
>
> 组件有两种形式： 函数组件(简单组件、无状态组件) 类组件（复杂组件、有状态组件）
>
> 16版本以前：类组件是学习的重点
>
> 16版本以后：Hook 钩子函数、可以通过Hook扩展函数组件的功能、让其有类组件所有的功能
>
> 现在的版本是 18.2
>
> 所以：我们学习的重点是 函数组件，类组件我们也会介绍，但不会用太多时间。

### 4-1. 脚手架目录结构介绍

![image-20220830091449227](05-React%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0.assets/image-20220830091449227.png)

```js
react-learn              项目目录
  |-node_modules         依赖包安装目录
  |-public			     静态资源目录
  |   |-favicon.ico      应用程序顶部的icon图标
  |   |-index.html       应用程序index.html入口文件
  |   |-logo192.png      在mainfest.json中被使用的
  |   |-logo512.png      在mainfest.json中被使用的
  |   |-mainfest.json    和web app 移动端配置有关
  |   |-robots.txt       指定搜索引擎可以或者不可爬取哪些文件
  |-src
  |	  |-App.css          App组件相关样式
  |   |-App.js           App组件的代码文件
  |   |-App.test.js      App组件的测试代码文件
  |   |-index.css        全局样式文件
  |   |-index.js         整个应用程序js的入口文件
  |   |-logo.svg         启动项目，看到的react图标
  |   |-reportWebVitals.js 给google搜索引擎提供的报告
  |   |-setupTests.js    测试初始文件
  |-.gitignore           git忽略文件
  |- package.json        整个应用程序的描述文件：包括应用的名称、版本号、依赖包、以及项目启动打包命令等等
  |- README.md           readme说明文件
  |- yarn.lock           yarn安装依赖包的具体版本信息文件

```

### 4-2.17 - 18 语法差别

![image-20220830094647121](05-React%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0.assets/image-20220830094647121.png)

### 4-3.脚手架项目改造

- 只保留public目录下的 index.html 和 favicon.ico两个文件，其他的删掉

  - public/index.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8" />
      <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      <meta name="theme-color" content="#000000" />
      <meta
        name="description"
        content="Web site created using create-react-app"
      />
      <title>React App</title>
    </head>
    <body>
      <div id="root"></div>
    </body>
  </html>
  
  ```

- src目录下只有两个文件

  - App.jsx

    ```js
    /**
     *  组件是由react元素组成的。
     * 
     *  类组件的定义：
     *  1. 类组件需要通过class进行定义，需要继承 React.Component 这个类
     *  2. 类组件必须有一个render方法，该方法的返回值是要渲染的内容。一般返回的是react元素
     *  3. 类组件的render方法返回值如果是 jsx【react元素】，那么有且只能有一个根元素
     *  4. 类名就是组件的名字，首字母必须大写，该组件在使用时，需要用jsx语法，<类组件名 /> 或 <类组件名></类组件名>   <App/> 或 <App></App>
     *  5. 当遇到 <App /> 时，会按照组件进行解析，会找App类，实例化App，用实例调用组件中的render方法执行，执行的返回值进行页面渲染
     *  6. 组件可以复用
     * 
     */
    
    import React, { Component } from 'react'
    // 以下两行等于上面一行
    // import React from 'react'
    // const {Component} = React
    
    export default class App extends Component {
        render() {
            return (
                <div>
                    <div>App123</div>
                    <h3>123</h3>
                </div>
            )
        }
    }
    
    ```

  - index.js

  ```js
  import React from 'react'
  import ReactDOM from 'react-dom/client'
  import App from './App'
  const root = ReactDOM.createRoot(document.getElementById('root'))
  root.render(<App/>)
  
  ```

### 4-4. 函数组件

  ```js
  /**
   * 函数组件：
   *      1. 函数组件是通过函数定义的、
   *      2. 函数组件必须要有返回值。【当前版本不报错】
   *      3. 函数的名字就是组件的名字。所以函数名首字母也要大写，因为调用的时候函数名 <函数名 />
   *      4. 函数组件一般情况下返回值都是 jsx [react元素集合]. JSX只能有一个根元素
   *      5. 建议返回的 jsx 通过（） 进行包裹
   *      6. 函数组件也可以复用【被多次调用】
   */
  
  function App(){
      return (
          <div>
              <h3>App</h3>
              <p>123123</p>
          </div>
      )
  }
  export default App
  ```

### 4-5. 类组件

```js
/**
 *  组件是由react元素组成的。
 * 
 *  类组件的定义：
 *  1. 类组件需要通过class进行定义，需要继承 React.Component 这个类
 *  2. 类组件必须有一个render方法，该方法的返回值是要渲染的内容。一般返回的是react元素
 *  3. 类组件的render方法返回值如果是 jsx【react元素】，那么有且只能有一个根元素
 *  4. 类名就是组件的名字，首字母必须大写，该组件在使用时，需要用jsx语法，<类组件名 /> 或 <类组件名></类组件名>   <App/> 或 <App></App>
 *  5. 当遇到 <App /> 时，会按照组件进行解析，会找App类，实例化App，用实例调用组件中的render方法执行，执行的返回值进行页面渲染
 *  6. 组件可以复用
 * 
 */

import React, { Component } from 'react'
// 以下两行等于上面一行
// import React from 'react'
// const {Component} = React

class App extends Component {
    render() {
        return (
            <div>
                <div>App123</div>
                <h3>123</h3>
            </div>
        )
    }
}
export default App
```



  #### 4-5-1.类组件和函数组件区别

  >  *  类组件和函数组件区别：
  >  * 
  >  *      1. 类组件有this：this指向当前组件的实例
  >  *      2. 类组件有状态【私有的数据】
  >  *      3. 类组件的调用，首先需要实例化，然后调用render方法
  >  * 
  >  * 函数组件：
  >  *      1. 函数组件调用，函数执行
  >  *      2. 函数组件没有this ，this指向 undefined
  >  *      3. 函数组件天生没有状态。需要通过 hook扩展

```js

// import React,{Component} from 'react'
// class App extends Component {
//     // 类组件是有状态的组件
//     constructor(){
//         // 子类继承父类，那么在子类的构造函数中，第一句就要执行父类的构造函数
//         // 执行父类的构造函数
//         super()
//         // 类组件定义自己的状态【自己的数据】
//         // state是固定名，定义状态就是给state赋值。值是一个对象
//         this.state = {
//             // 定义状态 count初始值是1
//             count:1
//         }
//     }
//     // render方法一定是 App组件实例调用的。实例化和render的调用，都是react帮咱们执行的。
//     render() {
//         //
//         console.log('class render ');
//         // this 指向当前组件的实例
//         console.log(this);
//         return (
//             <div>
//                 <div>App123</div>
//                 <h3>123</h3>
//                 <h3>{this.state.count}</h3>
//             </div>
//         )
//     }
// }

/***
 *  类组件和函数组件区别：
 * 
 *      1. 类组件有this：this指向当前组件的实例
 *      2. 类组件有状态【私有的数据】
 *      3. 类组件的调用，首先需要实例化，然后调用render方法
 * 
 * 函数组件：
 *      1. 函数组件调用，函数执行
 *      2. 函数组件没有this ，this指向 undefined
 *      3. 函数组件天生没有状态。需要通过 hook扩展
 * 
 */
function App(){
    console.log('function app render')
    // this 是undefined
    // 1. 函数组件没有this，this指向 undefined
    // 2. 说明在react脚手架中，使用的是严格模式

    // 因为函数组件没有this，所以函数组件天生没有状态。后天可以通过hook进行拓展，使其具备状态
    // 执行时机，都是在render 渲染的时候执行。
    console.log(this);
    return (
        <div>
            <h3>App</h3>
            <p>123123</p>
        </div>
    )
}
export default App
```

#### 4-5-2.类组件的状态
>  *  状态：
>  *  1. 状态如何定义？
>  *  2. 状态如何读取？
>  *  3. 状态如何改变？
>  * 
>  *  1.类组件定义状态：
>  *      1. 在构造函数中定义：
>  *      2. 直接在类中赋值定义  
>  * 
>  *  2. 读取方式
>  *      1. this.state.xxx
>  *      2. 解构： {xxx,yyy} = this.state
>  *  3. 设置： 状态的改变不能直接赋值，需要调用 setState方法
>  *      this.setState({
>  *          状态：状态的新值
>  *      })

```js
class App extends Component {
    // 方式一：比较麻烦
    // constructor() {
    //     super()
    //     this.state = {
    //         count: 1,
    //         username: 'atguigu'
    //     }
    // }
    // 方式二：因为方式二也相当于是在构造函数中执行的。
    state = {
        count:1,
        username:'atguigu'
    }
    fun = ()=>{
        // this 永远指向组件的实例。因为相当于是在构造函数中执行的
        console.log(this)
    }
    // 上面的代码，相当于是下面的contructor中的代码
    // constructor(){
    //     super()
    //     this.state = {
    //         count:1,
    //         username:'atguigu'
    //     }
    //     this.fun = ()=>{
    //         console.log(this)
    //     }
    // }

    render() {
        // 使用状态，经常会用解构赋值的方式
        let { count, username } = this.state
        console.log(this);
        return (
            <div>
                <h3>App</h3>
                <button onClick={()=>{
                    // 错误的写法，直接赋值
                    // this.state.count += 1

                    // 正确改变状态的方式： setState
                    this.setState({
                        count:this.state.count + 1
                    })
                }}>点击 + 1</button>
                <div>
                    {this.state.count}
                    {this.state.username}
                </div>
                <div>
                    {count}
                    {username}
                </div>
            </div>
        )
    }
}

export default App
```

#### 4-5-3. 类组件回调函数中的this问题[重要]

> 总结：
>
> 类组件中的事件回调函数：调用者不是组件实例。所以，事件回调函数中，会存在this指向问题。
>    我们希望事件回调函数中的this 能够指向 组件实例。
>    实现方式有三种：
>
>          1. 包裹箭头函数：         因为使用的是render中的this
>          2. bind        ：         bind也是render中的this
>          3. 在class中定义箭头函数：因为使用的是constructor中的this
>          4. 直接在onClick中，定义箭头函数完成业务
>
> ​      推荐使用：两种箭头函数的方式。因为简单
> ​      1. 如果需要传参的时候，使用包裹箭头函数的方式   onClick={()=>this.xxx()}
> ​            2. 如果不需要传参，那么使用class定义箭头函数的方式

```js
import React, { Component } from 'react'

class Count extends Component {

    // 定义状态
    state = {
        num: 0
    }
    // 普通函数的 this指向哪里，取决于调用的时候，是谁调用的，不取决于定义在哪儿
    click1() {
        console.log(this)
        this.setState({
            num: this.state.num + 1
        })
    }

    click2(props){
        console.log(this,props)
        this.setState({
            num: this.state.num + 1
        })
    }
    click3(){
        console.log(this)
        this.setState({
            num: this.state.num + 1
        })
    }

    click4 = ()=>{
        console.log(this)
        this.setState({
            num: this.state.num + 1
        })
    }

    render() {
        let { num } = this.state;
        return (
            <div>
                <h4>Num</h4>
                <p>num的值为： {num}</p>

                <button onClick={function () {
                    // 类组件中，事件的回调函数，this指向是undefined
                    // 说明事件的回调，不是组件实例调用的。
                    console.log(this)
                }}>有问题的按钮1 </button>

                <button onClick={this.click1}>有问题的按钮2 </button>

                <button onClick={()=>this.click2(123123)}>包裹箭头函数</button>

                {/* <button onClick={()=>
                    this.click2()
                }>包裹箭头函数</button> */}

                <button onClick={this.click3.bind(this)}>通过bind绑定</button>


                <button onClick={this.click4}>在class中定义箭头函数</button>

                <button onClick={()=>{
                    this.setState({
                        num:num + 1
                    })
                }}>直接在onClick中定义箭头函数</button>
            </div>
        )
    }
}
export default Count

/**
 *  总结：
 *      类组件中的事件回调函数：调用者不是组件实例。所以，事件回调函数中，会存在this指向问题。
 *      我们希望事件回调函数中的this 能够指向 组件实例。
 *      实现方式有三种：
 *      1. 包裹箭头函数：         因为使用的是render中的this
 *      2. bind        ：         bind也是render中的this
 *      3. 在class中定义箭头函数：因为使用的是constructor中的this
 * 
 *      推荐使用：两种箭头函数的方式。因为简单
 *      1. 如果需要传参的时候，使用包裹箭头函数的方式   onClick={()=>this.xxx()}
 *      2. 如果不需要传参，那么使用class定义箭头函数的方式
 * 
 * 
 */


```

#### 4-5-4. 父组件向子组件传参[props]

> 父组件通过在子组件标签上添加属性的方式，可以向子组件传参。
>
> 如果子组件是函数组件，那么接收的方式就是通过 函数的形参接收
>
> 如果子组件是类组件，那么接收方式是通过特殊属性名 this.props接收

- 父组件

```js
import React, { Component } from 'react'
import { ClassProps, FunProps } from "./components/PropsTest"

class App extends Component {
    // state = {
    //     qiuyan:{
    //         username:'秋燕',
    //         age:19
    //     },
    //     yaning:{
    //         username:'亚宁',
    //         age:20
    //     }
    // }

    render() {
        let qiuyan = {
            username: '秋燕',
            age: 19
        }
        let yaning = {
            username: '亚宁',
            age: 20
        }
        return (
            <div>
                <h3>App根组件</h3>
                <hr />
                <FunProps username={qiuyan.username} age={qiuyan.age}/>

                <FunProps {...qiuyan}/>
                <hr />
                <ClassProps username={yaning.username} age={yaning.age} />

                <ClassProps {...yaning}></ClassProps>
            </div>
        )
    }

}
export default App
```

- 子组件

```js
import React, { Component } from 'react'
class ClassProps extends Component {
    render() {
        // 获取父组件数据 this.props
        console.log('我是类组件，render方法')
        console.log(this.props)
        let { username, age } = this.props
        return (
            <div>
                <h4>类组件</h4>
                <p>用户名：{username}</p>
                <p>年龄: {age}</p>
            </div>
        )
    }
}
function FunProps(props) {
    console.log('我是函数组件，render')
    console.log(props);
    let { username, age } = props
    return (
        <div>
            <h4>函数组件</h4>
            <p>用户名：{username}</p>
            <p>年龄: {age}</p>
        </div>
    )
}
export {
    ClassProps,
    FunProps
}

```

#### 4-5-5. 父组件状态数据传递给子组件

>组件间通信：父组件向子组件通信==》 通过props进行传递

- 父组件

```js
import React, { Component } from 'react'
import { ClassProps, FunProps } from "./components/PropsTest"

class App extends Component {
    state = {
        qiuyan:{
            username:'秋燕',
            age:119
        },
        yaning:{
            username:'亚宁',
            age:120
        }
    }

    render() {
        console.log('我是父组件的render方法')
        let {qiuyan,yaning} = this.state
        return (
            <div>
                <h3>App根组件</h3>
                <h4>姓名：{qiuyan.username}</h4>
                <h4>年龄：{qiuyan.age}</h4>
                <button onClick={()=>{
                    this.setState({
                        qiuyan:{username:qiuyan.username + '-',age:qiuyan.age + 1}
                    })

                }}>更新秋燕的信息</button>
                <hr />
                {/* 父组件通过属性向子组件传参 */}
                <FunProps username={qiuyan.username} age={qiuyan.age}/>
                {/* 可以通过扩展运算批量传参 */}
                <FunProps {...qiuyan}/>
                <hr />
                <ClassProps username={yaning.username} age={yaning.age} />

                <ClassProps {...yaning}></ClassProps>
            </div>
        )
    }

}
/**
 *  当将父组件的状态信息传递给子组件时。
 *  当父组件的状态信息发生改变的时候，接收父组件的子组件，
 *  无论是否使用了父组件的变化状态信息，都会重新调用render，重新渲染
 * 
 *  组件什么时候会重新渲染呢？
 *  1. 自身状态发生改变时，state
 *  2. 接收外部的数据      props 发生变化时，会重新渲染
 *  3. 如果props的数据，来自于父组件的state，那么父组件的state数据发生变化时，相关子组件都要重新渲染
 * 
 */
export default App
```

- 子组件

```js
import React, { Component } from 'react'
class ClassProps extends Component {
    render() {
        // 类组件通过特殊属性 this.props接收父组件传递的数据
        // 获取父组件数据 this.props
        console.log('我是类组件，render方法')
        let { username, age } = this.props
        return (
            <div>
                <h4>类组件</h4>
                <p>用户名：{username}</p>
                <p>年龄: {age}</p>
            </div>
        )
    }
}
function FunProps(props) {
    console.log('我是函数组件，render')
    // 函数组件通过形参接收父组件传过来的数据。常用props
    let { username, age } = props
    return (
        <div>
            <h4>函数组件</h4>
            <p>用户名：{username}</p>
            <p>年龄: {age}</p>
        </div>
    )
}
export {
    ClassProps,
    FunProps
}
```

#### 4-5-6. 使用map返回组件，渲染列表

> map的返回值可以是 组件。所以可以循环组件

```js
import React, { Component } from 'react'
import { ClassProps, FunProps } from "./components/PropsTest"
class App extends Component {
    state = {
        students: [
            {
                id:1,
                username: '秋燕',
                age: 119
            },
            {
                id:2,
                username: '亚宁',
                age: 120
            }
        ]
    }
    render() {
        let { students } = this.state
        return (
            <div>
                <h3>App根组件</h3>
                <hr />
                {/* 组件也可以进行循环遍历 */}
                {students.map(stu => (
                    <FunProps key={stu.id} {...stu}/>
                ))}
                <hr />
                {
                    students.map(stu=>(
                        <ClassProps key={stu.id} {...stu}/>
                    ))
                }
            </div>
        )
    }

}
export default App
```

#### 4-5-7. 函数组件和类组件类型限定[了解]

> 使用 prop-types这个库实现的
>
> 函数组件限定语法：
>
> 函数名.propTypes = {
>
> // username 是字符串类型，并且是必填项
>
> username: PropsTypes.string.isRequired
>
> }
>
> 函数名.defaultProps = {
>
> // 如果不传，默认值age就是1000
>
> age: 1000
>
> }
>
> 类组件语法：
>
> 在类体中定义 静态属性
>
> static propTypes = {
>
> 	 username: PropsTypes.string.isRequired,
>
> ​    	age: PropsTypes.number
>
> }
>
> static defaultProps = {
>
> ​    	age:2000
>
> }

```js
import React, { Component } from 'react'
import PropsTypes from 'prop-types'
class ClassProps extends Component {
    static propTypes = {
        username: PropsTypes.string.isRequired,
        age: PropsTypes.number
    }
    static defaultProps = {
        age:2000
    }
    render() {
        // 类组件通过特殊属性 this.props接收父组件传递的数据
        // 获取父组件数据 this.props
        console.log('我是类组件，render方法')
        let { username, age } = this.props
        return (
            <div>
                <h4>类组件</h4>
                <p>用户名：{username}</p>
                <p>年龄: {age}</p>
            </div>
        )
    }
}
// 定义传入数据的类型
FunProps.propTypes = {
    // username 是字符串类型，并且是必填项
    username: PropsTypes.string.isRequired
}

FunProps.defaultProps = {
    // 如果不传，默认值age就是1000
    age: 1000
}
function FunProps(props) {
    console.log('我是函数组件，render')
    // 函数组件通过形参接收父组件传过来的数据。常用props
    let { username, age } = props
    return (
        <div>
            <h4>函数组件</h4>
            <p>用户名：{username}</p>
            <p>年龄: {age}</p>
        </div>
    )
}
export {
    ClassProps,
    FunProps
}
```

#### 4-5-8. 组件分离

> 1. 组件是 页面结构 + js + css。
> 2. components | pages
> 3. 每一个组件都是一个独立的目录。目录中有一个 jsx文件 和 css样式文件
> 4. export导出组件
> 5. import 导入使用

#### 4-5-9. this.setState 深入

> setState是异步执行的。
>
> 用法：
>
> this.setState({
>
> ​	 	count: 最新结果
>
> })
>
> this.setState(回调函数) 
>
> this.setState((preState)=>{ // preState 是最新的状态值
>
> })
>
> this.setState(回调函数1， 回调函数2)



```js
import React, { Component } from 'react'
import "./App.css"
class App extends Component {
    state = {
        count:1
    }
    render() {
        console.log('render');
        return (
            <div>
                <h3>App根组件</h3>
                <hr />
                <div>{this.state.count}</div>
                <button onClick={()=>{
                    console.log(1)
                    // setState是异步执行的，当执行多次 + 1操作的时候，因为进入队列时，this.state.count 的值都是 1，所以，累加的结果还是2
                    // this.setState({
                    //     count:this.state.count + 1
                    // })
                    // this.setState({
                    //     count:this.state.count + 1
                    // })
                    // this.setState({
                    //     count:this.state.count + 1
                    // })

                    // 
                    this.setState((state)=>{
                        // 最新的状态值
                        console.log('setState')
                        return {
                            count:state.count + 1
                        }
                    })
                    this.setState((state)=>{
                        // 最新的状态值
                        console.log('setState')
                        return {
                            count:state.count + 1
                        }
                    })
                    this.setState((state)=>{
                        // 最新的状态值
                        console.log('setState')
                        return {
                            count:state.count + 1
                        }
                    })
                    console.log(2);
                }}> 增加 </button>
            </div>
        )
    }

}

export default App
```

2. this.setState(回调函数1,回调函数2)

```js
import React, { Component } from 'react'
import "./App.css"
class App extends Component {
    state = {
        count:1
    }
    render() {
        console.log('render');
        return (
            <div>
                <h3>App根组件</h3>
                <hr />
                <div>{this.state.count}</div>
                <button onClick={()=>{
                    console.log(1)
                    
                    this.setState((state)=>{
                        // 最新的状态值
                        console.log('setState')
                        return {
                            count:state.count + 1
                        }
                    },()=>{
                        // 执行时机：state更新之后，render重新渲染，之后执行
                        console.log('我是回调函数2')
                    })

                    
                    console.log(2);
                }}> 增加 </button>
            </div>
        )
    }

}

export default App
```



#### 4-5-10. 子组件向父组件传参

> 步骤：
>
> 1. 在父组件中定义函数，并且定义形参
> 2. 该函数作为属性传递给子组件
> 3. 子组件调用该方法，并将要传递的数据，以实参的形式进行传递

- App.jsx

```js
import React, { Component } from 'react'
import "./App.css"
import Child from './components/Child';
class App extends Component {
    state = {
        count:1
    }
    // 1. 父组件中定义一个方法
    addCount=(num)=>{
        console.log(this);
        this.setState({
            count: this.state.count + num
        })
    }
    render() {
        console.log('render');
        return (
            <div>
                <h3>App根组件</h3>
                <div>{this.state.count}</div>
                <button onClick={()=>this.addCount(5)}>父组件中的按钮</button>
                <hr />
                {/* <Child username={'atguigu'} addCount={this.addCount.bind(this)} /> */}
{/*2. 将该方法以属性的方式传递给子组件，实际传递的是函数的引用地址。要注意this指向的问题。 */}
                <Child username={'atguigu'} addCount={this.addCount} />
            </div>
        )
    }

}

export default App
```

- Child.jsx

```js
import React, { Component } from 'react'
/**
 * 组件向父组件传参
 */
export default class Child extends Component {
    render() {
        console.log(this.props)
        return (
            <>
                <div>Child</div>
                <button onClick={()=>{
            //子组件中调用父组件传过来的函数，并传递数据
                    this.props.addCount(100)
                }}>给组件传递数据</button>
                <div>{this.props.username}</div>
            </>
        )
    }
}

```

- 原理

```js
function father(num) {
    console.log('我是父组件',num)
}
function child(fn){
    fn(5)
}
child(father)
```

#### 4-5-11. props不可直接修改

> props是父组件传递给子组件的数据，都是只读属性，不可以在子组件中直接修改。
>
> 如果想要修改，需要修改数据的来源，修改父组件中的数据源。

- App.jsx

```js
import React, { Component } from 'react'
import "./App.css"
import Child from './components/Child';
class App extends Component {
    state = {
        count: 1,
        username:'atguigu'
    }
    // 父组件中定义一个方法
    addCount = (num) => {
        console.log(this);
        this.setState({
            count: this.state.count + num
        })
    }
    // 修改username
    rename=(newName)=>{
        this.setState({
            username:newName
        })
    }
    render() {
        console.log('render');
        return (
            <div>
                <h3>App根组件</h3>
                <div>{this.state.count}</div>
                <button onClick={() => this.addCount(5)}>父组件中的按钮</button>
                <hr />
                {/* <Child username={'atguigu'} addCount={this.addCount.bind(this)} /> */}
                <Child username={this.state.username} addCount={this.addCount} rename={this.rename}/>
            </div>
        )
    }

}

export default App
```

- Child.jsx

```js
import React, { Component } from 'react'
/**
 * 组件向父组件传参
 */
export default class Child extends Component {
    render() {
        console.log(this.props)
        return (
            <>
                <div>Child</div>
                <button onClick={()=>{
                    this.props.addCount(100)
                }}>给组件传递数据</button>
                <div>{this.props.username}</div>
                <hr />
                <h3>修改props</h3>
                <button onClick={()=>{
                    // props是只读的，不能够直接修改
                    // 我们可以修改数据源，父组件中的数据
                    // this.props.username = '尚硅谷'

                    this.props.rename('尚硅谷')
                }}>修改</button>
            </>
        )
    }
}

```

#### 4-5-12 props.children

> 可以获取组件对标签中的内容。比如：现实Button 组件功能

- App.jsx

```js
import React, { Component } from 'react'
import "./App.css"
import Button from './components/Button';
import Child from './components/Child';
class App extends Component {
    state = {
        count: 1,
        username:'atguigu'
    }
    render() {
        return (
            <div>
                <h3>App根组件</h3>
                <Child>父组件要传递给子组件的结构</Child>
                <Button>登录</Button>
                <Button>注册</Button>
            </div>
        )
    }
}
export default App
```

- Button

```js
import React, { Component } from 'react'
export default class Button extends Component {
  render() {
    return (
      <button>{this.props.children}</button>
    )
  }
}
```



#### 4-5-13 组件间通信

>父组件向子组件传递数据  props
>
>子组件向父组件传递数据  props
>
>兄弟组件通信
>
>祖孙组件通信
>
>目前：借助共同的父组件完成兄弟组件通信
>
>祖孙组件通信： 逐层传递

#### 4-5-14. ref

> ref 可以在react组件中获取到原生dom对象
>
> 方式一： 字符串
>
> ref={'字符串'}
>
> 方式二：回调函数
>
> ref={(el)=>{ }}
>
> 方式三：new React.createRef()

```js
import React, { Component } from 'react'
import "./App.css"
class App extends Component {
    // password是App的属性，以下代码相当于在构造函数中执行的 
    // password = new React.createRef()
    // password = React.createRef()
    
    password = createRef()
    render() {
        return (
            <div>
                <h3>App</h3>
                {/* 方式一：ref 值是字符串，dom会出现在 组件实例的refs属性上 */}
                <div ref={'username'}>我是div元素</div>
                {/* 方式二：ref 值是一个回调函数，回调函数的参数就是 dom元素
                    一般会将获取的dom元素，存储到组件的实例上
                */}
                <input type="text" name="" id="" ref={(el)=>{
                    this.oInput = el
                }}/>
                {/* 方式三：使用 React.createRef方法，创建一个ref对象。将该对象作为 标签ref的值
                    获取的方式，this.对象.current 获取dom元素
                */}
                <input type="password" name="" ref={this.password} />
                <button onClick={()=>{
                    // 即将废弃，不推荐使用
                    console.log(this.refs.username.innerHTML)
                    
                    console.log(this.oInput.value)
                    // 推荐写法：
                    console.log(this.password.current.value)
                }}>获取原生dom</button>
            </div>
        )
    }
}
export default App
```



#### 4-5-15. 受控组件非受控组件

> 受控组件和非受控组件，相对于表单来说
>
> 受控组件的表单值是跟组件的状态绑定的。
>
> 非受控组件的表单值和组件状态不绑定

- 受控组件：

  > 当给表单的value 赋值 组件的状态数据时，那么该表单的值就收到了 react状态数据的控制，变为只读不可修改。如果希望可以修改，那么需要增加onChange处理函数，在函数中改变状态数据的值。

```js
import React, { Component } from 'react'
/**
 *  react form表单研究两个问题：
 *  1. 表单可以显示我们组件中的状态数据
 *  2. 在react组件中，能够获取表单的最新输入的数据
 * 
 */

export default class FormControl extends Component {
    state = {
        username: 'atguigu',
        password: '123',
        sex: 1,
        city: 3,
        // 如果数据是数字，会出现 === 比较跟预期不一样，需要做数据类型转化
        hobby: [1, 4]
    }
    // 函数出现在对象的实例上，如果有多个实例，那么就会出现多个相同的函数
    submit = (e) => {
        // 阻止表单默认的提交行为。
        e.preventDefault()
        console.log('提交数据了')
        console.log('username', this.state.username)
        console.log('password', this.state.password)
        console.log('sex', this.state.sex)
        console.log('city', this.state.city)
        console.log('hobby', this.state.hobby)
    }
    // 优化后代码
    change(e) {
        // 复选框需要进行单独处理，那就要区分，什么时候是复选框？通过type进行判断
        let value = e.target.value
        if (e.target.type === 'checkbox') {
            let index = this.state[e.target.name].findIndex(item => item === e.target.value / 1)
            value = [...this.state[e.target.name]]
            if (index === -1) value.push(e.target.value / 1)
            else value.splice(index, 1)
        }
        this.setState({
            [e.target.name]: value
        })

    }
    // 优化前
    // change(e) {
    //     // 复选框需要进行单独处理，那就要区分，什么时候是复选框？通过type进行判断
    //     console.log(e.target.type)
    //     if (e.target.type === 'checkbox') {
    //         console.log(e.target.value)
    //         // 当前点击复选框的值，在原数组中是否存在？如果存在，那么删除，如果不存在，添加
    //         // 获取当前值在数组中的索引值
    //         let index = this.state[e.target.name].findIndex(item => item === e.target.value / 1)
    //         console.log(index)
    //         let copyHobby = [...this.state[e.target.name]]
    //         if (index === -1) {
    //             // 不在数组中，添加
    //             // 不建议直接操作原状态中的数组，因为数组是引用数据类型 ，地址值是不变的，所以操作数组中的数据不会触发页面更新
    //             // 复制一个一样的,复制后新数组地址跟原来的不一样，所以在setState重新赋值时，地址值改变，会触发页面更新
    //             // 因此，对于引用数据类型的状态，我们都要新建一个，不要操作原来的。
    //             copyHobby.push(e.target.value/1)
    //         }else{
    //             // 删除
    //             copyHobby.splice(index,1)

    //         }
    //         this.setState({
    //             hobby:copyHobby
    //         })

    //     } else {
    //         // change中修改，state中的对应属性
    //         console.log(e.target.name)
    //         this.setState({
    //             [e.target.name]: e.target.value
    //         })
    //     }
    // }
    // obj = {
    //     username:'atguigu'
    // }
    // name = 'username'
    // obj[name]
    // changePassword(e){
    //     console.log(e.target.name)
    //     this.setState({
    //         password:e.target.value
    //     })
    // }
    // 函数会出现在原型上，多个实例也共用同一个函数
    // test(){
    // }
    render() {
        return (
            <div>
                <h3>FormControl</h3>
                {/* 表单提交触发顺序
                    1. 触发 onsubmit 方法，然后提交数据到 action地址，如果没有action或action为空，那么就提交到当前页面的地址
                */}
                <form onSubmit={this.submit}>
                    <p>用户：<input type="text" name="username" value={this.state.username} onChange={this.change.bind(this)} /></p>
                    <p>密码：<input type="password" name="password" value={this.state.password} onChange={this.change.bind(this)} /></p>
                    <p>性别：
                        <input type="radio" name="sex" value={0} checked={this.state.sex / 1 === 0} onChange={this.change.bind(this)} />女
                        <input type="radio" name="sex" value={1} checked={this.state.sex / 1 === 1} onChange={this.change.bind(this)} />男
                    </p>
                    <p>城市：<select name="city" value={this.state.city} onChange={this.change.bind(this)}>
                        <option value={1}>西安</option>
                        <option value={2}>咸阳</option>
                        <option value={3}>宝鸡</option>
                        <option value={4}>渭南</option>
                    </select></p>
                    <p>爱好：
                        <input type="checkbox" name="hobby" value={1} checked={this.state.hobby.includes(1)} onChange={this.change.bind(this)} /> 台球
                        <input type="checkbox" name="hobby" value={2} checked={this.state.hobby.includes(2)} onChange={this.change.bind(this)} /> 打出溜
                        <input type="checkbox" name="hobby" value={3} checked={this.state.hobby.includes(3)} onChange={this.change.bind(this)} /> 憋气
                        <input type="checkbox" name="hobby" value={4} checked={this.state.hobby.includes(4)} onChange={this.change.bind(this)} /> 羽毛球
                    </p>

                    {/* 会导致表单提交的按钮 */}
                    <p><button>提交数据</button></p>
                    {/* <p><input type="submit" value="提交表单数据" /></p> */}
                    {/* <p><button type="submit">提交</button></p> */}

                    {/* 不会被提交 */}
                    {/* <button type="button">提交</button> */}
                </form>
            </div>
        )
    }
}

```

- 非受控组件

  > 通过 ref  获取表单的数据

```js
import React, { Component, createRef } from 'react'
/**
 *  react form表单研究两个问题：
 *  1. 表单可以显示我们组件中的状态数据,非受控组件，显示数据 defaultValue  defaultChecked
 *  2. 在react组件中，能够获取表单的最新输入的数据
 * 
 */
export default class FormControl extends Component {
    state = {
        username: 'atguigu',
        password: '123',
        sex: 1,
        city: 3,
        hobby: [1, 4]
    }
    // 函数出现在对象的实例上，如果有多个实例，那么就会出现多个相同的函数
    submit = (e) => {
        // 阻止表单默认的提交行为。
        e.preventDefault()
        console.log('提交数据了')
        console.log('username', this.formData.username.current.value)
        console.log('password', this.formData.password.current.value)
        console.log('city', this.formData.city.current.value)
        console.log('sex', this.formData.sex.find(item=>item.current.checked).current.value)
        console.log('hobby', this.formData.hobby.filter(item=>item.current.checked).map(item=>item.current.value))

    }

    formData = {
        username: createRef(),
        password: createRef(),
        city: createRef(),
        sex: [createRef(), createRef()],
        hobby: [createRef(),createRef(),createRef(),createRef()]
    }

    render() {
        let { username, password, city, sex,hobby } = this.formData
        return (
            <div>
                <h3>FormOutOfControl</h3>

                <form onSubmit={this.submit}>

                    <p>用户：<input type="text" ref={username} name="username" defaultValue={this.state.username} /></p>
                    <p>密码：<input type="password" ref={password} name="password" defaultValue={this.state.password} /></p>
                    <p>性别：
                        <input type="radio" ref={sex[0]} name="sex" value={0} defaultChecked={this.state.sex === 0} />女
                        <input type="radio" ref={sex[1]} name="sex" value={1} defaultChecked={this.state.sex === 1} />男
                    </p>
                    <p>城市：<select name="city" defaultValue={this.state.city} ref={city}>
                        <option value={1}>西安</option>
                        <option value={2}>咸阳</option>
                        <option value={3}>宝鸡</option>
                        <option value={4}>渭南</option>
                    </select></p>
                    <p>爱好：
                        <input type="checkbox" ref={hobby[0]} name="hobby" defaultChecked={this.state.hobby.includes(1)} value={1} /> 台球
                        <input type="checkbox" ref={hobby[1]} name="hobby" defaultChecked={this.state.hobby.includes(2)} value={2} /> 打出溜
                        <input type="checkbox" ref={hobby[2]} name="hobby" defaultChecked={this.state.hobby.includes(3)} value={3} /> 憋气
                        <input type="checkbox" ref={hobby[3]} name="hobby" defaultChecked={this.state.hobby.includes(4)} value={4} /> 羽毛球

                    </p>
                    <p><button>提交数据</button></p>
                </form>
            </div>
        )
    }
}
```

#### 4-5-16.  todolist练习

> 组件化开发三板斧
>
> 1. 静态页面构建
> 2. 首屏数据渲染
> 3. 动态交互实现

![组件划分](05-React%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0.assets/%E7%BB%84%E4%BB%B6%E5%88%92%E5%88%86.png)

##### 1 静态页面拆分

> 将对应的html 和css 发入对应组件
>
> 需要注意：class===> className.  有可能把 class组件 关键字也变成 className

##### 2. 首屏数据渲染

1. 需要什么数据结构和数据
2. 数据应该在哪个组件定义状态

###### 1. 数据格式？

1. 任务名
2. id
3. 完成与否的标识

```js
todolist 是一个列表数据，所以必须是一个数组。
数组中的每一项 是一个对象 {id:'xxx',title:'任务名',isDone:boolean}

todos = [
    {id:'xxx',title:'任务名',isDone:true},
    {id:'xxx',title:'任务名',isDone:false},
    {id:'xxx',title:'任务名',isDone:false}
]
```

###### 2. 数据放在哪个组件？

> 如果多个组件都要使用同一个状态数据，那么就要做状态提升，将数据放在他们共同的父组件中。

###### 3. 列表数据渲染

- App.jsx

```js
import React, { Component } from 'react'
import './App.css'
import Footer from './components/Footer'
import Header from './components/Header'
import List from './components/List'
export default class App extends Component {
    state = {
        todos:[
            {id:1,title:'吃饭',isDone:false},
            {id:2,title:'睡觉',isDone:true},
            {id:3,title:'打代码',isDone:false}
        ]
    }
    render() {
        let {todos} = this.state
        return (
            <div className="todo-container">
                <div className="todo-wrap">
                    <Header todos={todos}/>
                    <List todos={todos}/>
                    <Footer todos={todos}/>
                </div>
            </div>
        )
    }
}

```

- List.jsx

```js
import React, { Component } from 'react'
import Item from '../Item'
import './index.css'
export default class List extends Component {
    render() {
        let {todos} = this.props
        return (
            <ul className="todo-main">
                {todos.map(todo=>(<Item key={todo.id} todo={todo}/>))}
            </ul>
        )
    }
}

```

- Item

```js
import React, { Component } from 'react'
import './index.css'
export default class Item extends Component {
    render() {
        let {todo} = this.props
        return (
            <li>
                <label>
                    <input type="checkbox" checked={todo.isDone}/>
                    <span>{todo.title}</span>
                </label>
                <button className="btn btn-danger">删除</button>
            </li>
        )
    }
}
```

##### 3. 交互操作-1-添加todo

> 将文本输入的 未完成事件名 发送给父组件App
>
> 子组件向父组件传递数据：
>
> 1. 父组件中定义方法
> 2. 将方法以属性的方式传递给子组件
> 3. 子组件调用方法，并携带参数

- Header.jsx

```js
import React, { Component,createRef } from 'react'
import './index.css'
export default class Header extends Component {
    keywordRef = createRef()
    keydownHandler(e){
        // 判断是否按的是回车键
        if(e.key === 'Enter'){
            let keyword = e.target.value.trim();
            this.props.addTodo(keyword)
            // 清空文本框
            e.target.value = ''
        }
    }
    render() {
        return (
            <div className="todo-header">
                <input ref={this.keywordRef} type="text"  placeholder="请输入你的任务名称，按回车键确认" onKeyDown={this.keydownHandler.bind(this)}/>
            </div>
        )
    }
}

```

- App.jsx

```js
import React, { Component } from 'react'
import './App.css'
import Footer from './components/Footer'
import Header from './components/Header'
import List from './components/List'
export default class App extends Component {
    state = {
        todos:[
            {id:1,title:'吃饭',isDone:false},
            {id:2,title:'睡觉',isDone:true},
            {id:3,title:'打代码',isDone:false}
        ]
    }
    addTodo(keyword){
        console.log('App',keyword)
        // 拼接一个对象 {id:xxx, title:keyword,isDone:false}
        let todo = {
            id:Date.now(),
            title:keyword,
            isDone:false
        }
        // todos是引用数据类型，不要直接操作它
        let copyTodos = [...this.state.todos]
        copyTodos.unshift(todo)
        // 更新状态
        this.setState({
            todos:copyTodos
        })
        
    }
    render() {
        let {todos} = this.state
        return (
            <div className="todo-container">
                <div className="todo-wrap">
                    <Header addTodo={this.addTodo.bind(this)}/>
                    <List todos={todos}/>
                    <Footer todos={todos}/>
                </div>
            </div>
        )
    }
}

```

##### 4-交互-删除todo

> 点击孙组件 item中的删除按钮，删除爷爷组件中对应id的数据。
>
> 孙子组件向爷爷组件传递数据。需要逐层传递
>
> 函数在App.jsx中定义，然后逐层传递给 Item   App===> List ===> Item

- App.jsx

```js
import React, { Component } from 'react'
import './App.css'
import Footer from './components/Footer'
import Header from './components/Header'
import List from './components/List'
export default class App extends Component {
    state = {
        todos:[
            {id:1,title:'吃饭',isDone:false},
            {id:2,title:'睡觉',isDone:true},
            {id:3,title:'打代码',isDone:false}
        ]
    }
    addTodo(keyword){
        console.log('App',keyword)
        // 拼接一个对象 {id:xxx, title:keyword,isDone:false}
        let todo = {
            id:Date.now(),
            title:keyword,
            isDone:false
        }
        // todos是引用数据类型，不要直接操作它
        let copyTodos = [...this.state.todos]
        copyTodos.unshift(todo)
        // 更新状态
        this.setState({
            todos:copyTodos
        })  
    }
    deleteTodo(id){
        // 通过过滤，进行删除
        let copyTodos = [...this.state.todos].filter(todo=>todo.id !== id)
        this.setState({
            todos:copyTodos
        })  
    }
    render() {
        let {todos} = this.state
        return (
            <div className="todo-container">
                <div className="todo-wrap">
                    <Header addTodo={this.addTodo.bind(this)}/>
                    <List todos={todos} deleteTodo={this.deleteTodo.bind(this)}/>
                    <Footer todos={todos}/>
                </div>
            </div>
        )
    }
}

```

- List.jsx

```js
import React, { Component } from 'react'
import Item from '../Item'
import './index.css'
export default class List extends Component {
    
    render() {
        let {todos} = this.props
        return (
            <ul className="todo-main">
                {todos.map(todo=>(<Item deleteTodo={this.props.deleteTodo} key={todo.id} todo={todo}/>))}
            </ul>
        )
    }
}

```

- Item

```js
import React, { Component } from 'react'
import './index.css'
export default class Item extends Component {
    render() {
        let {todo} = this.props
        return (
            <li>
                <label>
                    <input type="checkbox" checked={todo.isDone}/>
                    <span>{todo.title}</span>
                </label>
                <button className="btn btn-danger" onClick={()=>this.props.deleteTodo(todo.id)}>删除</button>
            </li>
        )
    }
}
```

##### 5- checked状态改变

> 思路与删除基本相同

- App.jsx

```js
import React, { Component } from 'react'
import './App.css'
import Footer from './components/Footer'
import Header from './components/Header'
import List from './components/List'
export default class App extends Component {
    state = {
        todos:[
            {id:1,title:'吃饭',isDone:false},
            {id:2,title:'睡觉',isDone:true},
            {id:3,title:'打代码',isDone:false}
        ]
    }
    addTodo(keyword){
        console.log('App',keyword)
        // 拼接一个对象 {id:xxx, title:keyword,isDone:false}
        let todo = {
            id:Date.now(),
            title:keyword,
            isDone:false
        }
        // todos是引用数据类型，不要直接操作它
        let copyTodos = [...this.state.todos]
        copyTodos.unshift(todo)
        // 更新状态
        this.setState({
            todos:copyTodos
        })  
    }
    deleteTodo(id){
        // 通过过滤，进行删除
        let copyTodos = [...this.state.todos].filter(todo=>todo.id !== id)
        this.setState({
            todos:copyTodos
        })  
    }
    // checked
    checkedHandler(todo){
        console.log('app',todo)
        // 因为todo是引用数据类型，所以修改todo的isDone, todos中的对应的todo的isDone数据就已经改变了。
        // 但是不会触发页面重新渲染，因为todos的地址没有变化。所以，我们需要生成一个新的todos，执行setState触发render
        todo.isDone = !todo.isDone
        this.setState({
            todos:[...this.state.todos]
        })
    }
    render() {
        let {todos} = this.state
        return (
            <div className="todo-container">
                <div className="todo-wrap">
                    <Header addTodo={this.addTodo.bind(this)}/>
                    <List todos={todos} deleteTodo={this.deleteTodo.bind(this)} checkedHandler={this.checkedHandler.bind(this)}/>
                    <Footer todos={todos}/>
                </div>
            </div>
        )
    }
}
```

- List.jsx

```jsx
import React, { Component } from 'react'
import Item from '../Item'
import './index.css'
export default class List extends Component {
    
    render() {
        let {todos} = this.props
        return (
            <ul className="todo-main">
                {todos.map(todo=>(<Item deleteTodo={this.props.deleteTodo} checkedHandler={this.props.checkedHandler} key={todo.id} todo={todo}/>))}
            </ul>
        )
    }
}

```

- Item.jsx

```jsx
import React, { Component } from 'react'
import './index.css'
export default class Item extends Component {
    render() {
        let {todo} = this.props
        return (
            <li>
                <label>
                    <input type="checkbox" checked={todo.isDone} onChange={()=>{
                        this.props.checkedHandler(todo)
                    }}/>
                    <span>{todo.title}</span>
                </label>
                <button className="btn btn-danger" onClick={()=>this.props.deleteTodo(todo.id)}>删除</button>
            </li>
        )
    }
}
```

##### 6. 剩余功能实现

- App.jsx

```jsx
import React, { Component } from 'react'
import './App.css'
import Footer from './components/Footer'
import Header from './components/Header'
import List from './components/List'
export default class App extends Component {
    state = {
        todos:[
            {id:1,title:'吃饭',isDone:false},
            {id:2,title:'睡觉',isDone:true},
            {id:3,title:'打代码',isDone:false}
        ]
    }
    // 添加todo
    addTodo(keyword){
        console.log('App',keyword)
        // 拼接一个对象 {id:xxx, title:keyword,isDone:false}
        let todo = {
            id:Date.now(),
            title:keyword,
            isDone:false
        }
        // todos是引用数据类型，不要直接操作它
        let copyTodos = [...this.state.todos]
        copyTodos.unshift(todo)
        // 更新状态
        this.setState({
            todos:copyTodos
        })  
    }
    deleteTodo(id){
        // 通过过滤，进行删除
        let copyTodos = [...this.state.todos].filter(todo=>todo.id !== id)
        this.setState({
            todos:copyTodos
        })  
    }
    // checked
    checkedHandler(todo){
        console.log('app',todo)
        // 因为todo是引用数据类型，所以修改todo的isDone, todos中的对应的todo的isDone数据就已经改变了。
        // 但是不会触发页面重新渲染，因为todos的地址没有变化。所以，我们需要生成一个新的todos，执行setState触发render
        todo.isDone = !todo.isDone
        this.setState({
            todos:[...this.state.todos]
        })
    }
    // 全选非全选功能
    checkAll(checked){
        console.log('app',checked)
        const copyTodos = [...this.state.todos]
        copyTodos.forEach(todo=>todo.isDone = checked)
        this.setState({
            todos:copyTodos
        })
    }
    // 删除已完成
    deleteDone(){
        const copyTodos = this.state.todos.filter(todo=>!todo.isDone)
        this.setState({
            todos:copyTodos
        })
    }
    render() {
        let {todos} = this.state
        return (
            <div className="todo-container">
                <div className="todo-wrap">
                    <Header addTodo={this.addTodo.bind(this)}/>
                    <List todos={todos} deleteTodo={this.deleteTodo.bind(this)} checkedHandler={this.checkedHandler.bind(this)}/>
                    <Footer todos={todos} checkAll={this.checkAll.bind(this)} deleteDone={this.deleteDone.bind(this)}/>
                </div>
            </div>
        )
    }
}

```

- Footer.jsx

```jsx
import React, { Component } from 'react'
import './index.css'
export default class Footer extends Component {
    render() {
        // 获取todos
        let {todos} = this.props
        // 获取一共多少条记录
        let total = todos.length;
        // 统计已完成记录
        let doneNum = todos.reduce((pre,cur)=>pre + cur.isDone,0)
        // 当已完成和总数相等时，要选中
        let checked = total > 0 && total === doneNum
        return (
            <div className="todo-footer">
                <label>
                    <input type="checkbox" checked={checked} onChange={(e)=>{
                        console.log(e.target.checked)
                        this.props.checkAll(e.target.checked)
                    }}/>
                </label>
                <span>
                    <span>已完成 {doneNum}</span> / 全部 {total}
                </span>
                <button className="btn btn-danger" onClick={this.props.deleteDone}>清除已完成任务</button>
            </div>
        )
    }
}

```

##### 7. 本地持久化数据

> 将todos 通过声明周期钩子，进行初始化以及更新
>
> localStorage

```js
// 页面挂载完，初始化数据
    componentDidMount(){
        // localStorage.setItem('todos',JSON.stringify(this.state.todos))
        let initTodos = JSON.parse(localStorage.getItem('todos')) ? JSON.parse(localStorage.getItem('todos')) : []
        // console.log(initTodos,JSON.parse(localStorage.getItem('todos')))
        this.setState({
            todos:initTodos
        })
    }
// 数据变化时，更新数据
    componentDidUpdate(){
        // 将todos变化的数据，写入localStorage
        localStorage.setItem('todos',JSON.stringify(this.state.todos))
    }
```



#### 4-5-17. 高阶组件

> 高阶函数：定义了一个函数，如果该函数的形参是一个函数，或返回的结果是一个函数，那么可以将定义的函数称为高阶函数
>
> 高阶组件：高阶组件是一个函数，该函数接收一个组件，返回一个组件。称该函数为高阶组件。
>
> 高阶组件 又被称称：hoc（higher order component）。
>
> 高阶组件的作用：1- 属性代理  2-反向继承
>
> 高阶组件常被放置到src->hoc->index.jsx,高阶组件的函数名常以with开头
>
> 高阶组件可以将公共的代码进行抽象封装



```js
import { Component } from 'react'
// 高阶组件：1.0
// function WithApp(WC){ // WrappedComponent
//     return WC
// }

// 2- 返回一个类组件，使出入的组件称为子组件
// function WithApp(WC){
//     return class extends Component{
//         render(){
//             return (<WC/>)
//         }
//     }
// }

// 3. 初级属性代理
// function WithApp(WC){
//     return class extends Component{
//         render(){
//             return (<WC username={'atguigu'} age={10}/>)
//         }
//     }
// }

// 属性代理 2.0
// function WithApp(WC){
//     return class extends Component{
//         state = {
//             username:'atguigu',
//             age:10
//         }
//         render(){
//             return (<WC {...this.state}/>)
//         }
//     }
// }

// 属性代理 3.0
// function WithApp(WC,mapState){
//     return class extends Component{
//         state = {
//             ...mapState
//         }
//         render(){
//             return (<WC {...this.state}/>)
//         }
//     }
// }

// function WithApp(mapState) {
//     return function (WC) {
//         return class extends Component {
//             state = {
//                 ...mapState
//             }
//             render() {
//                 return (<WC {...this.state} />)
//             }
//         }
//     }
// }

function WithApp(mapStateToProps) {
    return function (WC) {
        return class extends Component {
            state = {
                ...mapStateToProps()
            }
            render() {
                return (<WC {...this.state} />)
            }
        }
    }
}
export default WithApp
```

- 猫爪老鼠

```jsx
//  hoc/index.js
import { Component } from 'react'

function WithApp(mapStateToProps) {
    return function (WC) {
        return class extends Component {
            state = {
                ...mapStateToProps(),
                x:0,
                y:0
            }
            mousemoveHandler(e){
                this.setState({
                    x:e.clientX,
                    y:e.clientY
                })
            }
            componentDidMount(){
                window.addEventListener('mousemove',this.mousemoveHandler.bind(this))
            }
            componentWillUnmount(){
                window.removeEventListener('mousemove',this.mousemoveHandler.bind(this))
            }
            
            render() {
                return (<WC {...this.state} />)
            }
        }
    }
}
export default WithApp

// -components/HocCat.jsx
import React, { Component } from 'react'
import WithCat from '../hoc/index'
class Cat extends Component {
    
    render() {
        let {x,y} = this.props
        x += 250
        y += 120
        return (
            <img style={{position:'absolute',left:x,top:y}} width={200} height={200} src={require('../assets/images/cat.gif')} alt="" />
        )
    }
}
export default WithCat(()=>{
    return {}
})(Cat)

// -components/HocMouse.jsx
import React, { Component } from 'react'
import WithMouse from '../hoc/index'

class Mouse extends Component {
    
    render() {
        let {x,y} = this.props
        return (
            <img style={{position:'absolute',left:x,top:y}} width={200} height={200} src={require('../assets/images/mouse.gif')} alt="" />
        )
    }
}
export default WithMouse(()=>({}))(Mouse)
// App.jsx
import React, { Component } from 'react'
import HocCat from './components/HocCat'
import HocMouse from './components/HocMouse'
class App extends Component {
    render() {
        return (
            <div>
                <h3>App</h3>
                <HocCat/>
                <HocMouse/>
            
            </div>
        )
    }
}
export default App
```

- 反向继承-统一处理loading

```js
// hoc/HocLoading.jsx
function HocLoading(WC){
    return class extends WC{
        render(){
            if(this.state.isLoading){
                return (<div>页面正在加载中....</div>)
            }
            return super.render()
        }
    }
}

export default HocLoading

// App.jsx
import React, { Component } from 'react'
import axios from 'axios'
import HocLoading from './hoc/HocLoading'
class App extends Component {
    state = {
        isLoading:true,
        items:[]
    }
    async componentDidMount(){
        let res = await axios.get('https://api.github.com/search/users',{
            params:{
                q:'aa'
            }
        })
        let items = res.data.items
    
        this.setState({
            items,
            isLoading:false
        })
    }
    render() {
        return (
            <div>
                
                {this.state.items.map(item=>(<div key={item.id}>{item.login}</div>))}
            </div>
        )
    }
}
export default HocLoading(App)
```

### 4-6. 样式处理

#### 4-6-1. 样式引入位置

1. 全局通用样式：

   1. 位置：public/css/common.css
   2. 引入：public/index.html 使用 link标签引入

   ```html
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="utf-8" />
       <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
       <meta name="viewport" content="width=device-width, initial-scale=1" />
       <meta name="theme-color" content="#000000" />
       <meta
         name="description"
         content="Web site created using create-react-app"
       />
       <title>React App</title>
        <!-- 引入全局通用样式 -->
       <link rel="stylesheet" href="/css/common.css">
     </head>
     <body>
       <div id="root"></div>
     </body>
   </html>
   
   ```

2. 全局通用引入方式2

   1. src/App.css
   2. App.jsx中通过import进行引入

   ```js
   //在 src/App.jsx 文件头部引入
   import "./App.css"
   ```

3. 组件特有样式

   1. 在组件自己的文件夹中，创建css文件

   2. 引入：在组件文件中 通过 import 引入即可

   3. 也可以使用css模块化操作。 

      1. xxx.module.css 文件后缀必须是 .module.css

      2. 引入： import styles from './xxx.module.css'

      3. 使用：

         ```html
         <div className={styles.login}>
             123123
         </div>
         ```

#### 4-6-2. CSS模块化

- child.jsx

```js
/**
 * css模块化的前提是，文件名后缀必须是 xxx.module.css
 *  css 模块化：
 *      将css中定义的样式，导出为 js对象来使用
 * 
 * 
 * 
 */
import styles from './index.module.css'
console.log(styles)
function Child(){
    return (
        <div className={styles.child}>
            <h4 className={styles.fontColor}>Child</h4>
            <p className="my">123123</p>
        </div>
    )
}
export default Child
```

- index.module.css

```css
.child{
    background-color: aquamarine;
}
.fontColor{
    color:blueviolet
}
/* 我不想让他导出为对象，还是使用全局的样式 */
:global(.my){
    color:blue;
    font-size: 100px;
}
```

### 4-7. 图片处理

#### 1. 网络图片

- 直接img 标签 src 上填地址

#### 2. 本地图片

- require
- import

```js
import React, { Component } from 'react'
import "./App.css"
// 使用import 将图片导入成 js变量
import mi from './assets/images/2.webp'

class App extends Component {
   
    render() {
        console.log(require);
        return (
            <div>
                <h3>App根组件</h3>
                {/* 网络图片使用 */}
                <img width={300} src={'https://cdn.cnbj1.fds.api.mi-img.com/mi-mall/053d963a35ffe0654dfc2ac3d20466ee.jpg?w=2452&h=920'} alt="" />

                {/* 本地图片 使用方式1： */}
                <img width={400} src={require("./assets/images/2.webp")} alt="" />
                {/* 本地图片引入 方式2： */}
                <img src={mi} alt=""  width={100}/>

            </div>
        )
    }

}
export default App
```

### 4-7-3 Swipper练习

```js
import React, { Component } from 'react'
import './index.css'
export default class Swipper extends Component {
    state = {
        index:1
    }
    render() {
        let {index} = this.state
        return (
            <div className={'swipperWrapper'}>
                <img src={require(`../../assets/images/${this.state.index}.webp`)} alt="" />
                <span className={'preBtn'} onClick={()=>{
                    index--;
                    if(index < 1){
                        index = 4
                    }
                    this.setState({
                        index
                    })
                }}> &lt; </span>

                <span className={'nextBtn'} onClick={()=>{
                    index++
                    if(index > 4){
                        index = 1
                    }
                    this.setState({
                        index
                    })
                }}> &gt; </span>
            </div>
        )
    }
}
```



### 4-8. 生命周期

> 只在类组件中有，函数组件通过hook，也可以有生命周期的钩子函数
>
> 组件的生命周期：组件从创建到销毁过程。
>
> 整个过程有一些关键节点：在这些节点有钩子函数，可以在钩子函数中，执行一些业务



#### 4-8-1. 挂载阶段

![image-20220901145121368](05-React%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0.assets/image-20220901145121368.png)

> constructor
>
> render
>
> componentDidMount
>
> 只有render函数会被多次调用，其他挂载函数，只执行一次。更新数据也不会重复执行

```js
import React, { Component } from 'react'

export default class Life extends Component {
    
    // 生命周期的钩子函数，跟定义的顺序没有关系
    componentDidMount(){
        /**
         *  虚拟dom 已经转化为了真实dom。执行
         *  在这个周期里：一般我们会执行如下操作
         * 1. 发起ajax请求
         * 2. 订阅消息
         * 3. 启动定时器
         * 
         */
        console.log('3- componentDidMount');
    }
    // 初始化状态，优先与constructor执行
    state = {
        count:1
    }
    constructor(){
        /**
         *  初始化状态
         */
        super()
        console.log('1- constructor');
        // 构造函数，进行状态的初始化
        console.log(this.state.count);
        // 在构造函数中，直接修改状态，没有错误提示。并且可以直接赋值
        // 直接赋值改变状态，不会触发render的渲染。
        // 国通setState函数修改状态，会触发render重新渲染
        // this.state.count = 2
        // 可以对状态二次增加，需要注意，必须之前就已经初始化 state
        this.state.username = 'atguigu'
    }
    render() {
        console.log('2- render');
        console.log(this.state)
        return (
            <div>Life</div>
        )
    }
}
```

#### 父子组件挂载过程

![image-20220901143815701](05-React%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0.assets/image-20220901143815701.png)

#### 4-8-2. 更新阶段

> 什么会触发render函数执行更新？
>
> 1. setState
> 2. newProps 
> 3. forceUpdate()
>
> render
>
> componentDidUpdate

- components/Life.jsx

```js
import React, { Component } from 'react'

export default class Life extends Component {

    // 生命周期的钩子函数，跟定义的顺序没有关系
    componentDidMount(){
        /**
         *  虚拟dom 已经转化为了真实dom。执行
         *  在这个周期里：一般我们会执行如下操作
         * 1. 发起ajax请求
         * 2. 订阅消息
         * 3. 启动定时器
         * 
         */
        console.log('3- child - componentDidMount');
    }
    // 初始化状态，优先与constructor执行
    state = {
        count:1
    }
    constructor(){
        /**
         *  初始化状态
         */
        super()
        console.log('1- child- constructor');
        // 构造函数，进行状态的初始化
        console.log(this.state.count);
        // 在构造函数中，直接修改状态，没有错误提示。并且可以直接赋值
        // 直接赋值改变状态，不会触发render的渲染。
        // 国通setState函数修改状态，会触发render重新渲染
        // this.state.count = 2
        // 可以对状态二次增加，需要注意，必须之前就已经初始化 state
        this.state.username = 'atguigu'
    }
    render() {
        console.log('2- child - render');
        console.log(this.state)
        return (
            <div>
                <h3>Life</h3>
                <div>App-state-money: {this.props.money}</div>
                <button onClick={()=>{
                    // 只要执行 setState就触发更新生命周期，不管值是否改变
                    this.setState({
                        count: 2
                    })
                }}>更新state {this.state.count}</button>

                <button onClick={()=>{
                    this.forceUpdate()
                }}>强制更新</button>
            </div>
        )
    }

    componentDidUpdate(){
        console.log('Child - componentDidUpdate')
    }
}

```

- App.jsx

```js
import React, { Component } from 'react'
import "./App.css"
import Life from './components/Life'
class App extends Component {
    state = {
        money:100
    }
    constructor(){
        super()
        console.log('1- App- constructor')
    }
    componentDidMount(){
        console.log('3- App - componentDidMount')
    }
    render() {
        console.log('2- App - render')
        return (
            <div>
                <h3>App</h3>
                
                <button onClick={()=>{
                    this.setState({
                        money:this.state.money + 100
                    })
                }}>更新 money</button>
                <hr />
                <Life money={this.state.money}/>
            </div>
        )
    }

}
export default App
```

#### 4-8-3. 卸载

> componentWillUnmount
>
> 组件卸载前执行
>
> 清除定时器
>
> 取消订阅

```js
import React from 'react'
import { PureComponent } from 'react';

/**
 *  纯组件：
 *  1. 如果组件用到的 外部数据和自身状态数据，没有变化的时候，那么render函数不执行
 * 
 */
export default class Count extends PureComponent {
    /**
     * 组件即将卸载前执行
     * 1. 清除定时器
     * 2. 取消订阅
     * 
     */
    componentWillUnmount(){
        console.log('componentWillUnmount')
    
    }
    render() {
        console.log('count -render');
        return (
            <div>
                <h3>Count</h3>
                <div>{this.props.count}</div>
            </div>
        )
    }
    /**
     * 组件挂载后执行
     * 1. 设置定时器
     * 2. 订阅消息
     * 3. 发送ajax请求获取数据
     */
    componentDidMount(){
        
    }
}

```

- 电子时钟

```js
import React, { Component } from 'react'
import moment from 'moment'
export default class Timer extends Component {
    state = {
        nowTime:''
    }
    componentDidMount(){
        // 将timer放在组件实例的属性timer上
        this.timer = setInterval(()=>{
            console.log('111')
            this.setState({
                nowTime: moment().format('YYYY-MM-DD HH:mm:ss')
            })
        },1000)
    }
    componentWillUnmount(){
        clearInterval(this.timer)
    }
    render() {
        return (
            <div>
                <h3>Timer</h3>
                <div>{this.state.nowTime}</div>
            </div>

        )
    }
}
```



#### 4-8-4. 纯组件 PureComponent

> 当组件中使用到的数据，外部数据 props和 state自身数据，没有变化的时候。render不执行
>
> 继承 PureComponent 即可。实现的效果跟 shouldComponentUpdate 类似

- shouldComponentUpdate 

```js
/**
     * 控制render函数的执行
     * 
     * @param {*} nextProps 目标 props值
     * @param {*} nextState 目标 state值
     * @returns boolean  
     */
    shouldComponentUpdate(nextProps, nextState) {

        console.log('shouldComponentUpdate')
        console.log('this.props', this.props)
        console.log('nextProps', nextProps)

        console.log('nextState',nextState)
        console.log('this.state', this.state);
        // 返回true 更新，触发后续的render方法
        // 返回false 不更新，后续render方法不执行
        for (let attr in this.props) {
            if (this.props[attr] !== nextProps[attr]) {
                return true
            }
        }
        // 判断 state
        for(let attr in this.state){
            if(this.state[attr] !== nextState[attr]){
                return true
            }
        }
        return false;
    }
```

- 纯组件

```js
import React from 'react'
import { PureComponent } from 'react';
/**
 *  纯组件：
 *  1. 如果组件用到的 外部数据和自身状态数据，没有变化的时候，那么render函数不执行
 * 
 */
export default class Count extends PureComponent {
    render() {
        console.log('count -render');
        return (
            <div>
                <h3>Count</h3>
                <div>{this.props.count}</div>
            </div>
        )
    }
}
```



#### 4-8-5. 错误边界

> 当组件挂载时发生错误，并且错误不是异步的。可以对错误进行检测。不显示错误组件，并且不影响其他组件的渲染
>
> 只能应用于子组件
>
>      * 
>           * @returns  当有错误发生的时候执行，返回值会合并到state中去.如果属性名一致，就会发生覆盖
>           *  常常和componentDidCatch 配合使用，生成错误日志，方便后续版本迭代
>                *  必须要有state
>                *  getDerivedStateFromError：处理错误不是万能的。只能处理render中的同步错误。
>                     *  1. 异步错误无法处理
>                  2. 事件回调中的错误也无法处理
>                  3. 只能处理子组件中的错误

- App.jsx

```js
import React, { Component } from 'react'
import "./App.css"
import ErrorBoundary from './components/ErrorBoundary'
class App extends Component {
    state = {
        isError:false
    }
    /**
     * 
     * @returns  当有错误发生的时候执行，返回值会合并到state中去.如果属性名一致，就会发生覆盖
     *  常常和componentDidCatch 配合使用，生成错误日志，方便后续版本迭代
     *  必须要有state
     *  getDerivedStateFromError：处理错误不是万能的。只能处理render中的同步错误。
     * 1. 异步错误无法处理
     * 2. 事件回调中的错误也无法处理
     * 3. 只能处理子组件中的错误
     * 
     */
    static getDerivedStateFromError(){
        console.log('getDerivedStateFromError')
        // return {
        //     count:100
        // }
        return {
            isError:true
        }
    }
    componentDidCatch(error){
        console.log('componentDidCatch')
        console.log(error);
        // 发送ajax请求，将错误信息返回给服务器，生成错误日志
    }

    render() {
        console.log(this.state);
        let {isError} = this.state;
        return (
            <div>
                <h3>App</h3>
                <div>其他组件内容</div>
                {isError ? <div>组件渲染失败</div> : <ErrorBoundary/>}
                <div>其他组件内容</div>
            </div>
        )
    }

}

export default App
```

- ErrorBoundary

```js
import React, { Component } from 'react'

export default class ErrorBoundary extends Component {
    render() {

        return (
            <div>
                <h3>错误边界</h3>
                <button onClick={() => {
                    let a = 1;
                    a()
                }}>点击</button>
            </div>

        )
    }
}
```

## 五、函数组件

### 1. Hook

#### 1-1. useState【重要】

> 用来给函数组件定义状态
>
> 语法：
>
> let [ 状态, 设置状态函数] = useState(初始值)

```jsx
import React  from 'react'
import { useState } from 'react'

/**
 * Hook 函数都是以 useXxx 命名的
 * Hook 函数都在 react对象下面
 * useState 就是让函数组件也有自身的状态
 * 
 * 
 */
export default function App() {
    console.log(this) // 函数组件中的this 都是undefined。
    console.log('App render')

    // 返回值是[状态值,设置状态值的方法] =  useState(初始值)
    // let res = useState(100)
    // console.log(res);
    let [count, setCount] = useState(100)

    // 设置多个状态？ useState可以执行多次
    let [msg, setMsg] = useState('atguigu')
    return (
        <div>
            <h3>App</h3>
            <div>{count}</div>
            <div>{msg}</div>
            <button onClick={()=>{
                // setCount(最终值)
                setCount(count + 1)
                // 执行setXxx函数的时候，会导致函数组件重新调用
                // useState 会缓存数据，初始化数据只执行一次，后续会读取缓存中的最新值
            }}> + </button>
        </div>
    )
}
```

###### setXxx 设置值的两种方式

1. setXxx(最终值)
2. setXxx((最新的状态数据)=>{  return 最终值 })

```jsx
import React from 'react'
import { useEffect } from 'react'
import { useState } from 'react'
export default function App() {
    let [msg,setMsg] = useState('atguigu')

    useEffect(()=>{// componentDidMount

        setTimeout(()=>{
            // setMsg 直接设置时，由于闭包，取不到最新设置的值
            // setMsg(msg + '---')

            // 让set函数能够取到最新变化的值
            setMsg((msg)=>{
                //返回值就是设置的值
                return msg + '---'
            })
        },2000)

    },[])
    return (
        <div>
            <h3>{msg}</h3>
            <button onClick={()=>{
                setMsg(msg + '+')
            }}>修改msg</button>
        </div>
    )
}
```

#### 1-2. useEffect【重要】

> 在函数组件中模拟类组件的生命周期
>
> 1. componentDidMount ： 
> 2. componentDidUpdate
> 3. componentWillUnmout

```js
import React, { useState } from 'react'
import { useEffect } from 'react'
import Child from './components/Child'

export default function App() {
    let [count, setCount] = useState(0)
    let [msg,setMsg] = useState('atguigu')
    // useEffect(()=>{
    //     // 回调函数的外围函数，模拟的是组件挂载
    //     console.log('挂载')
    //     let timerid = setInterval(()=>{
    //         console.log(123)
    //     },2000)
    //     return ()=>{
    //         // return 内层函数，模拟的是卸载
    //         console.log('卸载')
    //         clearInterval(timerid)
    //     }
    // })

    useEffect(()=>{
        // 回调函数的外围函数，模拟的是组件挂载
        console.log('挂载 或更新 ')
        
    })

    // useEffect(()=>{
    //     console.log('挂载或更新')
    // },[])

    // useEffect(()=>{ // componentDidMount + componentDidUpdate [count]
    //     console.log('挂载或更新')
    // },[count])

    // useEffect(()=>{ // componentDidMount + componentDidUpdate [count,msg] count状态和msg状态的变化都检测
    //     console.log('挂载或更新')
    // },[count,msg])

    /**
     *  useEffect(回调函数) : 没有第二个参数。那么外层函数就是模拟挂载，内层函数写在return中，模拟的是卸载
     *  useEffect(回调函数) ：没有第二个参数，并且没有return 内部函数。模拟挂载 + 所有 state和props的监听更新
     *  useEffect(回调函数,[]) : 模拟挂载
     *  useEffect(回调,[state,props]) : 挂载 + 更新 （state和props数据都可以检测）
     * 
     *  用的最多的两个阶段： 挂载和卸载=>90% 。更新用的较少
     * 
     */
    return (
        <div>
            <div>{count}</div>
            <div>{msg}</div>
            <button onClick={()=>{
                setCount(count + 1)
            }}> + </button>
            <button onClick={()=>{
                setMsg(msg + '+')
            }}> 修改 msg </button>

            <hr />
            {/* {count===0 && <Child />} */}
            <Child count={count}/>
        </div>
    )
}


// 子组件 Child.jsx
import React from 'react'
import { useEffect,useState } from 'react'

export default function Child(props) {
    let [c,setC] = useState(0)
    // useEffect(()=>{

    //     return ()=>{
    //         console.log('卸载')
    //     }
    // })

    // useEffect(()=>{
    //     console.log('Child子组件 挂载或更新')
    //     // 自身状态和外部传过来的状态都可以检测
    // },[props])

    useEffect(()=>{
        console.log('挂载 + 所有state 或 所有 props更新都会触发')
    })

    return (
        <div>
            <h3>Child</h3>
            子组件自己的C ：{c}
            <div>父组件传过来的count: {props.count}</div>
            <button onClick={()=>{
                setC(c+1)
            }}>改变 C</button>
        </div>
    )
}
```



#### 1-3 useRef【重要】

> 可以获取 真实dom元素
>
> 1. 引入 
>
> import {useRef} from 'react'
>
> inputRef = useRef()
>
> 2. 绑定
>
> <input ref={inputRef } />
>
> 3. 获取
>
> inputRef .current

```js
import React from 'react'
import { useRef } from 'react'
import FormControl from './components/FormControl';

export default function App() {
    // 1. 创建 ref对象
    let username = useRef()
    console.log(username);
    return (
        <div>
            {/* 绑定dom元素 */}
            <input type="text" ref={username} name="" id="" />
            <button onClick={()=>{
                // 获取dom元素
                console.log(username.current.value)
            }}>获取</button>

            <hr />

            <FormControl/>
        </div>
    )
}

```

##### 1-3-1 高阶函数解决受控组件 onchange函数多次定义问题

```js
import React from 'react'
import { useState } from 'react'

export default function FormControl() {
    let [username,setUsername] = useState('atguigu')
    let [password,setPassword] = useState('123')

    // function changeHandler(e){
    //     console.log(e.target.value);
    //     setUsername(e.target.value)
    // }

    // function changePasswordHandler(e){
    //     setPassword(e.target.value)
    // }

    /**
     *  高阶函数处理 onChange
     * 
     */
    function changeHandler(fn){
        return function (e){
            fn(e.target.value)
        }
    }

    return (
        <div>
            <h3>FormControl</h3>
            <form action="">
                <p>姓名： <input type="text" name="username" value={username} onChange={changeHandler(setUsername)}/>    </p>
                <p>密码： <input type="password" name="password" value={password} onChange={changeHandler(setPassword)}/></p>
            </form>
        </div>
    )
}
```

##### 1-3-2. 单独模拟 componentDidUpdate

```js
let username = useRef()
	//创建一个标识变量
    let firstRef = useRef(true)
    
    useEffect(()=>{
        // 有没有办法单独模拟 didUpdate     
        // didMount + count didUpdate

        // 通过 useRef 作为标识，单独模拟 didUpdate生命周期
        if(firstRef.current){
            // didMount进入时，直接返回，并将标识变量改为 false
            firstRef.current = false;
            return
        }
        // 以下代码之后 count 状态改变时，才会执行，实现了单独模拟 didUpdate
        console.log('count update')

    },[count])
```

##### 1-3-3. 非受控组件

```jsx
import React from 'react'
import { useRef } from 'react';
import { useState } from 'react';
export default function OutForm() {
    let [username,setUsername] = useState('atguigu')
    let [password,setPassword] = useState('123')
    let usernameRef = useRef()
    let passwordRef = useRef()
    function submitHandler(e){
        e.preventDefault()
        console.log('username', usernameRef.current.value)
        console.log('password', passwordRef.current.value)
    }
    return (
        <div>
            <h3>非受控组件</h3>
            <form onSubmit={submitHandler}>
                <p>姓名： <input type="text" ref={usernameRef} name="" defaultValue={username} /></p>
                <p>密码： <input type="text" ref={passwordRef} name="" defaultValue={password} /></p>
                <button>提交</button>
            </form>
        </div>
    )
}
```

##### 1-3-4 ref深入

```js
import React from 'react'
import { useRef } from 'react'


class ClassCom extends React.Component {
    state = {
        test: 123123
    }
    render() {
        return <div>我是类组件</div>
    }
}

// function FnCom(){
//     return <div>我是函数组件</div>
// }

/**
 *  ref 就是函数组件标签身上绑定的ref
 */
const FnCom = React.forwardRef(function FnCom(props,ref) {
        console.log(props, ref)
        return <div>我是函数组件</div>
    }
)

export default function App() {
    let testRef = useRef()
    let classRef = useRef()
    let fnRef = useRef(213)
    return (
        <div>
            <h3 ref={testRef}>App</h3>
            <button onClick={() => {
                console.log(testRef)
                console.log(classRef)
                console.log(fnRef)

            }}> 获取 ref </button>
            {/* ref 用在类组件上，可以获取类组件实例 */}
            <ClassCom ref={classRef} />
            <hr />
            {/* ref 本身不能应用在函数组件身上，如果一定要用，那么需要 使用 React.forwardRef() */}
            <FnCom ref={fnRef} username={'atguigu'}/>

        </div>
    )
}
/**
 *  ref
 *     1. 给普通标签使用，获取dom元素
 *     2. 可以作为标识，模拟 componentDidUpdate
 *     3. 给类组件使用，获取类组件的实例
 *     4. 给函数组件使用，报错，可以通过  React.forwardRef，对函数组件进行处理，也可以让函数组件绑定ref
 * 
 * 
 */

```

#### 1-4. useContext【重要】

> 可以收到祖先组件挂载到context上的数据
>
> 1. 先定义一个 context
> 2. context.Provider 组件上  value={数据}
>
> 3. 孙组件通过 useContext接收数据

- context.js

```js
import React from 'react'
export default React.createContext()
```

- App.jsx

```js
import React from 'react'
import Father from './components/Father'
import context from './context'
/**
 * 祖先组件向子孙组件传输数据
 * 
 */
export default function App() {
    const  {Provider} = context
    return (
        <Provider value={{name:'atguigu'}}>
            <div>
                <h3>App</h3>
                <Father />
            </div>
        </Provider>
    )
}

```

- Father.jsx

```js
import React, { useContext } from 'react'
import Son from './Son'
import context from '../context'
export default function Father() {
    let data = useContext(context)
    console.log('father', data)
    return (
        <div>
            <h3>Father</h3>
            <Son />
        </div>
    )
}
```

#### 1-3-5. useCallback【次要】

> 作用:  可以缓存函数，执行的结果是返回一个新的函数。
>
> 1. useCallback(回调) ： 不会缓存
> 2. useCallback(回调,[]) : 缓存函数
> 3. useCallback(回调,[监视数据]) 监视数据发生变化的时候，重新创建。监视数据不发生变化，使用缓存函数

```js
import React from 'react'
import { useCallback } from 'react'
import { useState } from 'react'

export default function App() {
    let [count, setCount] = useState(0)
    // 每次更新，都会创建clickHander函数，希望将函数缓存起来
    // useCallBack作用就是，缓存函数，避免多次创建
    // 执行的结果会返回一个新的函数
    // function clickHander(){
    //     setCount(count + 1)
    // }

    // useCallback(函数,[])
    // 如果没有传第二个参数，那么函数还是会被重复创建
    // const clickHander = useCallback(function () {
    //     setCount(count + 1)
    // })
    
    // 传递第二个参数，并且是一个空数组，那么就可以缓存当前函数
    // const clickHander = useCallback(function () {
    //     setCount(count + 1)
    // },[])

    // 方案一：增加监听,当count数据发生变化的时候，不使用缓存的函数，重新创建
    // const clickHander = useCallback(function () {
    //     setCount(count + 1)
    // },[count])

    // 方案二：使用 回调函数，获取最新的状态值，进行累加
    const clickHander = useCallback(function () {
        setCount((count)=>{
            return count + 1
        })
    },[])
    return (
        <div>
            {count}
            <button onClick={clickHander}>加1</button>
        </div>
    )
}

```

#### 1-3-6. React.memo【次要】

> 类似于纯组件，如果自身状态和外部数据没有发生变化的时候，不会进行重新渲染

- App.jsx

```js
import React, { PureComponent } from 'react'
import Test from './components/Test'

// PureComponent  Component

function getRandomIntInclusive(min, max) {
    min = Math.ceil(min)
    max = Math.floor(max)
    return Math.floor(Math.random() * (max - min + 1)) + min //含最大值，含最小值
}
export default class App extends React.Component {
    state = {
        count:0
    }
    render(){
        console.log('App render')
        return (
            <div>
                <h4>App</h4>
                <div>App---count: {this.state.count}</div>
                <button onClick={()=>{
                    this.setState({
                        count:getRandomIntInclusive(1,2)
                    })
                }}>取值</button>
                <hr />
                <Test count={this.state.count}/>
            </div>
        )
    }
}
```

- Test.jsx

```js
import React from 'react'
import { useState } from 'react'
function Test({count}) {
    /**
     * useState 已经做了性能优化，当状态值每次设置相同时，不会进行render
     * 但是：state是组件自身的状态，它可以控制，如果是外部传过来的数据，也不发生变化，会不会重新渲染呢？
     * 
     * 无法对props中的数据进行优化，可以使用 React.memo,类似于 类组件中的纯组件 PureComponent 的作用
     * 
     */
    let [msg, setMsg] = useState('xxx')
    console.log('Test render');
    return (
        <div>
            <div>{msg}</div>
            <div>Test--- count: {count}</div>
            <button onClick={()=>{
                setMsg('哈哈哈')
            }}>哈哈哈</button>
        </div>
    )
}
export default React.memo(Test)
```

#### 1-3-7. useMemo【次要】

> 作用：缓存计算结果。

```js
import React,{useState} from 'react'
import { useMemo } from 'react';
export default function App() {
    const [count, setCount] = useState(1);
    const [val, setValue] = useState('');

    // 未改造前
    // function expensive() {
        // console.log('compute');
        // let sum = 0;
        // for (let i = 0; i < count * 100; i++) {
        //     sum += i;
        // }
        // return sum;
    // }

    /**
     *  useMemo 缓存结果。如果count 发生变化的时候，函数才会重新调用
     */
    let expensive = useMemo(()=>{
        console.log('compute');
        let sum = 0;
        for (let i = 0; i < count * 100; i++) {
            sum += i;
        }
        return sum;
    },[count])

    return <div>
        <h4>{count}-{val}-{expensive}</h4>
        <div>
            <button onClick={() => setCount(count + 1)}>+c1</button>
            <input value={val} onChange={event => setValue(event.target.value)} />
        </div>
    </div>;
}
```

#### 1-3-8. useImperativeHandle【了解】

> 配合 React.forwardRef 实现功能。
>
> 使父组件获得操作子组件 Dom元素的部分功能【完全由子组件自行控制的】

- App.jsx

```js
import React from 'react'
import { useRef } from 'react'
import Test from './components/Test'

/**
 * 需求：
 *  1. 在App组件中，获得Test子组件的dom元素
 *      forwardRef 可以实现
 * 
 *  2. 问题是，父组件权限太大了。如果子组件是第三发开发者开发的组件，他其实并不想让使用者随便改。它希望能够控制给你开发哪些权限。
 * 
 *  
 */
export default function App() {
    let appRef = useRef('app')
    return (
        <div>
            <h3>App</h3>
            <button onClick={()=>{
                console.log(appRef)
                appRef.current.bg()
                appRef.current.font()
                // appRef.current.style.backgroundColor = 'red'
            }}>获取子组件dom</button>
            <Test ref={appRef}/>
        </div>
    )
}
```

- Test.jsx

```js
import React ,{forwardRef, useImperativeHandle,useRef} from 'react'

function Test(props,ref) {
    let testRef = useRef()
    /**
     *  第一个参数是ref对象，第二个参数是一个函数，函数返回的对象，会挂载到第一个参数ref的current属性上
     */
    useImperativeHandle(ref,()=>({
        // 在内部，通过内部ref，操作子组件的dom元素。
        // 开放可控的功能给父组件使用
        bg:()=>{
            testRef.current.style.backgroundColor = 'red'
        },
        font:()=>{
            testRef.current.style.fontSize = '50px'
        }
    }))
    console.log(ref)
    return (
        <div ref={testRef}>Test</div>
    )
}
export default forwardRef(Test)
```

#### 1-3-9. useLayoutEffect【了解】

> useLayouEffect 与 useEffect 功能一致。
>
> useEffect不会阻塞js后续代码的执行
>
> useLayoutEffect会阻塞，直到里面的代码执行完毕，才会执行后续代码

```js
import React, { useEffect, useLayoutEffect, useRef } from 'react'
import TweenMax from 'gsap' // npm i gsap@3.7.0
import './App.css'

const App = () => {
    const REl = useRef(null)
    useEffect(() => {
        /*下面这段代码的意思是当组件加载完成后,在0秒的时间内,将方块的横坐标位置移到600px的位置*/
        TweenMax.to(REl.current, 0, { x: 600 })
    }, [])
    return (
        <div className="animate">
            <div ref={REl} className="square">
                square
            </div>
        </div>
    )
}
export default App
```

#### 1-3-10. useDebugValue【了解】

> 自定义hook时，在开发者工具中有一个标识

- useFriendStatus

```js
import { useState, useDebugValue } from 'react'
function getRandomIntInclusive(min, max) {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min + 1)) + min //含最大值，含最小值
}

export default function useFriendStatus() {
  const [isOnline, setIsOnline] = useState(null)

  setTimeout(() => {
    let result = getRandomIntInclusive(0, 1)
    console.log(result)
    setIsOnline(result ? 'online' : 'offline')
  }, 1000)

  // 在react开发者工具中的这个 Hook 旁边显示标签
  // "FriendStatus: Online"
  useDebugValue(isOnline)

  return isOnline
}
```

- App.jsx

```js
import React from 'react'
import useFriendStatus from './hook/useFriendStatus'
export default function App() {
    useFriendStatus()
    return (
        <div>App</div>
    )
}

```

#### 1-3-11. useId

> 生成一个唯一值

#### 1-3-12. useTransition

> 可以将任务设置为非紧急任务，使得紧急任务可以优先渲染。性能优化

#### 1-3-13. useDeferredValue

> 设置一个延时值，通过延时值改变影响的组件，会延时渲染。也是性能优化

### 2. Hook使用规则【重要】

> 1. 不能再类组件和普通函数中使用
> 2. 只能在函数组件和其他hook【第三方|自定义】中使用
> 3. 必须在第二条的顶级作用域中使用

```jsx
import React, { Component, useState } from 'react'

/**
 *  Hook 使用的原则：
 *  1. hook函数不能够在类组件中使用，也不能够在普通函数中使用。
 *  2. 只能在函数组件中使用，或者是在其他的hook函数【第三方的hook，自定义hook】中使用
 *  3. hook 函数只能在顶级作用域中使用
 */

// 在类组件中也无法使用
export default class App extends Component {
    getCount(){
        let [count,setCount] = useState()
    }
    
    // if(true){
    // //    {}会形成块作用域，导致 hook不在顶级作用域中
    //    let [count,setCount] = useState()
    // }

    // for(let i=0;i<100;i++){
    //     let [count,setCount] = useState()
    // }

    // 不能在普通函数中使用
    // function fn(){
        // let [count,setCount] = useState()
    // }

    render(){
        return (
            <div>App</div>
        )
    }
}

```

### 3. 自定义hook【次要】

> hook 其实就一个函数，是函数名特殊的函数。 函数名必须是以 useXxx

```js
import { useEffect, useState } from "react";

/**
 *  自定义hook，
 *  1. react区分你是普通函数还是自定义hook，就看你的函数名是否是use开头，是否是驼峰命名 useXxx
 *  2. 使用，就当普通函数用即可
 */
export default function usePosition(){
    // 说明 hook可以在其他hook中使用 【自定义hook，第三方hook使用】
    let [x,setX] = useState(0)
    let [y,setY] = useState(0)
    function mousemoveHandler(e){
        setX(e.clientX)
        setY(e.clientY)
    }
    useEffect(()=>{
        // 挂载阶段设置监听
        window.addEventListener('mousemove',mousemoveHandler)
        return ()=>{
            // 卸载阶段取消事件监听
            window.removeEventListener('mousemove',mousemoveHandler)
        }
    },[])
    return {
        x:x,
        y:y
    }
}
```



### 4. 组件通信之 pubsub【重要】

> 可以实现任意组件间通信
>
> 订阅消息 [ componentDidMount ]
>
> let token = PubSub.subscribe('消息名', (topic, data)=>{
>
> })
>
> 发布消息
>
> PubSub.publish(消息名, 数据)
>
> 取消订阅 【ComponentWillUnmount】
>
> PubSub.unsubscribe(消息名)
>
> PubSub.unsubscribe(消息id) 推荐
>
> 

- App.jsx

```js
import React from 'react'
import PubSub from 'pubsub-js'

import { useEffect } from 'react'
import Father from './components/Father'
import Son from './components/Son'
import Son2 from './components/Son2'

/**
 * 祖先组件向子孙组件传输数据
 * 
 */
export default function App() {
    /**
     * 接收数据的一方，需要订阅消息！
     * 订阅消息的代码写在哪里？
     */
    let tokenXxx
    useEffect(() => {
        // 挂载时订阅消息                     消息名  数据
        tokenXxx = PubSub.subscribe('xxx', (topic, data) => {
            // 该回调函数，当有xxx消息发布的时候，会执行
            console.log(topic, data)
        })
        // 订阅yyy
        let tokenYYY = PubSub.subscribe('yyy', (topic, data) => {
            console.log('App 收到 yyy', data)
        })
        return () => {
            //组件卸载时要取消订阅
            PubSub.unsubscribe(tokenXxx)
            PubSub.unsubscribe(tokenYYY)
        }
    }, [])
    return (
        <div>
            <h3>App</h3>
            <button onClick={() => {
                PubSub.unsubscribe(tokenXxx)
            }}>取消 Son xxx消息 </button>

            <button onClick={() => {
                PubSub.unsubscribe('xxx')
            }}>取消 所有的 xxx消息 </button>

            <Father />

            <Son />
            <Son2 />
        </div>
    )
}
```

- Son.jsx

```js
import React from 'react'
import PubSub from 'pubsub-js'
export default function Son() {
    return (
        <>
            <div>Son</div>
            <button onClick={()=>{
                PubSub.publish('xxx','Son data')
            }}>发布消息 xxx </button>
        </>
    )
}

```

- Son2.jsx

```js
import React from 'react'
import { useEffect } from 'react'
import PubSub from 'pubsub-js'
export default function Son2() {
    useEffect(() => {
        let tokenXxx = PubSub.subscribe('xxx', (topic, data) => {
            console.log('Son2接收', data)
        })
        return () => {
            PubSub.unsubscribe(tokenXxx)
        }
    }, [])
    return (
        <>
            <div>Son2</div>
            <button onClick={()=>{
                PubSub.publish('yyy','我是Son2 yyy 数据')
            }}>发布 yyy 消息</button>
        </>
    )
}
```

### 5. protal

> 可以改变组件渲染的位置
>
> 应用：模态框。模态框每个页面都需要，放入父组件中，可能会受到当前组件样式的影响，所以将其独立出去

- public->index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta name="description" content="Web site created using create-react-app" />
    <title>React App</title>
    <link rel="stylesheet" href="%PUBLIC_URL%/css/bootstrap.css">
</head>

<body>
    <div id="root"></div>
    <div id="container"></div>
</body>

</html>
```

- App.jsx

```jsx
import React from 'react'
import Test from './components/Test'
export default function App() {
    return (
        <div>
            <h3>App</h3>
            <hr />
            <Test/>
        </div>
    )
}

```

- Test.jsx

```js
import React from 'react'
// react 17  createProtal 在这里
import ReactDOM from 'react-dom'
// react 18
// import ReactDOM from 'react-dom/client'
export default function Test() {
    return ReactDOM.createPortal(<div>Test</div>,document.getElementById('container'))
}
```

## 六、React-Router-Dom 路由

> 路由：就是一一对应的一套规则
>
> 后端路由：地址栏url 与网络资源一一对应的规则
>
> 前端路由：地址栏url 与 视图[组件]一一对应的规则
>
> react路由：key  ---> value 
>
> ​						key -> url
>
> ​						value--> 视图组件
>
> React 学习：React 全家桶
>
> 1. React基础  jsx 组件....
> 2. React-router-dom  【pc浏览器端使用的路由】 react-router-native 移动端路由
> 3. Redux  集中状态管理
>
> 多页面应用：服务器端渲染
>
> 单页面应用：浏览器端【客户端】渲染页面
>
> 优势：
>
> 1. 减少请求次数。减轻服务器端压力
> 2. 服务器端只响应数据，渲染交给客户端，减轻服务器压力
>
> 劣势：
>
> 1. 不利于SEO搜索引擎优化 Search Engine Optimization [搜索引擎优化]
>
> 使网站排名靠前
>
> 前端路由的作用：就是构建单页面应用

### 6-1. 路由基本使用

#### 1. 安装

```yarn add react-router-dom``` ```npm i react-router-dom``` 

1. BrowserRouter : 是一个容器组件，所有的路由组件想要通过路由能够进行切换，必须包裹在这个容器中
2. Routes：组件：作用：我们会定义多个路由，路由都要放在 Routes进行统一管理
3. Route： 定义 key -》 value规则 ， 路径url  ---》 路由组件

#### 2. 路由配置

```jsx
import React from 'react'
import {BrowserRouter, Routes, Route} from 'react-router-dom'
import Index from './pages/Index'
import Login from './pages/Login'
import Test from './pages/Test'
export default function App() {
  return (
    <div>
        {/* BrowserRouter 包裹所有的路由组件容器，经过包裹，才能完成路由切换 */}
        <BrowserRouter>
            {/* 所有的路由组件都要在 Routes组件中 */}
            <Routes>
                {/* 具体的路径和路由组件定义一一对应关系的组件 */}
                {/* path 路径  element 该路径要渲染的组件 */}
                {/* 访问：localhost:3000/index     localhost:3000/login   localhost:3000/test */}
                <Route path='/index' element={<Index />}></Route>
                <Route path='/login' element={<Login />}></Route>
                <Route path='/test' element={<Test />}></Route>
            </Routes>
        </BrowserRouter>
    </div>
  )
}
```

#### 3. 跳转链接配置

> 可以通过 Link 和 NavLink进行组件无刷新切换

- 引入组件

```jsx
import {BrowserRouter, Routes, Route, Link, NavLink} from 'react-router-dom'
```

- App.jsx

```jsx
import React from 'react'
import {BrowserRouter, Routes, Route, Link, NavLink} from 'react-router-dom'
import Index from './pages/Index'
import Login from './pages/Login'
import Test from './pages/Test'
import './App.css'
export default function App() {
  return (
    <div>
        {/* BrowserRouter 包裹所有的路由组件容器，经过包裹，才能完成路由切换 */}
        <BrowserRouter>
            <nav>
                {/* 可以跳转，但是页面会刷新 */}
                {/* <a href="/index">首页</a>
                <a href="/login">登录页</a>
                <a href="/test">测试页</a> */}

                {/* Link内部阻止了a标签跳转的默认行为 ，能够无刷新切换，但是无法区分哪一个是当前的，不能进行高亮*/}
                {/* <Link to='/index'>首页</Link>
                <Link to='/login'>登录页</Link>
                <Link to='/test'>测试页</Link> */}

                {/* NavLink 也是生成a标签，实现无刷新跳转切换路由。但是当前的回一个 class="active" 可以通过定义样式，实现高亮 */}
                <NavLink to='/index'>首页</NavLink>
                <NavLink to='/login'>登录页</NavLink>
                <NavLink to='/test'>测试页</NavLink>

            </nav>
            {/* 所有的路由组件都要在 Routes组件中 */}
            <Routes>
                {/* 具体的路径和路由组件定义一一对应关系的组件 */}
                {/* path 路径  element 该路径要渲染的组件 */}
                {/* 访问：localhost:3000/index     localhost:3000/login   localhost:3000/test */}
                <Route path='/index' element={<Index />}></Route>
                <Route path='/login' element={<Login />}></Route>
                <Route path='/test' element={<Test />}></Route>
            </Routes>
        </BrowserRouter>
    </div>
  )
}
```

- App.css

```css
a{
    padding:5px 10px;
}
a.active{
    color:red;
}
```

#### 4. 自定义高亮类名

> className可以是一个函数，函数的参数，可以接到一个对象 {isActive:boolean} 如果是当前路由，那么值为true，不是 就是 false

- App.jsx

```jsx
<NavLink className={({isActive})=>isActive ? 'a_active':''} to='/index'>首页</NavLink>
<NavLink className={({isActive})=>isActive ? 'a_active':''} to='/login'>登录页</NavLink>
<NavLink className={({isActive})=>isActive ? 'a_active':''} to='/test'>测试页</NavLink>
```

- App.css

```css
a.a_active{
    color:blueviolet
}
```

#### 5. 路由默认显示的问题，重定向

> Navigate重定向组件， 可以重定向到 to指定的路由

- App.jsx

```jsx
import {BrowserRouter, Routes, Route, Link, NavLink, Navigate} from 'react-router-dom'

<Route index element={<Navigate to='/index'/>}></Route>
```

#### 6. 404页面

- src->pages/NotFound.jsx

```jsx
import React from 'react'

export default function NotFound() {
    return (
        <div>NotFound</div>
    )
}
```

- src->App.jsx

```jsx
{/* 当访问一个不存在的路径的时候，显示一个404 Page  Not Found 页面 */}
<Route path='*' element={<NotFound/>}></Route>
```

#### 7. 一级路由练习

- public -> index.html 引入bootstrap

```html
<link rel="stylesheet" href="/css/bootstrap.css">
```

- src->pages->home.jsx

```jsx
import React from 'react'

export default function Home() {
    return (
        <div className="panel">
            <div className="panel-body">
                <h3>我是Home的内容</h3>
            </div>
        </div>
    )
}
```

- src->pages->about.jsx

```jsx
import React from 'react'
export default function About() {
    return (
        <div className="panel">
            <div className="panel-body">
                <h3>我是About的内容</h3>
            </div>
        </div>
    )
}
```

- src->App.jsx

```jsx
import React from 'react'
import {Routes, Route, Navigate, NavLink} from 'react-router-dom'
import About from './pages/About'
import Home from './pages/Home'
export default function App() {
    return (
        <div>
            <div className="row">
                <div className="col-xs-offset-2 col-xs-8">
                    <div className="page-header">
                        <h2>React Router Demo</h2>
                    </div>
                </div>
            </div>
            <div className="row">
                <div className="col-xs-2 col-xs-offset-2">
                    <div className="list-group">
                       
                        <NavLink className="list-group-item" to='/home'>Home</NavLink>
                        <NavLink className="list-group-item" to='/about'>About</NavLink>
                    </div>
                </div>
                <div className="col-xs-6">
                    <Routes>
                        <Route path='/home' element={<Home/>}></Route>
                        <Route path='/about' element={<About/>}></Route>
                        <Route path='/' element={<Navigate to='/home' />}></Route>
                    </Routes>
                </div>
            </div>
        </div>
    )
}
```

#### 8. 二级路由练习

> 1. Route嵌套定义子路由
>
> <Route>
>
> ​			<Route />
>
> ​			<Route />
>
> </Route>
>
> 2. <Outlet/> 展示子路由组件

- App.jsx

```jsx
<Routes>
    <Route path='/home' element={<Home/>}>
        {/* 
           Route标签可以嵌套，嵌套后，内部的就是外部路由的子路由 
          子路由path有两种写法：
          1. 完整路径： /一级路由/二级路由
          2. 简写路径： 二级路由 ： 注意前面没有 /
        */}
    	<Route path='/home/news' element={<News/>} ></Route>
    	<Route path='message' element={<Message/>}></Route>
    </Route>
    <Route path='/about' element={<About/>}></Route>
    <Route path='/' element={<Navigate to='/home' />}></Route>
</Routes>
```

- Home.jsx

> 通过Outlet组件，来展示二级路由组件

```jsx
import React from 'react'
import { Outlet, NavLink } from 'react-router-dom'

export default function Home() {
    return (
        <div class="panel">
            <div class="panel-body">
                <div>
                    <h2>Home组件内容</h2>
                    <div>
                        <ul class="nav nav-tabs">
                            <li>
                                {/* <a class="list-group-item active" href="./home-news.html">News</a> */}
                                <NavLink className="list-group-item" to='/home/news'>News</NavLink>
                            </li>
                            <li>
                                <NavLink className="list-group-item" to='message'>Message</NavLink>
                            </li>
                        </ul>
                        {/*
                         二级路由显示区域
                            通过Outlet 组件，展示二级路由组件
                         */}
                        <Outlet />
                    </div>
                </div>
            </div>
        </div>
    )
}

```

#### 9. 路由表

> 1. 将路由配置信息单独写入文件 routes/index.js，并导出配置路由信息的数组
> 2. 导入路由表  `import routes from './routes/index.js'`
>
> 3. 使用  `const routers = useRoutes(routes)` 将路由表生成路由配置
>
> 4. 页面展示 `{routers}`

1. src/routes/index.js  配置路由表

```js
import About from '../pages/About'
import Home from '../pages/Home'
import Message from '../pages/Message'
import News from '../pages/News'
import Detail from '../pages/Detail'
import { Navigate } from 'react-router-dom'
const routes = [
    {
        path:'/home',
        element:<Home/>,
        children:[
            {
                //完整路径
                path:'/home/news',
                element:<News/>,
                // 配置子路由
                children:[
                    {
                        path:'detail',
                        element:<Detail/>
                    }
                ]
            },
            {
                // 路径简写
                path:'message',
                element:<Message/>
            },
            // 配置重定向
            {
                index:true,
                element:<Navigate to='news'/>
            }
        ]
    },
    {
        path:'/about',
        element:<About/>
    },
    {
        index:true,
        element:<Navigate to='/home' />
    }
]

export default routes
```

2. App.jsx   通过 useRoutes 使用路由表

```jsx
import React from 'react'
import {Routes, Route, Navigate, NavLink, useRoutes} from 'react-router-dom'
import routes from './routes'
export default function App() {
    //使用useRoutes hook，将路由表转化为可以用插值语法渲染的路由配置 routers
    const routers = useRoutes(routes)
    return (
        <div>
            <div className="row">
                <div className="col-xs-offset-2 col-xs-8">
                    <div className="page-header">
                        <h2>React Router Demo</h2>
                    </div>
                </div>
            </div>
            <div className="row">
                <div className="col-xs-2 col-xs-offset-2">
                    <div className="list-group">
                       
                        <NavLink className="list-group-item" to='/home'>Home</NavLink>
                        <NavLink className="list-group-item" to='/about'>About</NavLink>
                    </div>
                </div>
                <div className="col-xs-6">
                    {/* 路由配置信息 */}
                    {/* 使用插值语法进行渲染 */}
                    {routers}
                </div>
            </div>
        </div>
    )
}

```

#### 10. HashRouter 模式

```jsx
// const element = <HashRouter><App/></HashRouter>
const element = <BrowserRouter><App/></BrowserRouter>
```

#### 11. useNavigate 实现编程式导航

> 1. useNavigate  与 Link  NavLinK 区别是什么？什么场景使用那种？
>
> 答：当进行简单的跳转的时候，使用 Link 和 NavLink 。简单跳转指的是，没有逻辑的跳转，点击必然跳转。
>
> useNavigate 进行复杂逻辑的跳转。比方说，有权限的跳转。用户没有登录，跳到一个页面，登录后跳到另一个页面

使用：

```js
// 1. 创建 navigate函数
const navigate = useNavigate()
// 2. 进行跳转
navigate('跳转路由链接')

```

##### 编程式导航传参

1. query参数

```js
<button onClick={()=>{
    navigate('/detail?id=1')
}}>query参数</button>
// 注意路由表中没有参数占位符
// 传递 query, state 参数 
path:'/detail',
    
//接收：
 // 获取query参数
let [searchQuery,setSearchQuery] = useSearchParams()
let id1 = searchQuery.get('id');
console.log(id1);   

```

2. params参数：

```js
<button onClick={()=>{
     navigate('/detail/1')
}}>params参数</button>
// 路由表中需要有占位信息
// params参数
path:'/detail/:id',

// 接收
// params 参数
// 结构赋值的重命名： {id:别名}
let {id:id2} = useParams()
console.log(id2);
```

3. state参数

```js
<button onClick={()=>{
    navigate('/detail',{
        state:{id:3},
        // 替换掉上一次的历史记录
        replace:true
    })
}}>state参数</button>

// 路由表没有占位信息
path:'/detail',
    
// 接收
// state参数接收
let {state:{id:id3}} = useLocation()
console.log(id3)

```

4. 回退

```js
let navigate = useNavigate()

navigate(-1)

navigate(1)

navigate(-2)
```



#### 12. 路由传参之params参数

> username = 'atguigu'
>
> /user/atguigu    ==>               params
>
> /user?username=atguigu      query

```jsx
//路由表中占位符
//routes/index.js

import Hero from '../pages/Hero'
import Detail from '../pages/Detail'
import { Navigate } from 'react-router-dom'

const routes = [
    {
        
        path:'/hero',
        element:<Hero/>
    },
    {
        // 传递params参数   需要配置占位符
        path:'/detail/:id',
        element:<Detail/>
    },
    {
        index:true,
        element:<Navigate to='/hero' />
    }
]
export default routes


// 2. 在路由组件中使用 useParams获取 params参数
// 获取params 参数   路由表中配置的  path:'/detail/:id', 占位符是 id
// let res = useParams() 
// console.log(res);// {id:参数值}
let {id} = useParams()
```

#### 13. query参数

1. 跳转链接  /路径?键名=键值&键名2=键值2&.....

```jsx
 {heros.map(hero => (
    <li key={hero.id}><Link to={'/detail?id=' + hero.id+'&name=atguigu'}>{hero.name}</Link></li>
            ))}
```

2. 获取参数

```js
let [searchQuery, setSearchQuery] = useSearchParams()
// 获取id
let id = searchQuery.get('id')
```

#### 14. state参数

1. 跳转链接 : 传递的数据不同，也不会体现在地址栏中

```jsx
// state参数,可以传递复杂数据，并且数据不会出现在地址栏中
<li key={hero.id}><Link to={'/detail'} state={hero}>{hero.name}</Link></li>
```

2. 接收

```jsx
// 接收state参数
let x =  useLocation() 
    /*
    {
        "pathname": "/detail",
        "search": "",
        "hash": "",
        "state": {
            "id": 1,
            "name": "孙尚香",
            "say": "大小姐嫁到，统统闪开!"
        },
        "key": "bgyf61ta"
    }
    */
let {state:{id}} = x
```





#### 15. 路由组件懒加载

> 实现路由组件的按需加载
>
> 用户访问到的时候，才回去加载，没访问到，不加载
>
> 使用： lazy  Suspense

- src/routes/index.js

```js
import { Navigate } from 'react-router-dom'
// 同步加载：不管用户是否跳转到该页面，都会加载
// 希望：用户点到哪个路由，就加载哪个路由组件
// import Hero from '../pages/Hero'
// import Detail from '../pages/Detail'

// lazy 懒惰的  Suspense 存疑的
import {lazy,Suspense} from 'react'
// 懒加载hero组件 和 Detail组件
const Hero = lazy(()=>import('../pages/Hero'))
const Detail = lazy(()=>import('../pages/Detail'))

// 封装懒加载组件 函数
function load(Com){
   return (
    <Suspense fallback={<div>组件正在加载中.....</div>}>
        {Com}
    </Suspense>
   ) 
}

const routes = [
    {
        
        path:'/hero',
        element:(
            // fallback Hero组件未加载完成时，显示的内容
            // <Suspense fallback={<div>组件正在加载中.....</div>}><Hero/></Suspense>
            load(<Hero />)
        )
    },
    {
        // 传递 query 参数 
        path:'/detail',
        element:(
            // <Suspense fallback={<div>组件正在加载中.....</div>}><Detail/></Suspense>
            load(<Detail/>)
        )
    },
    {
        index:true,
        element:<Navigate to='/hero' />
    }
]
export default routes
```

### 6-2 路由权限验证

> 很多页面其实是需要有权限才可以访问。
>
> 希望定义一个权限验证的组件，来实现对需要权限访问的路由的一个限定
>
> 判断有没有权限的标准：
>
> 就是看localStorage里面是否有 token 属性

- 路由表配置：src/routes/index.js

```js
import { Navigate } from 'react-router-dom'
import Car from '../pages/Car'
import Index from '../pages/Index'
import Login from '../pages/Login'
import Users from '../pages/Users'
// 权限验证组件，需要验证权限的，用它来包裹
import Auth from '../components/Auth'

const routes = [
    {
        path:'/login',
        element:<Login/>
    },
    {
        path:'/index',
        element:<Index/>
    },
    {
        path:'/users',
        element:<Auth><Users/></Auth>
    },
    {
        path:'/car',
        element:<Auth><Car/></Auth>
    },
    {
        path:'/',
        element:<Navigate to='/index' />
    }
]

export default routes;
```

- 权限验证组件 src/components/Auth.jsx

```jsx
import React from 'react'
import { Navigate } from 'react-router-dom'

export default function Auth(props) {
    // 可以获取Auth组件的子组件 react元素
    console.log('children', props.children)

    if (!localStorage.getItem('token')) {
        console.log('car')
        // 没有权限 重定向
        return <Navigate to='/login'/>
    }
    // 有权限返回子组件
    return props.children
}
```

## 七. Redux

> React 的集中状态管理工具
>
> store【项目经理】：数据仓库
>
> reducer【苦逼程序员】：操作数据仓库中的数据，都要通过 reducer操作
>
> dispatch|action【产品经理】：发布操作指令
>
> 项目经理：管理整个项目【数据】，他不具体干活
>
> 程序员：具体干活的
>
> 产品经理：提需求的
>
> 产品经理可能管很多项目
>
> 每一个项目都有数据。每一个项目都有自己的干活的程序员，每一个项目都有自己提需求的产品经理

### 0. 基本使用：

1. 安装：

```shell
yarn add @reduxjs/toolkit
npm i @reduxjs/toolkit
```

2. 创建数据切片【模块】

```js
//1. 导入包 
// createSlice : 创建模块的【切片】
// configureStore： 创建仓库的 store
import {createSlice,configureStore} from '@reduxjs/toolkit'
// 创建模块
const goodsSlice = createSlice({
    // 模块名,必须要有
    name:'goods',
    // 模块数据  initial:初始化的  State 状态~ ! 值一般是一个对象
    initialState:{
        goodsName:'华为mate50Pro',
        price:100,
        num:10
    },
    // 干活的程序员
    reducers:{
        // 程序员能干什么活儿，就是一个一个的函数
        // state 是当前模块的最新数据
        // action是操作类型 {TYPE:'需求',payload:数据}
        // incrementNum(state,action){
        // }
        // 因为Type一般用不到，只用payload，所以 reducers 会直接解构出 payload进行使用
        // 在reducer中定义的函数，会出现在数据切片的actions属性上。比如： goodsSlice.actions的对象中
        incrementNum(state,{payload}){
            // 1. 可以直接给state赋值 
            // 2. 不用return
            state.num += 1
        },
        decrementNUm(state,{payload}){
            state.num -= 1
        }
    }
})
console.log(goodsSlice);// 观察goodsSlice.ations对象的值
```

3. 创建数据仓库

```js
// 使用 configureStore创建仓库【项目经理】
const store = configureStore({
    reducer:{
    // 给项目经理指派手底下干活的程序员
       goods: goodsSlice.reducer
    }
})
console.log(goodsSlice);// 观察goodsSlice.ations对象的值
// actions对象中存储的就是 指派的动作

// 获取状态数据唯一的途径，就是通过 store.getState()
console.log(store.getState().goods); // goods获取模块切片数据。名字是跟 configureStore 里面reducer定义的保持一致

// 获取动作
const {incrementNum, decrementNum} = goodsSlice.actions
console.log('incrementNum',incrementNum)// increament,decrement 都是 actionCreater，actionCreater是用来创建动作的函数

console.log(incrementNum(10)) //操作动作：{type: 'goods/incrementNum', payload: 10} increment函数的参数，就是payload的值

// store.dispatch(动作) :操作数据唯一正确的途径 dispatch
store.dispatch(incrementNum(10))

console.log(store.getState().goods)

```

4. 设置监听和取消监听

```js

// 监听仓库数据的变化，如果state发生变化，那么执行回调函数. subscribe 函数执行后，会有一个返回值
// 返回值也是一个函数，函数执行可以取消监听
const unsubscribe = store.subscribe(()=>{
    // 如果state发生变化，那么执行回调函数
    console.log(store.getState())
})

// store.dispatch(动作) :操作数据唯一正确的途径 dispatch
store.dispatch(incrementNum(10))
unsubscribe() // 取消监听后，后续变化，监听不到了
store.dispatch(incrementNum(20))
// console.log(store.getState().goods)
```

### 1. 添加商品练习

```js
/**
 *  需求： 维护管理商品数据
 *      goodsList:[
 *          {
 *              id:1,
 *              gname:商品名称,
 *              price:价格
 *          }
 *      ]
 */

// 1. 引入包
import {
    createSlice,
    configureStore
} from '@reduxjs/toolkit'

// 2. 创建模块 【切片】
const goodsSlice = createSlice({
    name:'goods',
    // 初始化状态
    initialState:{
        goodsList:[]
    },
    reducers:{
        addGoods(state, {payload}){
            // 添加goods
            state.goodsList.unshift({
                // id:Date.now()
                // 0-9 a-z 26字母  10 =》 a  11 b  36 z ==> 0.xxx
                id:Math.random().toString(36).slice(2), // 字母和26个英文字母随机的字符串
                ...payload
            })
        }
    }
})
// 3. 创建仓库【项目经理】
const store = configureStore({
    // 给项目经理指派程序员
    reducer:{
        goods:goodsSlice.reducer
    }
})
// 4. 获取动作： action 
const {addGoods} = goodsSlice.actions  // addGoods 是 actionCreator，addGoods(数据) ==> action ==》 {type:'模块名/方法名',payload:数据}
// 5. 设置监听
const unsubscribe = store.subscribe(()=>{
    // 获取状态唯一正确的方式
    console.log(store.getState())
})
// 6. 指派任务 dispatch(action)
store.dispatch(addGoods({
    gname:'华为mate50pro',
    price:7999
}))
store.dispatch(addGoods({
    gname:'iphone 14 max pro',
    price:9999
}))
```

### 2. 添加购物车

```js
/**
 *  需求： 维护管理商品数据
 *      goodsList:[
 *          {
 *              id:1,
 *              gname:商品名称,
 *              price:价格
 *          }
 *      ]
 */

// 1. 引入包
import {
    createSlice,
    configureStore
} from '@reduxjs/toolkit'
// 2. 创建模块 【切片】
const goodsSlice = createSlice({
    name:'goods',
    // 初始化状态
    initialState:{
        goodsList:[]
    },
    reducers:{
        addGoods(state, {payload}){
            // 添加goods
            state.goodsList.unshift({
                // id:Date.now()
                // 0-9 a-z 26字母  10 =》 a  11 b  36 z ==> 0.xxx
                id:Math.random().toString(36).slice(2), // 字母和26个英文字母随机的字符串
                ...payload
            })
        }
    }
})
// 购物车模块
const carSlice = createSlice({
    name:'car',
    initialState:{
        carList:[],
        totalPrice:0
    },
    reducers:{
        addCar(state, {payload}){
            // todo 
            // console.log('payload',payload)
            state.carList.unshift(payload)
            // 计算总价
            state.totalPrice = state.carList.reduce((pre,cur)=>pre+cur.price * cur.buyNum ,0)
        }
    }
})

// 3. 创建仓库【项目经理】
const store = configureStore({
    // 给项目经理指派程序员
    reducer:{
        goods:goodsSlice.reducer,
        car:carSlice.reducer
    }
})
// 4. 获取动作： action 
const {addGoods} = goodsSlice.actions  // addGoods 是 actionCreator，addGoods(数据) ==> action ==》 {type:'模块名/方法名',payload:数据}
const {addCar} = carSlice.actions

// 5. 设置监听
const unsubscribe = store.subscribe(()=>{
    // 获取状态唯一正确的方式
    console.log(store.getState())
})
// 6. 指派任务 dispatch(action)
store.dispatch(addGoods({
    gname:'华为mate50pro',
    price:7999
}))
store.dispatch(addGoods({
    gname:'iphone 14 max pro',
    price:9999
}))

// 添加购物车
// 
store.dispatch(addCar({
    ...store.getState().goods.goodsList[1],
    buyNum:1
}))

store.dispatch(addCar({
    ...store.getState().goods.goodsList[0],
    buyNum:1
}))

```

### 3. 实际开发时store模块拆分和目录结构

```js
src
 |-store
 |   |-index.js               仓库入口文件
 |   |-slice[module|model]    模块目录
 |   |   |-goodsSlice.js      商品数据模块|切片
 |   |   |-carSlice.js        购物车数据模块|切片
```

### 4. 模块拆分

- store->index.js

```js
/**
 *  需求： 维护管理商品数据
 *      goodsList:[
 *          {
 *              id:1,
 *              gname:商品名称,
 *              price:价格
 *          }
 *      ]
 */
// 1. 引入包
import {
    configureStore
} from '@reduxjs/toolkit'
import car from './slice/car'
import goods from './slice/goods'

// 3. 创建仓库【项目经理】
const store = configureStore({
    // 给项目经理指派程序员
    reducer:{
        goods,
        car
    }
})
export default store;
```

- store->slice->car.js

```js
import {createSlice} from '@reduxjs/toolkit'
// 购物车模块
const carSlice = createSlice({
    name:'car',
    initialState:{
        carList:[],
        totalPrice:0
    },
    reducers:{
        addCar(state, {payload}){
            // todo 
            // console.log('payload',payload)
            state.carList.unshift(payload)
            // 计算总价
            state.totalPrice = state.carList.reduce((pre,cur)=>pre+cur.price * cur.buyNum ,0)
        }
    }
})
/**
 * 对于一个切片来说，有哪些是需要暴露的呢？
 * 1.  reducer
 * 2.  actions
 *
 */
// 使用分别暴露，暴露 actions
export const {addCar} = carSlice.actions
// 使用默认暴露，暴露 reducer
export default carSlice.reducer
/**
 *  同一个文件，可以使用多种暴露方式
 * 
 */
```

- store->slice->goods.js

```js
import {createSlice} from '@reduxjs/toolkit'
// 2. 创建模块 【切片】
const goodsSlice = createSlice({
    name:'goods',
    // 初始化状态
    initialState:{
        goodsList:[]
    },
    reducers:{
        addGoods(state, {payload}){
            // 添加goods
            state.goodsList.unshift({
                // id:Date.now()
                // 0-9 a-z 26字母  10 =》 a  11 b  36 z ==> 0.xxx
                id:Math.random().toString(36).slice(2), // 字母和26个英文字母随机的字符串
                ...payload
            })
        }
    }
})
export const {addGoods} = goodsSlice.actions
export default goodsSlice.reducer
```

- src->index.js   使用仓库

```js
import store from "./store"
import { addCar } from "./store/slice/car"
import { addGoods } from "./store/slice/goods"
// 5. 设置监听
const unsubscribe = store.subscribe(()=>{
    // 获取状态唯一正确的方式
    console.log(store.getState())
})
// 6. 指派任务 dispatch(action)
store.dispatch(addGoods({
    gname:'华为mate50pro',
    price:7999
}))
store.dispatch(addGoods({
    gname:'iphone 14 max pro',
    price:9999
}))
// 添加购物车
store.dispatch(addCar({
    ...store.getState().goods.goodsList[1],
    buyNum:1
}))
store.dispatch(addCar({
    ...store.getState().goods.goodsList[0],
    buyNum:1
}))


```

### 5. redux 在react项目中的计数器

- src-->index.js
  - 订阅了监视器，使得仓库数据更新，重新渲染页面

```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import store from './store'
const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(<App/>)
// 创建监视器
store.subscribe(()=>{
    root.render(<App/>)
})
```

- App.js

```jsx
import React from 'react'
import store from './store'
import { increment,decrement } from './store/slice/count'
export default function App() {
    /**
     * 当前redux第一个问题：
     * 1. 数据改变页面刷新不了
     * 2. 数据其实都是服务器请求得来的。请求数据需要时间，就涉及到异步操作。
     *      真实情况，是我们需要发起请求，异步获取数据后在更新仓库数据
     * 3. redux怎么支持异步操作？
     * 
     * 4. 每次操作，都需要引入store，都需要引入 actions，比较麻烦。
     *      
     */
    return (
        <div style={{padding:50}}>
            <h3>计算器</h3>
            <p><button onClick={()=>{
                // 仓库里的数据已经改变，但是页面没有刷新
                store.dispatch(decrement(1))
            }}>-</button> {store.getState().count.num} <button onClick={()=>{
                store.dispatch(increment(3))
            }}>+</button></p>
        </div>
    )
}
```

- store->index.js

```js
import {
    configureStore
} from '@reduxjs/toolkit'
import count from './slice/count' // reducer

const store = configureStore({
    reducer:{
        count
    }
})
export default store;
```

- store->slice->count.js

```js
import {createSlice} from '@reduxjs/toolkit'
const countSlice = createSlice({
    name:'count',
    initialState:{
        num:100
    },
    reducers:{
        increment(state,{payload}){
            state.num += payload
        },
        decrement(state,{payload}){
            state.num -= payload
        }
    }
})
export const {increment, decrement} = countSlice.actions
export default countSlice.reducer
```

### 6. 异步修改数据

> store.dispatch( 回调函数 )
>
> 回调函数有两个参数 : 第一个参数就是 派发任务的 dispatch  第二个参数是获取状态的 getState【一般不用】
>
> 异步的修改，会定义在切片中。在切片中同时要定义同步的reducer，并且需要将reducer中的函数，通过 actions进行解构

- src->store->slice->count.js

```js
import { createSlice } from '@reduxjs/toolkit'

const countSlice = createSlice({
    name: 'count',
    initialState: {
        num: 100
    },
    reducers: {
        increment(state, { payload }) {
            state.num += payload
        },
        decrement(state, { payload }) {
            state.num -= payload
        }
    }
})
// 同步的action即便外部不适用，也要定义解构，目的是为了给下面异步的方法使用
export const { increment, decrement} = countSlice.actions

export default countSlice.reducer

export function asyncIncrement(payload) {
    return function fn(dispatch, getState) {
        setTimeout(() => {
            dispatch(increment(payload))
        }, 1000)
    }
}
// 纯函数：当参数相同的时候，函数执行的结果是确定，这种函数就是纯函数

// function sum(a , b){
//     return a + b
// }

// sum(1,2)
// let count = 10;
// function sum(a){
//     return a + count;
// }

// sum(1)
```

- App.jsx

```jsx
import React from 'react'
import store from './store'
import { increment, decrement, asyncIncrement, asyncDecrement } from './store/slice/count'
export default function App() {
    /**
     * 当前redux第一个问题：
     * 1. 数据改变页面刷新不了
     * 2. 数据其实都是服务器请求得来的。请求数据需要时间，就涉及到异步操作。
     *      真实情况，是我们需要发起请求，异步获取数据后在更新仓库数据
     * 3. redux怎么支持异步操作？
     * 
     * 4. 每次操作，都需要引入store，都需要引入 actions，比较麻烦。
     *      
     */

    return (
        <div style={{ padding: 50 }}>
            <h3>计算器</h3>
            <p><button onClick={() => {
                // 仓库里的数据已经改变，但是页面没有刷新
                store.dispatch(decrement(1))
            }}>-</button> {store.getState().count.num} <button onClick={() => {
                store.dispatch(increment(3))
            }}>+</button>

                <button onClick={() => {
                    //  setTimeout(()=>{
                    //     store.dispatch(increment(6)) // increment(6) ==> {type:'切片/函数',payload:6}
                    //  },2000)
                    // function fn(dispatch, getState) {
                    //     setTimeout(() => {
                    //         dispatch(increment(6))
                    //     }, 1000)
                    // }

                    // function asyncIncrement(payload) {
                    //     return function fn(dispatch, getState) {
                    //         setTimeout(() => {
                    //             dispatch(increment(payload))
                    //         }, 1000)
                    //     }
                    // }
                    store.dispatch(asyncIncrement(8))
                }}>异步修改 + 6</button>
                <button onClick={() => {

                }}>异步修改 - 5</button>
            </p>
        </div>
    )
}
/**
 * 异步的方法定义在切片里。
 * 定一个同步的 reducer，在异步的方法中调用
 * 一般而言，请求都是异步的，所以暴露给外部使用的，一般就是异步的方法
 * 同步的方法，需要从actions中解构。因为异步的函数后续要使用
 * 
 * 
 */
```

### 7. react-redux

> 作用：方便的在react项目中，使用reduxjs/toolkit
>
> 1. Provider : 是一个组件。有一个属性 store，将redux管理的数据，作为属性值。后代组件都可以接收到 store 【useSelector】
>
> 2. useSelector: 接收状态数据。它接收一个参数，参数是一个函数【回调函数】，回调函数的参数，就是state状态数据
>
>     内部回调函数的返回值，就是useSelector函数执行的结果
>
> 3. useDispatch: 用来生成 dispatch 函数，来批发任务，触发对应切片的reducer

1. 安装： `yarn add react-redux`
2. 使用：
   1. 在根组件外部包裹 Provider 并通过属性传递 store

- src ->index.js

```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import { Provider } from 'react-redux' // 可以将store数据，共享给所有的后代使用
import App from './App'
import store from './store'
const root = ReactDOM.createRoot(document.getElementById('root'))
root.render((
    // Provider 定一个一个固定属性 store，将redux管理的数据作为属性值
    <Provider store={store}>
        <BrowserRouter><App /></BrowserRouter>
    </Provider>
))
```

- 组件中使用  src--> pages--> car.jsx

```js
import {useSelector, useDispatch} from 'react-redux'
// 获取状态数据
//let data = useSelector(state=>state)  // 获取redux 管理的所有数据

let {carList, totalPrice} = useSelector(state=>state.car)
let dispatch = useDispatch()

dispatch(changeBuyNum({
    id:goods.id,
    buyNum:-1
}))
```



## 八. Redux 最佳实践

1. 封装了request

   > 配置 axios，配置了baseURL  配置响应拦截器，方便直接拿到 响应体

```js
src->request->index.js

/**
 * 
 *  配置axios
 */
import axios from 'axios'
// 配置公共路径几种方式？
// 方式一：
// axios.defaults.baseURL = 'http://localhost:7000'
// 方式二：
const request = axios.create({
    baseURL:'http://localhost:7000'
})
// 配置拦截器-- 响应拦截器
request.interceptors.response.use(response=>response.data)
export default request
```

2. Api

   > 将所有的请求后端的接口进行统一管理

```js
//src->api->todo.js

import request from '../request'

// 添加todo
export function addTodo(payload){ //payload => {title,isDone}
    // 是一个异步请求，promise对象
    return request.post('/todos',payload)
}
```

#### 异步操作数据库和仓库数据

> 在对应的数据切片中，
>
> 1. 定义一个同步的修改仓库的方法【reducers】
> 2. 在定义一个异步修改数据库的方法
>    1. 在异步方法中，发送ajax请求【api】请求后端接口。
>    2. 然后调用同步的方法，修改仓库数据
> 3. 一般我们会将切片中定义的异步方法，暴露出去在外部调用。调用的时候，即改变远端数据库，又改变本地仓库数据

- src->store->slice->todo.js

```js
import {createSlice} from '@reduxjs/toolkit'
import * as todoApi from '../../api/todo'
const todoSlice = createSlice({
    name:'todolist',
    initialState:{
        todos:[]
    },
    reducers:{
        addTodo(state,{payload}){ // payload ==> {_id,title,isDone}
            state.todos.push(payload)
        }
    }
})

export const {addTodo} = todoSlice.actions
export default todoSlice.reducer

export function asyncAddTodo(payload){
    return async dispatch=>{
        // 执行异步的代码
        let {todo} = await todoApi.addTodo(payload)
        // console.log('todo',todo);
        // 执行添加仓库的操作
        dispatch(addTodo(todo))
    }
}
```

- src->components->Header.jsx

```js
import React, { useRef } from 'react'
import { useDispatch } from 'react-redux'
import {asyncAddTodo} from '../store/slice/todo'
export default function Header() {
    const dispatch = useDispatch()
    function keyUpHandler(e){
        if(e.key !== 'Enter') return 
        let keyword = e.target.value.trim()
        // 需要将keyword 组合成一个 todo 发送给后台接口，添加数据
        dispatch(asyncAddTodo({
            title:keyword,
            isDone:false
        }))
    }
    return (
        <div className="todo-header">
            <input  type="text" placeholder="请输入你的任务名称，按回车键确认" onKeyUp={keyUpHandler}/>
        </div>
    )
}

```

#### 根据id 删除 todo

> 执行删除操作：需要即修改远程数据库中的数据，又要修改本地redux中的数据
>
> 在数据切片中：定义异步函数，在函数中完成上述两步操作

- src->store->slice->todo.js
  1. 定义asyncXxxx异步函数
  2. 为了发送ajax请求，需要定义api
  3. 为了修改本地redux，需要定义同步的reducer
  4. 为了dispatch派发任务，需要结构 actions

```js
import {createSlice} from '@reduxjs/toolkit'
import * as todoApi from '../../api/todo'
const todoSlice = createSlice({
    name:'todolist',
    initialState:{
        todos:[]
    },
    reducers:{
        addTodo(state,{payload}){ // payload ==> {_id,title,isDone}
            state.todos.push(payload)
        },
        getTodos(state, {payload}){
            state.todos = payload
        },
        // 根据id删除todo
        deleteTodo(state,{payload}){
            state.todos = state.todos.filter(todo=>todo._id !== payload)
        }

    }
})

export const {addTodo,getTodos,deleteTodo} = todoSlice.actions
export default todoSlice.reducer
// 添加todo
export function asyncAddTodo(payload){
    return async dispatch=>{
        // 执行异步的代码，修改远程数据库
        let {todo} = await todoApi.addTodo(payload)
        // console.log('todo',todo);
        // 执行添加仓库的操作
        dispatch(addTodo(todo))
    }
}

// 获取todoList ==> todos
export function asyncGetTodos(){
    return async dispatch=>{
        // 发送请求，获取所有的todos ==》 api
        let {todos} = await todoApi.getTodos()
        // 修改本地仓库
        dispatch(getTodos(todos))
    }
}
// 根据id删除todo
export function asyncDeleteTodo(payload){ // payload = _id
    return async dispatch=>{
        // 发请求api请求删除数据
        await todoApi.deleteTodo(payload)
        // 修改本地仓库库
        dispatch(deleteTodo(payload))
    }
}
```

- src->api->todo.js

```js
// 根据id删除todo
export function deleteTodo(id){
    return request.delete('/todos/' + id)
}
```

- 在组件中调用切片中定义好的 异步函数，执行操作

```jsx
// src->components->Item.jsx

<button className="btn btn-danger" onClick={()=>{
        dispatch(asyncDeleteTodo(todo._id))
    }}>删除</button>
```

