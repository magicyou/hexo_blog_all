---
title: vuex使用方式
date: 2018-05-11 22:13:02
updated: 2018-05-11 22:13:04
tags: [Vue]
categories: [前端]
---
vuex的五个属性 state, getters, mutations, actions, modules
<!--more-->

主要参考来源：

* https://vuex.vuejs.org/zh/

* https://blog.csdn.net/wh710107079/article/details/88181015

#### vuex是什么？

vuex是一个专门为Vue.js应用开发的状态管理扩展（官方文档叫状态管理模式）

官方建议：vuex适合大型单页应用，如果你的项目足够小，不建议使用，小项目使用vuex可能是繁琐冗余的。



#### 使用方法

vuex怎么在vue脚手架里面使用呢？

1. src目录下创建store，store下创建文件store.js

2. store.js内容如下（demo）：

   ```javascript
   import Vue from 'vue';
   import Vuex from 'vuex';
   Vue.use(Vuex)
   ```

3. 然后就是五个属性的使用方法

    state, getters, mutations, actions, modules
##### State

定义了应用状态的数据结构，可以在这里设置默认的初始状态。

```
/**
 state即Vuex中的基本数据！
 */
var state = {
	count: 0,
	token: '21212',
	userId: '还没有',
}
```

组件内使用

```
export default {
  computed: {
        tokenUser: function(){
            return this.$store.state.token
        }
   }
 }
```



##### Getter

state的派生状态，（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。。

```
/*
即从store的state中派生出的状态。
getters接收state作为其第一个参数，接受其他 getters 作为第二个参数，如不需要，第二个参数可以省略如下例子：
*/
var getters = {
	// 单个参数
	countTokenUser: function(state){
		return state.token + state.userId
	},
	// 两个参数
	countDoubleAndDouble: function(state, getters) {
		return getters.countDouble * 2
	}
}
```
组件内使用

```
export default {
  computed: {
        tokenUser: function(){
            return this.$store.getters.countTokenUser
        }
   }
 }
```


##### Mutation

是唯一更改 store 中状态的方法，且必须是同步函数。

```
/*
提交mutation是更改Vuex中的store中的状态的唯一方法。
mutation必须是同步的，如果要异步需要使用action。
每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数，提交载荷作为第二个参数。（提交荷载在大多数情况下应该是一个对象）,提交荷载也可以省略的。
*/
var mutations = {
	login(state, userInfo) {
		state.token = userInfo.token
		state.userId = userInfo.userId
	},
	jian(a) {
		a.count--
	}
}
```

```
  methods: {
    login(){
				this.$store.commit('login', {token: '333333', userId: 'lxlxlxl'})
      },
   }
```

##### Action

用于提交 mutation，而不是直接变更状态，可以包含任意异步操作。

```
/**
 actions
Action 类似于 mutation，不同在于：
Action 提交的是 mutation，而不是直接变更状态。
Action 可以包含任意异步操作
 */
var actions = {
    increment (context) {
      setInterval(function(){
        context.commit('increment')
      }, 1000)
	}
}
```
感觉官方例子好理解：

+++++++++++来自官方
让我们来注册一个简单的 action：

```
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```
Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。当我们在之后介绍到 Modules 时，你就知道 context 对象为什么不是 store 实例本身了。

实践中，我们会经常用到 ES2015 的 参数解构 来简化代码（特别是我们需要调用 commit 很多次的时候）：
```
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```
分发 Action
Action 通过 store.dispatch 方法触发：

store.dispatch('increment')
乍一眼看上去感觉多此一举，我们直接分发 mutation 岂不更方便？实际上并非如此，还记得 mutation 必须同步执行这个限制么？Action 就不受约束！我们可以在 action 内部执行异步操作：

```
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
Actions 支持同样的载荷方式和对象方式进行分发：

// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})


```

+++++++++++


##### Module

允许将单一的 Store 拆分为多个 store 且同时保存在单一的状态树中。

```
/**
 Modules
使用单一状态树，导致应用的所有状态集中到一个很大的对象。但是，当应用变得很大时，store 对象会变得臃肿不堪。
为了解决以上问题，Vuex 允许我们将 store 分割到模块（module）。每个模块拥有自己的 state、mutation、action、getters、甚至是嵌套子模块——从上至下进行类似的分割：
 */
const moduleA = {
	state: {
		name: 'lxl',
		age:23
	},
	mutation: {
		setUserInfo(state, userInfo) {
			state.name = userInfo.name
			state.age = userInfo.age
		}
	}
}
const moduleB = {
	state: {
		name: 'ljz',
		age: 22
	},
	mutation: {
		setUserInfo2(state, userInfo) {
			state.name = userInfo.name
			state.age = userInfo.age
		}
	}
}
```



#### 当然还需要使用这五个属性

```
export default new Vuex.Store({
	state,
	mutations,
	getters,
	actions,
	modules: {
		a: moduleA,
		b: moduleB,
	}
})
```

#### 最后别忘了要在main.js添加：

```
import store from './store/store.js'
```




