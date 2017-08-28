# -vue-
浅析vue原理
浅析vue原理
===
网上看了好多关于vue的文章，对vue的剖析基本一致。有些过于繁琐，让人懵懂。基于[http://www.cnblogs.com/kidney/p/6052935.html?utm_source=gold_browser_extension](url)这篇博文，自己动手实践了一下，就自己的理解来阐述这段代码。

观察者模式
-----
```
function observe(obj, vm) { 
    Object.keys(obj).forEach(function (key) { 
        defineReactive(vm, key, obj[key]) 
    }) 
}   
 ```
访问器属性
-----
```
function defineReactive(obj, key, val) {
    var dep = new Dep();
    Object.defineProperty(obj, key, {
        get: function () {
            if (Dep.target) dep.addSub(Dep.target)
            return val
        },
        set: function (newValue) {
            if (newValue === val) return
            val = newValue
            console.log('值是这么变化的，正确的值是' + val)
            dep.notify()
        }
    })
}
```
订阅发布模式
-----
```
function Dep() {
    this.subs = []
}
Dep.prototype = {
    addSub: function (sub) {
        this.subs.push(sub);
    },
    notify: function () {
        this.subs.forEach(function (sub) {
            sub.update();
        })
    }
}
```
文档拦截
-----
```
function nodeToFragment(node, vm) {
    var flag = document.createDocumentFragment();
    var child;
    while (child = node.firstChild) {
        compile(child, vm);
        flag.appendChild(child)
    }
    return flag;
}
```
编译
-----
```
function compile(node, vm) {
    var reg = /\{\{(.*)\}\}/;
    if (node.nodeType === 1) {
        var attr = node.attributes;
        for(var i = 0; i < attr.length; i++) {
            if (attr[i].nodeName == 'v-model') {
                var name = attr[i].nodeValue;
                node.addEventListener('input', function (e) {
                    vm[name] = e.target.value
                })
                node.value = vm[name];
                node.removeAttribute('v-model');
            }
        }
    }
    if (node.nodeType === 3) {
        if (reg.test(node.nodeValue)) {
            var name = RegExp.$1;
            name = name.trim();
            // node.nodeValue = vm[name];
            new Watcher(vm, node, name)
        }
    }
}
```
监听数据变化文本响应
-----
```
function Watcher(vm, node, name) {
    Dep.target = this;
    this.name = name;
    this.node = node;
    this.vm = vm;
    this.update();
    Dep.target = null;
}
Watcher.prototype = {
    update: function () {
        this.get();
        this.node.nodeValue = this.value;
    },
    get: function () {
        this.value = this.vm[this.name];
    }
}
```
Vue实例对象
-----
```
function Vue(options) {
    this.data = options.data;
    var data = this.data;
    observe(data, this);
    var id = options.el;
    var dom = nodeToFragment(document.getElementById(id), this);
    document.getElementById(id).appendChild(dom)
}
```
创建Vue实例对象
-----
```
var vm = new Vue({
    el: 'app',
    data: {
        text: 'hello world',
        demo: 'ppp'
    }
});
```
html代码
-----
```
<div id="app">
        <input type="text" v-model="text">
        {{text}}
    </div>
```
