---
title: vue---组件封装中之slot插槽使用
date: 2019-08-03 21:33:00
updated: 2019-08-03 21:33:00
tags: ['vue','JavaScript']
categories: ['前端']
---

引用一下vue官方的说法 https://cn.vuejs.org/v2/guide/components-slots.html

```
在 2.6.0 中，我们为具名插槽和作用域插槽引入了一个新的统一的语法 (即 v-slot 指令)。它取代了 slot 和 slot-scope 这两个目前已被废弃但未被移除且仍在文档中的 attribute。新语法的由来可查阅这份 RFC。
```

通俗的说就是插槽可以用来在组件封装的时候放自定义的任何内容


#### 基本使用

* 子组件 aimm-panel

```html
<template>
	<div class="wrapper-panel">
		<p class="txt-header">
			<span>{{ title }}</span>
			<span class="header-right">
				<slot></slot>
			</span>
		</p>
	</div>
</template>
```

* 父组件里面用法

```html
<aimm-panel title="Position">
                <template v-slot>
                    <div class="box-item-info">
                        <div class="item-position-info">
                            <p class="txt-title">LATITUDE</p>
                            <p class="txt-value">8° 0.499' N</p>
                        </div>
                        <div class="item-position-info">
                            <p class="txt-title">LONGITUDE</p>
                            <p class="txt-value">8° 0.499' N</p>
                        </div>
                    </div>
                </template>
            </aimm-panel>
```



#### 有name的插槽

有时候我们需要在组件内多个指定地方放不同内容，可以加上name属性

* 子组件 aimm-panel

```html
<template>
	<div class="wrapper-panel">
		<p class="txt-header">
			<span>{{ title }}</span>
			<span class="header-right">
				<slot name="headerRight"></slot>
			</span>
		</p>
		<div>
			<slot name="body"></slot>
		</div>
	</div>
</template>
```

* 父组件用法

```html
   <aimm-panel title="Position">
                <template v-slot:body>
                    <div class="box-item-info">
                        <div class="item-position-info">
                            <p class="txt-title">LATITUDE</p>
                            <p class="txt-value">8° 0.499' N</p>
                        </div>
                        <div class="item-position-info">
                            <p class="txt-title">LONGITUDE</p>
                            <p class="txt-value">8° 0.499' N</p>
                        </div>
                    </div>
                </template>
            </aimm-panel>
```

或许官方组件更清晰

```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

