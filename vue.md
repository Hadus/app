# vue
[TOC]

## 1.介绍
 > https://cn.vuejs.org/
 
### 优点：
+ 做单页面应用程序
+ 比angular小巧，有很多第三方库配合开发
+ MVVM 思想让开发只关注于业务逻辑（VM）层的开发
+ 到底用哪个框架，需要大家讨论，公司决定
+ 把angular中的指令，拆分成为指令和组件两部分

### 框架与类库的区别
+ 框架：提供一整套解决方案，对项目侵入性大，更换时需要重构整个项目
+ 库：多用于解决某些特定问题，对项目侵入性小，更换方便
+ jquery一个很重要的优点是：提供操作dom的很多api，而vue不提倡操作dom，而要让开发者更注重业务逻辑层

### MVVM：
+ M：model 页面数据层，数据的增删改查
+ V：view 视图层
+ VM：viewModel model和view之间的调度层

### MVC：
+ M：model 页面数据层，数据的增删改查
+ V：view 视图层
+ C：controller 逻辑层（包括路由模块）

## VUE实现双向数据绑定的原理
- 通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调
- 步骤
    + 首先我们为每个vue属性用Object.defineProperty()实现数据劫持，为每个属性分配一个订阅者集合的管理数组dep；
    + 然后在编译的时候在该属性的数组dep中添加订阅者，v-model会添加一个订阅者，{{}}也会，v-bind也会，只要用到该属性的指令理论上都会，
    + 接着为input会添加监听事件，修改值就会为该属性赋值，触发该属性的set方法，在set方法内通知订阅者数组dep，订阅者数组循环调用各订阅者的update方法更新视图。
- 实现代码

```javascript
function defineReactive (obj, key, val) {
    var dep = new Dep();
        Object.defineProperty(obj, key, {
             get: function() {
                    //添加订阅者watcher到主题对象Dep
                    if(Dep.target) {
                        // JS的浏览器单线程特性，保证这个全局变量在同一时间内，只会有同一个监听器使用
                        dep.addSub(Dep.target);
                    }
                    return val;
             },
             set: function (newVal) {
                    if(newVal === val) return;
                    val = newVal;
                    console.log(val);
                    // 作为发布者发出通知
                    dep.notify();//通知后dep会循环调用各自的update方法更新视图
             }
       })
}
function observe(obj, vm) {
    Object.keys(obj).forEach(function(key) {
        defineReactive(vm, key, obj[key]);
    })
}

```

## 2.基本概念
- 使用过程
    + 引入vue.js会生成一个全局的Vue构造函数
    + 实例化Vue这个构造函数-并创建VM层（实例化过程）和M层（data）
    + data、computed、methods、watch的区别
        - data：定义属性
        - computed：属性调用，定义计算属性，value值为方法（返回一个属性），结果会被缓存
        - methods：定义方法，主要处理业务逻辑
        - watch：value值为对象，监控某些值的变化，
    ```javascript
    var vm = new Vue({
        el: '#app', // 需要vue控制的元素
        data: { // 数据
            msg: 'hello-world'
        },
        methods:{// 方法：书写业务逻辑
            click1(){
                console.log(1);
            },
            click2(a){
                console.log(a);
            }
        },
        filters: {// 自定义私有过滤器

        },
        directives: {// 自定义私有指令

        },
        components: {// 自定义私有组件

        },
        render(createdEle) { // 将组件compt放在#app处(webpack中使用)
            return createdEle(compt); // 返回一个组件对象，并替换el指定的容器
        }
        http: {// 配置http
            root: '/root',
            headers:{}
        },
        router: routerObj,// 将路由与模块绑定，用来监听hash变化触发路由
        watch: { // 监控vue实例的数据变化
            msg(newVal,oldVal){},
            '$route.path'(newVal,oldVal){} // 可用来监控路由变化
        },
        computed: { // 计算属性
            // 1.配置计算属性，属性和data平级，使用方式相同
            // 2.当n1,n2发生改变时候触发函数刷新n3的值
            // 3.n3不管被引用几次只会触发一次函数，结果会被缓存下来
            n3() {
                console.log('触发n3计算函数');
                return parseInt(this.n1 || 0) + parseInt(this.n2 || 0);
            }
        },

        // 实例的生命周期函数：1.创建期间，2.运行期间，3.销毁期间

        /*创建阶段*/
        // 1. 创建事件和生命周期
        beforeCreate() { // 生命周期函数-1-实例创建之前执行
            // 此时data/methods中的数据还未初始化
        },
        // 2. 开始创建实例
        created() { // 生命周期函数-2-实例创建完成执行
            // 此时data/methods中的数据已经被初始化
        },
        // 3. 开始编译模板（就是el绑定的元素），包括directives，并在内存中渲染为一个虚拟DOM
        beforeMount() { // 生命周期函数-3-虚拟DOM挂载到页面之前
            // 页面的数据还是旧的，没有用虚拟DOM替换页面的dom
        },
        // 4.将虚拟Dom挂载到页面上替换原始的dom 
        mounted() { // 生命周期函数-4-虚拟DOM挂载到页面之后（实例期间的最后一个周期函数，实例被完全创建完成）
            // 操作页面上的DOM节点，最早只能在这里操作
        },

        /*运行阶段*/
        beforeUpdate() { // 生命周期函数-5-data发生改变时触发，data数据不改变则不触发
            // 数据被更新，但页面更新之前
        },
        // 1. 更新虚拟DOM，并重新挂载到页面中
        updated() { // 生命周期函数-6-data发生改变时触发
            // 页面更新完成之后
        },

        /*销毁阶段--页面被卸载*/
        beforeDestroy() { // 生命周期函数-6-实例销毁之前
            // 实例上面的所有内容均可用
        },
        destroyed() { // 生命周期函数-7-实例销毁后

        },
    });
    ```

    + 绑定数据
    ```html
    <div id="app">
        <p>{{msg}}</p>
        <p v-text="msg">{{msg}}</p>
    </div>
    ```
    + 注意
        - 不要将body实例化为vue，2.x后的版本不建议这么做

    ## 3.内置指令
    ### 1.绑定数据
    - {{}}：插值表达式
    - v-text="" ：不会有闪烁，不需要v-clock
    - v-html="" ：绑定数据中含有html
    
    ### 2.闪烁：v-clock 
    - 添加了该属性的元素，在vue解析时候会先display:none，解析完成并绑定数据后在显示

    ### 3.绑定属性：v-bind
    - v-bind:title（简写为:title）
    - 绑定类名 :class
    ```html
        <div class="box" :class="['all',flag?'bg-red':'bg-green']">盒子</div>
        <div class="box" :class="['all',{'bg-green':true}]">盒子</div>
        <div class="box" :class="{all:true,'font-size':true}">盒子</div>
        <div class="box" :class="obj">盒子</div>
    ```
    ```javascript
    new Vue({
        el: '#app',
        data: {
            flag: true,
            obj: {
                'all': true,
                'bg-green': true,
                'font-big': true
            }
        }
    });
    ```
    - 绑定行内样式 :style
    ```html
    <div class="box all" :style="{'background-color':'red',color:'#fff'}">盒子</div>
    <div class="box all" :style="obj1">盒子</div>
    <div class="box all" :style="[obj1,obj2]">盒子</div>
    ```
    ```javascript
    new Vue({
        el: '#app',
        data: {
            obj1: {
                'background-color': 'green',
                color: 'white'
            },
            obj2: {
                'background-color': '#ccc',
                'font-size': '40px'
            }
        }
    });
    ```

    ### 4.绑定事件：v-on
    - v-on:click(简写为：@)
        + 不传参数的话，绑定函数名称自动触发
        ```html
        <div id="app">
            <button v-on:click="click1()">不传参数</button>
            <button v-on:click="click2('传参数')">传参数</button>
            <button @click="click3">data中的数据</button>
        </div>
        ```
        ```javascript
        new Vue({
            el: '#app',
            data: {
                msg: 'data中的数据'
            },
            methods: {
                click1() {
                    console.log('不传参数');
                },
                click2(a) {
                    console.log(a);
                },
                click3() {
                    console.log(this.msg);
                }
            }
        });
        ```

    - 监控多个事件
    `<button v-on="{mouseenter: onEnter,mouseleave: onLeave}">鼠标进来1</button>`
    - 绑定多个事件
    `<button @click="a(),b()">点我ab</button>`

    - 事件修饰符：直接在事件后面使用
        + .stop：阻止自己向上冒泡/向下捕获
        + .prevent：阻止默认行为
        + .capture：元素内部子元素以捕获方式触发事件
        + .self：只触发自己绑定的事件，不触发别的元素作用在自己身上的冒泡/捕获，依然会触发自己作用于别的元素上的冒泡/捕获
        + .once：只触发一次事件，别人作用于自己的冒泡/捕获和自身事件合并只触发一次，依然会触发自己作用于别人身上的冒泡/捕获
        + .prevent.once：让其他事件处理符和自身事件都只触发一次，与顺序无关
    
    - 内置按键修饰符
        + .113（直接加键盘对应的码值即可，为了好记，下面自定义按键修饰符给码值绑定别名）
        + .enter
        + .tab
        + .delete（捕获删除和退格）
        + .esc
        + .space
        + .up
        + .down
        + .right
        + .left

    - 给按键绑定别名（就是将键盘码值绑定别名）
    ```html
    <label for="">enter<input type="text" @keydown.enter="keydown('enter')"></label>
    <label for="">113---f2<input type="text" @keydown.113="keydown('未绑定f2')"></label>
    <label for="">绑定f2<input type="text" @keydown.f2="keydown('绑定f2')"></label>
    ```
    ```javascript
    Vue.config.keyCodes.f2 = 113;
    Vue.configkeyCodes = {
        v: 86,
        'media-play-pause': 179, // 不能用驼峰
        up: [38, 87]
    };
    ```

    ### 5.数据双向绑定：v-model
    - v-model="value"

    ### 6.循环：v-for（必须使用key，key只接受string/number类型数据）
    - 循环数组
    `<p v-for="(ele,index) in list" :key="index">ele--{{ele}}---index--{{index}}</p>`
    - 循环对象
    `<p v-for="(ele,key,index) in obj" :key="index">ele--{{ele}}---key--{{key}}---index--{{index}}</p>`
    -  循环数字：index从1开始
    `<p v-for="(ele,index) in 5">ele--{{ele}}---index--{{index}}</p>`

    ### 7.显示隐藏 v-ifv/v-else和/v-show
    - v-if/v-else :较高的代码执行消耗
    - v-show :较高的初始渲染消耗

    ## 4.自定义过滤器
    ### 1.自定义过滤器——全局
    ```javascript
    Vue.filter('timeFomat', timeStr => { // 第一个参数默认是进入管道的字符串
        var dateObj = new Date(timeStr);
        var y = dateObj.getFullYear();
        var m = dateObj.getMonth() + 1;
        var d = dateObj.getDate();
        return `${y}-${m}-${d}`;
    });
    ```
    ```html
    <h3>全局：{{time|timeFomat}}</h3>
    <h3>私有：{{time|timeFomat_mine('aaa')}}</h3>
    ```
    ### 2.自定义过滤器-私有
    ```javascript
    var app = new Vue({
        el: '#app',
        data: {
            time: new Date
        },
        filters: {
            timeFomat_mine(timeStr, addStr) { // 第一个参数默认是进入管道的字符串
                var dateObj = new Date(timeStr);
                var y = dateObj.getFullYear();
                var m = dateObj.getMonth() + 1;
                var d = dateObj.getDate();
                return `${y}-${m}-${d}------${addStr}`;
            }
        }
    })

    // 或者
    app.filters =  {
        timeFomat_mine(timeStr, addStr) { // 第一个参数默认是进入管道的字符串
            var dateObj = new Date(timeStr);
            var y = dateObj.getFullYear();
            var m = dateObj.getMonth() + 1;
            var d = dateObj.getDate();
            return `${y}-${m}-${d}------${addStr}`;
        }
    };
    ```
## 5.自定义指令
### 1.自定义指令-全局
```html
<p v-font-weight="900">color</p>
<p v-background="'pink'">color</p>
```
```javascript
Vue.directive('font-weight', {
    // 钩子函数
    bind(el, binding, vnode, oldVnode) { // 触发一次，指令绑定到元素上时，元素还未放在DOM树上渲染
        // 样式相关的操作，一般都可以在bind中执行

        // el---元素---DOM对象
        // binding: {
        //      name: 指令名称,
        //      value: 指令绑定值（计算之后）,
        //      expression: 指令绑定值的字符串形式（不计算）,
        //      oldValue: 指令绑定的前一个值，仅在update和componentUpdated中使用，无论值是否改变都可用,
        //      arg: 传给指令的参数,
        //      modifiers:包含修饰符的对象 
        // }
        // vnode: // VUE编译时产生的虚拟节点
        // oldVnode: // 上一个虚拟节点，仅在update和componentUpdated中使用

        el.style.fontWeight = binding.value;
    },
    inserted(el, ) { // 触发一次，元素插入到父节点中（父节点存在即可，不必存在在document中）
        // JS相关的操作，一般放在inserted中执行

        el.focus();
    },
    updated() { // 触发多次，vnode更新时

    },
    componentUpdated() { // 所有组件的vnode及其孩子的vnode更新时触发

    },
    ubind() { // 触发一次，指令解绑元素上时

    }
});
Vue.directive('background', (el, binding) => { // 一般只用到钩子函数bind+inserted，直接用这种简写方式
    el.style.background = binding.value;
});
```

### 2.自定义指令-局部
```html
<p v-font-size="24">color</p>
<input type="text" value="hhhhhhh" v-color="'red'" v-focus>
```
```javascript
var vm = new Vue({
    el: '#app',
    data: {

    },
    directives: {
        fontSize(el, binding) { // 一般只用到钩子函数bind+inserted，直接用这种简写方式
            el.style.fontSize = binding.value + 'px';
        },
        color: {
            bind(el, binding) {
                el.style.color = binding.value;
            },
            inserted(el, binding) {
                el.focus();
            }
        }
    }
})

// 或者
vm.directives = {  
    fontSize(el, binding) { // 一般只用到钩子函数bind+inserted，直接用这种简写方式
        el.style.fontSize = binding.value + 'px';
    },
    color: {
        bind(el, binding) {
            el.style.color = binding.value;
        },
        inserted(el, binding) {
            el.focus();
        }
    }};
```

## 6.动画
###  1.内置`<trasition></trasition>`
- 使用样式:CSS中默认以v-开头，指定了transition 的name之后 ，CSS中以name开头
```html
<head>
    .v-enter,
    .v-leave-to {
        opacity: 0;
        transform: translateX(100px);
    }

    .v-enter-active,
    .v-leave-active {
        transition: all 1s;
    }

    .my-enter,
    .my-leave-to {
        opacity: 0;
        transform: translateY(100px);
    }

    .my-enter-active,
    .my-leave-active {
        transition: all 1s;
    }
    </style>
</head>
<body>
    <div id="app">
        <div class="box">
            <button @click="flag1=!flag1">toggle</button>
            <transition>
                <h3 v-show="flag1">这是一个h3</h3>
            </transition>
        </div>
        <hr>
        <div class="box">
            <button @click="flag2=!flag2">toggle</button>
            <transition name="my">
                <h3 v-show="flag2">这是一个h3</h3>
            </transition>
        </div>
    </div>
</body>
```
```css
/*定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 v-leave 被删除)，在过渡/动画完成之后移除。*/
/*v-move对于设置过渡的切换时机和过渡曲线非常有用*/
    .v-move{
        transition: all 1s;
    }

    .v-leave-to {
        position: absolute;
    }
```

- 使用类名
`他们的优先级高于普通的类名，这对于 Vue 的过渡系统和其他第三方 CSS 动画库，如 Animate.css 结合使用十分有用。`
    + enter-class
    + enter-active-class
    + enter-to-class (2.1.8+)
    + leave-class
    + leave-active-class
    + leave-to-class (2.1.8+)
```html
<div class="box">
    <button @click="flag1=!flag1">toggle</button>
    <transition enter-active-class="slideOutRight" leave-active-class="rotateOutUpLeft" :duration="100">
        <h3 v-show="flag1" class="animated">这是一个h3</h3>
    </transition>
    <transition enter-active-class="slideOutRight" leave-active-class="rotateOutUpLeft" :duration="{enter:1000,leave:300}">
        <h3 v-show="flag1" class="animated">这是第二个h3</h3>
    </transition>
</div>
```
- 组件动画
    + name属性：指定name，类名用name-开头
    + mode属性：组件切换时候指定切换方式in-out/out-in
```html
<transition mode="out-in">
    <component :is="'login'"></component>
</transition>
```
- v-for循环出来的元素组`<transition-group></transition-group>`
    + name属性：指定name，类名用name-开头
    + appear属性：可以实现元素组第一次加载时的入场动画
    + tag属性：指定将渲染为什么元素，不指定会渲染为span
    + v-move对于设置过渡的切换时机和过渡曲线非常有用

```html
<transition-group appear tag="ul">
    <li v-for="(ele,index) in list" :key="index"></li>
</transition-group>
```

### 2.钩子函数--多用于设置半场动画
- 推荐对于仅使用 JavaScript 过渡的元素添加 v-bind:css="false"，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响
- 当只用 JavaScript 过渡的时候，在 enter 和 leave 中必须使用 done 进行回调。否则，它们将被同步调用，过渡会立即完成。
```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```
```javascript
methods: {
    /*动画的生命周期函数*/
    beforeEnter(el) { // 生命周期函数-1-动画入场前
        // 设置动画入场前的起始样式
    },
    enter(el, done) { // 生命周期函数-2-动画入场
        // 设置动画入场，样式、动画时间及方式
        el.offsetWidth; // el.offsetHeight;el.offsetTop;el.offsetLeft; 这句话没有特定的作用，但要动画必须加上

        done(); // 这里的done就是afterEnter函数的引用，不调用的话，afterEnter会出现延迟
    },
    afterEnter(el) { // 生命周期函数-3-动画入场后
        // 设置动画入场完成的样式
    },
    beforeleave(el) {

    },
    leave(el) {
        done(); // 
    },
    afterleave(el) {

    },
}
```

## 7.自定义组件
- 拆分vue实例代码量
- 需要什么页面小功能调用什么组件
- 组件化：前端，从UI界面角度划分，模块化：后端，从业务功能角度划分
- 注意：组件template必须有且只有一个根元素包裹

### 1.自定义组件-全局 
- 自定义
```html
<body>
    <div id="app">
        <h1>全局组件</h1>
        <all-hello1></all-hello1>
        <all-hello2>adf</all-hello2>
        <all-hello3></all-hello3>
        <all-hello4></all-hello4>
        <hr>
        <h1>私有组件</h1>
        <my-hello1></my-hello1>
    </div>
    <!-- 组件模板 -->
    <template id="template1">
        <div>
            <span>all-hello4</span><span>all-hello4</span>
        </div>
    </template>
</body>
```
```javascript
/*名称可以驼峰，可以非驼峰，但html中必须横杠*/
// 组件1
var allHello1 = Vue.extend({
    template: '<h3>all-hello1</h3>'
});

Vue.component('all-hello1', allHello1)

// 组件2
Vue.component('allHello2', Vue.extend({
    template: '<h3>all-hello2</h3>'
}));

// 组件3
Vue.component('all-hello3', {
    template: '<h3>all-hello3</h3>'
});

// 组件4
Vue.component('all-hello4', {
    template: '#template1'
});

new Vue({
    el: '#app',
})
```

### 2.自定义组件-私有
```javascript
new Vue({
    el: '#app',
    data: {},
    components: {
        myHello1: {
            template: '<h3>my-hello1</h3>'
        }
    }
})

// 或者使用render(webpack中使用)
new Vue({
    el: '#app',
    data: {},
    render(createdEle) { // 将组件compt放在#app处
            return createdEle(compt); // 返回一个组件对象，并替换el指定的容器
    }
})
```

### 3.参数
```javascript

new Vue({
    el: '#app',
    components: {
        myHello1: {
            template: `
                        <div>
                            <button @click="changeBGC" :style="{width:'100px',height:'50px'}">点击变颜色</button>
                            <h3 :class="[flag?'red':'green']">my-hello1</h3>
                        </div>
            `,
            data() { // data是一个闭包函数
                return {
                    flag: true,
                    msg: '私有组件啊'
                };
            },
            props: ['父组件传递过来的值'],// 定义接收父组件传值的变量
            methods: {
                changeBGC() {
                    this.flag = !this.flag;
                }
            }
        }
    }
})
```

### 4.组件切换`<component :is="'要展示的组件名称'"></component>`
```html
<component :is="whatComponenet"></component>
```

### 5.组件传值
- 父组件向子组件传数据
    + 1.通过v-bind将父组件的值绑定给子组件的变量
    ```html
    <div id="app">
        <hello v-bind:prop_msg="msg" :prop_title="title"></hello>
    </div>
    ```
    + 2.子组件props数组中定义接收父组件传数据的变量
    ```javascript
    new Vue({
        el: '#app',
        data: {
            msg: '这是父组件msg',
            title: '父组件的title'
        },
        components: {
            hello: {
                template: '<h3>这是H3----父组件prop_title:{{prop_msg}}---prop_title{{prop_title}}</h3>',
                data() {
                    return {
                        prop_title: '子组件的title'
                    }
                },
                // 用来定义接收父组件传数据的变量，1.只读（最好不要修改），2.会覆盖子组件data中的属性值
                props: ['prop_msg', 'prop_title']
            }
        }
    })
    ```
- 父组件向子组件传方法
    - 1.通过v-on将父组件的方法绑定给子组件的变量
    ```html
    <div id="app">
        <hello @son_func="parent_func"></hello>
    </div>
    ```
    - 2. 子组件的方法中通过this.$emit()调用方法
    ```javascript
    new Vue({
        el: '#app',
        data: {
            dataObj: null
        },
        methods: {
            parent_func(str, data_from_son) {
                data_from_son.id = 2;
                this.dataObj = data_from_son;
                console.log(this.dataObj);
                console.log('这是父组件方法' + '---' + str);
            }
        },
        components: {
            hello: {
                template: '<button @click="click_btn">点击获取父组件方法，并向父组件传递子组件数据</button>',
                data() {
                    return {
                        son_obj: {
                            id: 1,
                            msg: '子组件的数据'
                        }
                    }
                },
                methods: {
                    click_btn() {
                        // 参数1：子组件的方法名
                        // 参数2-n：父组件传递过来的方法的参数
                        // 注意：名字不能使用驼峰
                        this.$emit('son_func', '点击了子组件btn', this.son_obj)
                    }
                }
            }
        }
    })
    ```

## 8.路由
- > https://router.vuejs.org/zh/
- 后端路由：对于普通的网站，所有超链接都是URL地址，所有URL地址都对应服务器上的资源。
- 前端路由：对于单页面应用程序，主要通过URL的hash（哈奇/锚点）来实现不同页面的切换，hash有一个特点，HTTP请求不会包含hash相关的内容

### 1.使用
- 1.在vue后面引入vue-router
- 2.引入后window下会有一个VueRouter的路由构造函数，且url最后上会自动加上#/
- 3.配置路由
```javascript
var obj_router = new VueRouter({
    routers: [
        path: '/login',
        component: {}// 组件对象        
    ]
})
```
- 4.将配置好的路由与Vue的实例vm进行绑定
```javascript
new Vue({
    el: '#app',
    router : obj_router
})
```
- 5.html中使用`<router-view></router-view>`占位符，用来标识路由组件展示的地方
- 6.html中来跳转路由
```html
<a href="#/home">主页/不会添加active类名</a>
<router-link to="/login">登录/默认转换为a标签</router-link>
<router-link to="/register" tag="span">注册/使用tag属性处理标签</router-link>
```
- 7.被激活的路由，会给`<router-link></router-link>`元素身上加上router-link-active类名，可以通过linkActiveClass属性进行修改
- 8.路由动画，给路由加上`<transition></transition>`标签

### 2.配置路由
```javascript
var obj_router = new VueRouter({
    routes: [ // 这里是routes不是routers
    {
        path: '/',
        redirect: '/account', // 让根路径跳转至login路由
        name: 'root' ,// 指定当前路由的别名
        meta: {keepAlive:true} // 
    },{
        path: '/account', // path后面的路由名称以’/‘开头，表示从根路径开始
        component: {}, // 组件对象 
        children: [{
            path: 'login',// 不带’/‘，带了表示从根路由开始
            component: {},
        }],
        components: { // 路由引入多个组件时，通过name区分，默认为default
            default,
            left,
        }
    },{
            path: '*', // 遇到未配置的路由时--必须放在最下面
            redirect: '/home'
    }],
    linkActiveClass:'类名', // 修改激活路由的元素的默认active类名
}）
```

### 3.路由传参
- 使用查询字符串，调用this.$route.query
- 使用占位符，调用this.$route.params
```html
<router-link to="/login?id=12" tag="a">登录组件</router-link>
<router-link to="/register/13/tom">注册组件</router-link>
<router-link to="/user/13">用户组件</router-link>
<router-view></router-view>
```
```javascript
var router = new VueRouter({
    routes: [{
        path: '/login',
        component: {
            template: '<h3>登录组件---{{$route.query.id}}</h3>',
            data() {
                return {
                    msg: '啊啊啊啊'
                }
            },
            created() {
                console.log('-----登录组件-----');
                console.log(this.$route);
            }
        }
    }, {
        path: '/register/:id/:name', // :id占位符
        component: {
            template: '<h3>注册组件---id:{{$route.params.id}}---name：{{$route.params.name}}</h3>',
            created() {
                console.log('-----注册组件-----');
                console.log(this.$route);
            }
        }
    }, {
        path: '/user/:id/:name?', // :id占位符，? 表示可传可不传
        component: {
            template: '<h3>用户组件---id:{{$route.params.id}}---name：{{$route.params.name}}</h3>',
            created() {
                console.log('-----用户组件-----');
                console.log(this.$route);
            }
        }
    }]
});
new Vue({
    el: '#app',
    router
})
```

### 4.子路由
```html
<div id="app">
    <router-link to="/account">account</router-link>
    <router-view></router-view>
</div>
```
```javascript
var compt_account = {
    template: `
        <div>
            <h3>账户组件</h3>
            <router-link to="/account/login">登录</router-link>
            <router-link to="/account/register">注册</router-link>
            <router-view></router-view>
        </div>
    `
};

var compt_login = {
    template: '<h3>登录组件</h3>'
};

var compt_register = {
    template: '<h3>注册组件</h3>'
};

var router = new VueRouter({
    routes: [{
        path: '/account',
        component: compt_account,
        children: [{
            path: 'login', // 这里不能加/，否则从根目录开始
            component: compt_login
        }, {
            path: 'register',
            component: compt_register
        }]
    }]
});
new Vue({
    el: '#app',
    router
})
```

### 5.路由引入多个组件
```html
<div id="app">
    <router-view></router-view>
    <router-view name="box_left"></router-view>
    <router-view name="box_right"></router-view>
</div>
```
```javascript
var box_header = {
    template: `<h1 :style="{height:'200px',background:'pink'}">这是头部</h1>`
};

var box_left = {
    template: `<aside :class="['left']">这是left</aside>`
};

var box_right = {
    template: `<main :class="['right']">这是right</main>`
};

var router = new VueRouter({
    routes: [{
        path: '/',
        components: {
            default: box_header,
            box_left,
            box_right
        }
    }]
});

new Vue({
    el: '#app',
    router
})
```

### 6.路由缓存<keep-alive></keep-alive>
    + 如果需要频繁切换路由，这个时候就可以考虑用keep-alive了，来达到避免数据的重复请求的目的。
```
<keep-alive>
    <router-view> </router-view>
</keep-alive>
```

### 7.导航守卫
#### 1.全局前置守卫router.beforeEach
```javascript
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
    // to: Route: 即将要进入的目标 路由对象
    // from: Route: 当前导航正要离开的路由
    // next: Function: 确保要调用 next 方法，否则钩子就不会被 resolved。
        // next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。

        // next(false): 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。

        // next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 next 传递任意位置对象，且允许设置诸如 replace: true、name: 'home' 之类的选项以及任何用在 router-link 的 to prop 或 router.push 中的选项。

        // next(error): (2.4.0+) 如果传入 next 的参数是一个 Error 实例，则导航会被终止且该错误会被传递给 router.onError() 注册过的回调
})
```

#### 2.全局后置钩子router.afterEach
```javascript
router.afterEach((to, from) => {
  // ...
})
```

#### 3.路由独享的守卫
```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

#### 4.组件内的守卫
- 离开守卫通常用来禁止用户在还未保存修改前突然离开。该导航可以通过 next(false) 来取消。
```javascript
beforeRouteLeave (to, from , next) {
  const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (answer) {
    next()
  } else {
    next(false)
  }
}
```

```javascript
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```

#### 5.完整的导航解析流程
- 1.导航被触发。
- 2.在失活的组件里调用离开守卫。
- 3.调用全局的 beforeEach 守卫。
- 4.在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
- 5.在路由配置里调用 beforeEnter。
- 6.解析异步路由组件。
- 7.在被激活的组件里调用 beforeRouteEnter。
- 8.调用全局的 beforeResolve 守卫 (2.5+)。
- 9.导航被确认。
- 10.调用全局的 afterEach 钩子。
- 11.触发 DOM 更新。
- 12.用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。

## 9 vuex：状态管理模式，公共数据管理工具
- 让整个应用程序共享某些状态/数据，方便整个应用中的任何组件读取/修改这些状态/数据
- 只有组件之间共享的数据才需要存在vuex中

### 1.使用
- 1.在vue后面引入vuex
- 2.安装vuex
`Vue.use(Vuex)`
- 3.配置vuex
```javascript
var obj_router = new Vuex.store({
    state: { // 定义放入store中的属性
        count: 0
    },
    // 定义放入store的方法
    // 每个方法第一个参数必须是state
    // 推荐通过调用方法的方式改变state中的属性
    mutations: {
        increment (state) {
            state.count++
        },
    },
    // 只能获取state中的属性，不能改变，改变在mutations中
    // 实时监控所绑定的值
    getters: {
        doneTodos: state => {
            return state.todos.filter(todo => todo.done)
        }
    },
    // Action 提交的是 mutation，而不是直接变更状态
    // Action 可以包含任意异步操作
    //Action 函数接受一个与 store(或者module) 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters
    actions: {
        increment (context) {
        context.commit('increment')
    },
    // Vuex 允许我们将 store 分割成模块（module）。
    // 每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块
    modules: {
      a: moduleA,
      b: moduleB
    }
})
```
- 4.将配置好的store与Vue的实例vm进行绑定
```javascript
new Vue({
    el: '#app',
    store
})
```
- 5.使用
    + state中的属性：$store.state.count
    + mutation中的方法：$store.commit('increment',params)--最多只能传2个参数
    + getters中的属性：$store.getters.doneTodos
    + action中的方法：$store.dispatch('increment',params)--最多只能传2个参数

## 常用API
### this.$emit:触发函数
`this.$emit('函数名',函数的参数...)`

### Vue.nextTick( [callback, context] )延迟运行-DOM更新后执行callback
- 直接调用，2个参数（回调函数和上下文），如果没有传入参数就作为promise使用
- 在Vue生命周期的created()进行的DOM操作一定要放在this.$nextTick()的回调函数中
- 在数据变化后要执行的某个操作，而这个操作需要使用随数据改变而操作的DOM结构的时候，这个操作都应该放进this.$nextTick()的回调函数中。
```html
<div id="app">
    <div ref="msgDiv">{{msg}}</div>
    <hr>
    <div>$nextTick上面: {{msg1}}</div>
    <div>$nextTick里面: {{msg2}}</div>
    <div>$nextTick下面: {{msg3}}</div>
    <hr>
    <button @click="changeMsg"> 按钮</button>
</div>
```
```javascript
new Vue({
    el: '#app',
    data: {
        msg: 'hello-vue',
        msg1: '',
        msg2: '',
        msg3: ''
    },
    methods: {
        changeMsg() {
            console.log(1);
            this.msg = "hello-world";
            this.msg1 = this.$refs.msgDiv.innerHTML; // 这时候DOM还没更新

            this.$nextTick(() => {
                // DOM更新后才会执行的代码
                this.msg2 = this.$refs.msgDiv.innerHTML;
            });

            this.msg3 = this.$refs.msgDiv.innerHTML;
        }
    }
})
```

### this.$route:获取路由参数对象

### this.$router:获取路由导航对象

### this.$refs:获取元素，组件（父组件获取子组件的数据和方法）
- 1.元素/组件添加 ref 属性来标识
```html
<div id="app">
    <button @click="click_getDOM" ref="getDOM">点击获取元素</button>
    <br>
    <button @click="click_getCompt">点击获取组件</button><br>
    <hello ref="compt"></hello>
</div>
```
- 2.父组件方法中调用this.$refs(对象)获取用 ref 标识的元素/组件
```javascript
var hello = {
    template: '<h3>hello组件</h3>',
    data() {
        return {
            msg: '子组件的数据'
        }
    },
    methods: {
        func_son() {
            console.log('调用了子组件方法');
        }
    }
};
new Vue({
    el: '#app',
    data: {},
    methods: {
        click_getDOM() {
            console.log(this.$refs.getDOM);
        },
        click_getCompt() {
            console.log(this.$refs.compt);
            console.log(this.$refs.compt.msg);
            this.$refs.compt.func_son();
        }
    },
    components: {
        hello
    }
})
```

## 插件/包
### 1.http请求
- vue-resource：
    + https://github.com/pagekit/vue-resource
    + 引入包后，Vue上挂载了一个http（http:{}）属性
        - get
        - post :option参数中，emulateJSON最好设置为true，表示用application/x-www-form-urlencoded(表单形式)发送请求
        - jsonp
        - delete
        - head
        - put
        - patch
    + 全局：
        - 设置
        ```javascript
        Vue.http.options.root = '/root';// 使用的时候用相对路径即可，带了/表示根路径
        Vue.http.headers.common['Authorization'] = '';
        ```
        - 使用
        ```javascript
        Vue.http.get('url',[option]).then(successCallback,errorCallback);
        Vue.http.get('url',[body],[option]).then(successCallback,errorCallback);
        ```
        
    + 私有：
        - 设置
        ```javascript
        app.http= {
            root: '/root',// 使用的时候用相对路径即可，带了/表示根路径
            headers:{}
        };
        ```
        - 使用
        ```javascript
        this.$http.get('url',[option]).then(successCallback,errorCallback);
        this.$http.get('url',[body],[option]).then(successCallback,errorCallback);
        ```

- axios

# webpack 4.x
## node工具
- nvm： node version manager node版本管理
- npm： node package manager node包管理
- nrm： node registry manager node注册地址管理，提供大陆境内镜像地址
- cnpm： 大陆装包工具--类似于npm

## 介绍
- 1.为什么使用管理工具
    + 请求静态文件多，网页加载慢，因为发起很多二次请求。
        - JS、CSS压缩合并
        - 小图片转精灵图，或base64编码（src指向base64编码后的字符串）和html第一次就一起请求过来
    + 包/文件依赖关系复杂
- 2.webpack是基于node开发的包管理工具
- 3.gulp和webpack
    + gulp：基于task任务，每个小任务都要建一个task，项目太大的话task太多，不适用于特大项目
    + webpack： 基于项目/模块

## 安装
- npm install webpack -g
- npm install webpack --save-dev

## main.js 使用ES6语法
```javascript
// 使用ES6语法，浏览器兼容性不高可能报错，需要webpack打包处理
// 不加./表示在当前目录下的node_modules文件夹下面找
import $ from 'jquery';
// webpack默认只能处理JS文件，需要处理非JS文件需要loader
import './css/index.css';

$(function() {
    $('li:odd').css({ background: 'pink' });
    $('li:even').css({
        background: () => {
            return# +'lightblue';
        }
    });
});
```
## webpack.config.js webpack配置文件
```javascript
// webpack的配置文件，基于node开发的，所以是node 语法
// 依赖path模块
const path = require('path');

// 向外暴露一个配置对象
module.exports = {
    entry: path.join(__dirname, './main.js'), // 入口文件地址
    output: { // 输入文件配置
        path: path.join(__dirname, './dist'), // 输入文件目录
        filename: 'bundle.js' // 输出文件名
    },
    devServer: { // 设置webpack-dev-server（推荐在package中使用，这种比较麻烦）
        // open：默认打开浏览器
        // port：端口
        // contentBase：根目录
        // hot：热重载（不重新生成新文件，只局部更新，浏览器无刷新重载CSS，对JS无效）
        // host：指定项目运行的服务器地址
        open: true,
        port: 3000,
        contentBase: 'src',
        // 这里启用热重载还需要引入webpack模块
        // 1.hot:true
        // 2.const webpack = require('webpack'),
        // 3.配置插件 new webpack.HotModuleReplacementPlugin()
        hot: true,
        host: '192.168.1.1',
    },
    plugins: [ //配置插件--需要先require('插件')
        new webpack.HotModuleReplacementPlugin(),// 配置热重载第三步
        new htmlWebpackPlugin({ // 根据指定模板页面生成内存镜像
            template: path.join(__dirname,'index.html'),
            filename: '生成文件名称'
        }),
        // 4.x需要配置 const VueLoaderPlugin = require('vue-loader/lib/plugin');
        new VueLoaderPlugin(), 
    ],
    module: { // 配置第三方文件/模块
        rules: [ // 匹配规则，多个loader时 从右到左依次调用
            {test: /\.css$/ , use: ['style-loader','css-loader']},
            {test: /\.less$/ , use: ['style-loader','css-loader','less-loader']},
            {test: /\.scss$/ , use: ['style-loader','css-loader','sass-loader']},
            // use后面可以传参
            // limit：转换图片编码的最大大小
            // name：控制文件名【name】-保持原名，[ext]-保持原后缀[hash:8]取32位哈希的前8位
            {test: /\.(jpg|png|gif|jpeg|bmp)$/ , use: 'url-loader?limit=7624&name=[name].[ext]'},
            {test: /\.(ttf|eot|svg|woff|woff2)$/ , use: 'url-loader'},
            // babel-exclude：排除（不编译）的文件/目录
            {test:/\.js$/ , use: 'babel-loader' , exclude: /node_modules/}
        ]
    }
}
```

## package.json
```json
"script": {// 可以定义npm运行的脚本，控制台直接npm build即可
    // 设置webpack-dev-server（推荐使用这种方式，不推荐在webpack.config.js中配置，比较麻烦）
    // --open：默认打开浏览器
    // --port：端口
    // --contentBase：根目录
    // --hot：热重载（不重新生成新文件，只局部更新，浏览器无刷新重载CSS，对JS无效）
    // host：指定项目运行的服务器地址
    "build": "webpack-dev-server --open --port 3000 --contentBase src --host 192.168.1.1"
}
```

## webpack-dev-server 本地服务器
- 要求必须在项目本地安装webpack
- 内存中生成bundle.js文件

## html-webpack-plugin 插件
- 根据指定模板页面生成内存镜像
- 默认引入webpack-dev-server生成并存放在内存中的buddle.js文件
- 在plugins中设置

## loader 加载器
- css：style-loader css-loader
- less：less-loader（内部依赖less）
- sass/scss（新版后缀叫scss）: sass-loader(内部依赖node-sass，node-sass用npm一般装不下来，cnpm可以)
- url：url-loader(内部依赖 file-loader)：匹配文件和字体等url
    + 默认将图片转成base64字符串，不管大图小图
    + 大图不需要转不然字符串太长了

- babel：ES6//ES7高级语法转化为低级语法(babel-core@6版本，新版@babel/core不适用)
    + 转化工具（根据转化关系进行转化）：babel-core babel-loader babel-plugin-transform-runtime
    + 语法（高级语法到低级语法的转化关系）：babel-preset-env（之前是babel-preset-env，babel-preset-env包含所有es的语法） babel-preset-stage-0
    + 1.module的rules规则中添加babel-loader的匹配规则
    + 2.项目根目录中创建.babelrc文件
    ```json
    {
        "presets": ["env" , "stage-0"],
        "plugins": ["transform-runtime"]
    }
    ```

- vue: vue-loader(内部依赖 vue-template-compiler，webpack3.x版本)
    + 4.x需要在plugins中配置 const VueLoaderPlugin = require('vue-loader/lib/plugin');

# webpack+VUE
### 基本配置
- npm install vue -S
- main.js 导入 `import Vue from 'vue'` 此时导入的是一个runtime的阉割版
- 要使用传统网页方式开发需要引入vue.js
    + 修改导入路径
    + 修改vue/package.json中main对应的文件名
    + webpack.config.js中的resolve属性配置默认值
    ```javascript
    resolve: {// 设置vue导入时候包的路径
        alias: {
            'vue$': 'vue/dist/vue.js'
        }
    }
    ```

### 组件
- 安装 vue-loader(内部依赖 vue-template-compiler)
- module的roles中配置vue-loader
- html组件（后缀.vue）
```html
<template>
    <h1>这是login组件{{123}}</h1>
</template>
<script>
    export default {
        data(){
            return {
                msg: 123
            }
        },
        methods: {}
    }
</script>
<style scoped lang="sass">
<!-- scoped表示独立作用域，style只作用在当前组件中 -->
<!-- lang表示使用哪种语言编写CSS -->
</style>
```
- 导入组件，并渲染
```javascript
import login from './login.vue'

new Vue({
    el: '#app',
    data: {},
    render(createdEle) { // 将组件compt放在#app处
            return createdEle(compt); // 返回一个组件对象，并替换el指定的容器
    },
    // 或者写成--箭头函数简化而来
    render: c=>c(login)
})
```

### 路由
- 安装 vue-router(内部依赖 vue-template-compiler) 
- main.js中导入vue-router
```javascript
import VueRouter from 'vue-router;
Vue.use(VueRouter); // 手动安装VueRouter
```
```javascript
var obj_router = new VueRouter({
    routes: [{
        path: '/',
        redirect: '/login'
    }, {
        path: '/login',
        component: {
            template: '<h3>登录组件</h3>'
        }
    },
    linkActiveClass: 'my-active'
});
new Vue({
    el: '#app',
    data: {},
    router: obj_router
})
```

# 其他
## node中的模块化
- 导出模块
```javascript
module.exports obj1;
exports obj2;
return obj3;
```
- 导入模块
```javascript
var obj = require('./index.js');
```

## ES6中的模块化
- 导出模块
```javascript
// 每个模块只能有一个export default，导入模块可以用任意变量接收
export default obj1; 
// 一个模块可以有多个export，导入模块只能用这里暴露的变量接收，但可以收到后修改
export obj2; 
export obj3;
export obj4;
```
- 导入模块
```javascript
// export default导出的数据可以随便起名
// export导出的数据，必须用{}接收；必须名称对应，但可以通过as增加别名；可以不全接收
import obj_my1 {obj2 , obj4 as obj_my2} from 'index.js'
```

- 数组新方法
    - forEach
    - some
    - every
    - findIndex
    - filter
    - map
    - indexof
    - splice
  字符串新方法
    - includes
    - indexof
    - padStart('填充完字符串总长度','填充物')
    - padEnd
- localStorage中只支持存放字符串
- div.domObject.getBoundingClientRect() 获取元素到屏幕的四个距离

## mint-ui： 基于vue的移动端组件库

## MUI：类似于bootstrap，并不是基于vue的，更适合移动端

