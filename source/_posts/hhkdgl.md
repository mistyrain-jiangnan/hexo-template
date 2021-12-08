---
title: 打字机效果组件 。
urlname: hhkdgl
date: '2021-10-23 12:05:05 +0800'
tags: []
categories: []
---

```javascript
<template>
  <div>
    <slot></slot>
  </div>
</template>
<script>
export default {
  name: 'VueTextDot',
  props: {
    interval: { type: Number, default: 75 }
  },
  methods: {
    typewriter () {
      var $ele = this.$el
      var str = $ele.innerHTML

      var progress = 0
      $ele.innerHTML = ''
      const interval = this.interval
      var timer = setInterval(function () {
        var current = str.substr(progress, 1)
        if (current === '<') {
          progress = str.indexOf('>', progress) + 1
        } else {
          progress++
        }
        $ele.innerHTML = str.substring(0, progress) + (progress < str.length && (progress & 1) ? '_' : '')
        if (progress >= str.length) {
          clearInterval(timer)
        }
      }, interval)
    }
  }
}
</script>

```

## 使用

```javascript
<TypeWriter class="tl" ref="typewriter" :interval="200" :style="{display: status}">
      <div class="comments">
        <p><span class="space"></span>欢迎来到应用系统缺陷报修管理系统</p>
      </div>
    </TypeWriter>
 data () {
    return {
      // 表头
      columns,
      // 示例功能列表
      data,
      chooseProjectShow: true,
      status: 'none'
    }
  },
      mounted () {
    this.type()
  },
```