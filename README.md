# Vue组件通信

Vue组件的通信关系：父子关系，兄弟关系、祖先后代关系、无直接（非简单）关系。

## 父子关系
父子关系的双工通信方式，主要有以下几种方法：
1. 父向子传递关系：**props、$attrs**
2. 父向子访问关系：**$children、ref**
3. 子向父亲传递关系：**$emit**
4. 子向父亲访问关系：**$parent、$listeners**

Vue提倡一种理念：祖先(层级高的)组件向子孙(层级低的)组件传递的值，是不能直接修改的。下面通过代码详细说明父子关系组件通信的几种情况。

假定APP是最高级根组件，存在A组件和E组件作为App组件的儿子。

App.vue
```
<template>
	<div style="border:1px solid #343443;padding:20px;">
		<div style="display:inline-block;">
		  <A :msg="msg" ref="A" @toAlert="Alert" />
		  <button @click="onClick" style="margin:10px;">
		  	点击后，修改A组件data中refValue值
		  </button>
    	</div>
		<div style="display:inline-block;margin-left:10px;">
		  <E :msg1="msg1" @eval="getE($event)"/>
		  <p>等待子组件抛出值：{{childVal}}</p>
		  <button @click="onClick1" style="margin:10px;">
		  	点击后，修改E组件data中childValue值
		  </button>
    	</div>
	</div>
</template>
<script>
import A from "./components/A";
import E from "./components/E";

export default {
  name: "app",
  data() {
    return {
      msg: `子组件props方式：${this.$options.name}（父）向A组件（子）传递属性msg`,
      msg1: `子组件$attrs方式：${this.$options.name}（父）向E组件（E）传递属性msg1`,
	  parentVal:'app中的值',
	  childVal:"用于保留子组件抛出的值"
    };
  },
  components: {
    A,
    E
  },
  methods: {
	Alert() {
      alert("来自APP组件的弹框");
    },
    onClick() {
      this.$refs["A"].refValue =
	  `${this.$options.name}修改了A组件的refValue值`;
    },
    onClick1() {
      for (let i in this.$children) {
        if (this.$children[i].$options.name === "E") {
          this.$children[i].childValue = 
		  `${this.$options.name}修改了E组件的childValue`;
        }
      }
    },
	getE(e){
		this.childVal = e;
  	},
};
</script>
```
在APP组件（父）中向A组件（子）和E组件（子）分别绑定msg和msg1，APP中定义方法onClick方法，点击按钮通过ref引用组件A的实例，从而访问A组件的数据和方法，onClick1方法通过$children的方式遍历查找目标孩子组件实例，这里查找了E组件，从而访问E组件的数据和方法，这里可以封装成更通用的方法。childVal保存子组件抛出的值，当子组件emit后，触发getE。


A.vue
```
<template>
 <div style="border:1px solid #343443;padding:20px;">
     <h1>A组件：B组件是我儿子</h1>
     <p>{{msg}}</p>
     <p>{{refValue}}</p>
     <button style="margin:10px;" @click="$listeners.toAlert">A组件：调用app组件（根）的方法</button>
 </div>
</template>
<script>
export default {
    name:"A",
    props:{
        msg:String
    },
    data(){
        return {
            refValue:`${this.$options.name}组件data中refValue值`,
        }
    },
}
</script>

```
A组件（子）通过props对象保留App（父）组件传递的值，refValue供App组件通过ref方式访问，button按钮点击后，通过$listeners的方式访问App组件中的方法。

E.vue
```
<template>
 <div style="border:1px solid #787878;padding:10px;">
     <h1>E组件：A组件是我兄弟</h1>
     <p>{{$attrs.msg1}}</p>
     <p>{{childValue}}</p>
	 <p>访问父组件值{{$parent.parentVal}}</p>
	 <button @click="onClick"><button>
     <F/>
 </div>
</template>

<script>
export default {
    name:"E",
    data(){
        return{
            childValue:`${this.$options.name}组件中data的childValue值`, 
			emitVal:'E组件的emit出来的值'
        }
    },
	methods:{
		onClick(){
			this.$emit('eval',emitVal)
		}
	}
}
</script>
```
E组件（子）通过$attrs接收APP组件（父）传递的属性，childValue供App中onClick2方法访问,在p标签中通过$parent的方式访问APP的值，点击按钮，$emit自定义事件向App组件抛出一个值。

**根据官方文档中的说明，指出$parent、$children应当适当使用！**

其中对于子组件访问父组件所用的$parent和$listeners，这种方式可以向下传递，也就是继续把这两个值向A或者E的子组件传递，后续会引出A组件的儿子B组件和A组件的孙子C和D组件，这个故事还没结束。


## 兄弟关系
按照上面例子来看，A组件和E组件就是兄弟关系，兄弟关系使用搭桥的方式，也就是通过他们共同的父亲来传值。这里不谈总线模式，总线模式可以说是适用于任何组件关系的传值。

这里忽略上述已经表述过的代码!

A.vue
```
<script>
	export default {
	...
		data(){
			return {
				...
				brother:"A组件的值"
				...
			}
		}
		methods:{
			...
			onClick(){
				this.$parent.$emit('aclick',this.brother);
			}
			...
		},
	...
	}
</script>
```
通过按钮来触发这个onClick事件，A和E共同的父亲就是this.$parent,这就是搭桥的思量，也就是A和E组件的父亲等待接受一个值，这个感知的过程在父亲组件，但实际的传值和取值本身还是在各自组件内。

E.vue
```
<script>
	export default{
	...
		data(){
			return {
				...
				brotherVal:"用于接受A组件（兄）的值"
				...
			}
		},
		mounted(){
			...
			this.$parent.$on('aclick',(e)=>{
				this.brotherVal = e;
			})
			...
    	}
	...
	}

</script>
```
这里只演示了A向E的传值过程，按照同样的道理，反之，E也可以向A进行传值。


## 祖先后代关系

-  祖先向子孙传值： ** $listeners、provide & inject**
-  子孙向祖先传值： 暂时没有想到什么好方法、没有办法就用总线、哈哈

那这里就主要展示$listeners和provide&inject如何使用，简单描述下组件结构，上述提到App有儿子A和E，现在假设A组件有儿子B，B组件有儿子D组件，下面的代码会有所省略。

---
$listeners 演示代码
App.vue
```
	<template>
		...
		 <A :msg="msg" ref="A" @toAlert="Alert" />
		...
	</template>
	<script>
		export default{
			...
			methods:{
				Alert() {
				  alert("来自APP组件的方法");
				},
			}
			...
		}
	</script>
```


A.vue

```
<template>
	...
	<B v-on="$listeners"/>
	...
</template>
```

B.vue

```
<template>
	...
	 <D v-on="$listeners"/>
	...
</template>
```

D.vue
```
<template>
	...
	<button @click="$listeners.toAlert">
		D组件：调用app组件（根）的方法
	</button>
	...
</template>
```
实际上这就是将$listeners对象不断传递给子组件的过程。

---
provide&inject 演示代码

App.vue
```
<template>
	 <button @click="onClick2">
	 	点击按钮，App组件向D组件（子孙或后代）provide传值
	 </button>
</template>
<script>
	export default{
		...
		data() {
			return {
			  val: "默认值",
			  val1: "",
			  val2: "默认值",
			};
		},
		provide() {
			this.val = Vue.observable({ 
				Val: "provide中注册一个响应值"
			});
			this.val1 = Object.defineProperty({}, "Val", {
			  value: "初始值",
			  writable: true,
			  configurable: true,
			  enumerable: false
			});
			this.data = new Proxy(this.$data, {
			  get: (target, name) => {
				return name in target ? target[name] : "undefined";
			  }
			});
			return {
			  val: this.val,
			  val1: this.val1,
			  data: this.data
			};
		},
		methods:{
			...
			onClick2() {
			  alert("修改provide中的val值");
			  this.val.Val = "默认值=>修改provide中的val值";
			  this.val1.Val = "修改provide的val1值";
			  this.data.val2 = "修改provide的val2值";
			}
			...
		}
	...
	}
</script>
```
App组件中在provide提供了2中注册响应值的方式，vue.observable在vue2中是对defineProperty的封装，在vue3中是对Proxy的封装，这样注册的变量在点击按钮后，修改后的新的值可以同步到子孙组件中。


D.vue
```
<template>
	...
	<h3>D组件：我的父亲是B组件</h3>
    <p>准备接受根组件inject值:{{val.Val}}</p>
    <p>通过defineproperty，inject传递的值：{{val1.Val}}</p>
    <p>通过proxy,inject传递的值：{{data.val2}}</p>
	...
</template>
<script>
	export default{
		...
		 inject:['val','val1','data'],
		...
	}
</script>
```
inject接受祖辈组件传过来的值，语法上和props类型，存在数组和对象的方式，具体的差异可以参考官方文档。

## 非简单关系的组件
因为非简单关系，想想脑壳就疼，所以提供了傻瓜式的**总线模式**，但为了更好的管理总线上的事件，如果组件关系并非很复杂，请使用上述介绍的方案，本文并不会对组件通信的优缺点进行说明，萝卜白菜各有所爱，但各自方法需要注意的点，参见vue官方文档API。

现在已有E组件、A组件的子孙D组件，假设E组件存在F组件(子)。那么F组件和D组件的传值就是一种非简单关系。以下代码存在省略。

D.vue
```
<template>
	...
	<p>准备接受F组件（复杂关系）的值：{{localVal}}</p>
	...
</template>
<script>
	...
	data(){
        return {
            localVal:""
        }
    },
    mounted() {
        this.$root.$on('ftod',(e)=>{
            this.localVal = e;
        })
    },
</script>
```
$root就可以理解为根组件实例，这里$root就是所谓的总线，$on监听事件，等待$emit触发。


F.vue
```
<template>
 <div style="border:1px solid #787878;">
     <h2>F组件：C、D组件与我的关系难以描述</h2>
     <button @click="onClick" style="margin:10px;">
		点击按钮，向D组件传值
	 </button>
 </div>
</template>
<script>
export default {
    name:"F",
    data(){
        return {
            mi:'F组件data中的值'
        }
    },
    methods:{
        onClick(){
            this.$root.$emit('ftod',this.mi)
        }
    }
}
</script>

```
通过点击事件发送emit事件，传递mi值，这是最简单的总线使用方式。
Vue本身已经实现了$on and $emit 方法，另外还存在这样一种实现总线的方式。
```
export new Vue()
```
下面通过代码演示基本原理

**$on、$emit基本原理**

Bus.js
```
class Bus{
	constructor(){
		this.callbacks = {}
	}
	$on(name,fn){
		this.callbacks[name] = this.callbacks[name] || []
		this.callbacks[name].push(fn)
	}
	$emit(name,args){
		if(this.callbacks[name]){
			this.callbacks[name].forEach(cd=>cb(args))
		}
	}
}

export new Bus()
```
为了方便调用，挂在到Vue实例上！


main.js
```
import Vue from 'vue'
import bus from './Bus'
Vue.prototype.$bus = bus;

```

##总结
以上说明了绝大多数组件通信的情况，算是Vue知识的普及，提供给大家补充基础知识。为了方便实践，这里提供一份[demo]()，如果在练习过程中存在代码问题，欢迎交流。微信：**cch_lover** 唯一的联系方式。
