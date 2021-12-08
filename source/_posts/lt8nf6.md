---
title: 横向滚动通知栏
urlname: lt8nf6
date: '2021-08-06 16:13:45 +0800'
tags: []
categories: []
---

### Dom 结构

```javascript

<template lang="pug">
.notice-board-wrapper(v-show="show")
  .left(ref="animateParent")
    .animate(ref="animate") {{content}}
  .right(@click="handleClose")
    i.el-icon-close
    | ×
</template>

<template >
  <div class="notice-board-wrapper" v-if="show">
    <div class="left" ref="animateParent">
      <div class="animate" ref="animate">  {{ content }}</div>
    </div>
    <div class="right" @click="handleClose" ><a-icon type="close" /></div>
  </div>
</template>
```

## JavaScript

```javascript
<script>
export default {
  props: {
    show: {
      type: Boolean,
      default: false
    },
    content: {
      type: String,
      default: ''
    }
  },
  mounted () {
    // 监听窗口变化，优化滚动栏，debounce防止多次触发
    window.addEventListener(
      'resize',
      this.debounce(() => this.calcAnimateWidth(), 2000)
    )
    this.calcAnimateWidth()
  },
  methods: {
    calcAnimateWidth () {
      this.$nextTick(() => {
        if (!this.$refs.animateParent) return
        // 计算keyframes偏移距离，使得滚动字段可以衔接
        const animateParentWidth = this.$refs.animateParent.clientWidth
        const animateWidth = this.$refs.animate.clientWidth
        const itemsLength = Math.ceil(animateParentWidth / animateWidth)
        // 将样式加入样式表
        document.styleSheets[0].insertRule(
          `@keyframes wordsLoop { 0% { transform: translateX(100%); -webkit-transform: translateX(100%); } 100% { transform: translateX(-${itemsLength}00%); -webkit-transform: translateX(-${itemsLength}00%);}}`,
          0
        )
        // 若insertRule报错，使用以下方法，并在index.html中添加： <style id="banner"></style>
        // let test = document.getElementById('banner').sheet
        // test = document.styleSheets[0]
        // test.insertRule(
        //   `@keyframes wordsLoop { 0% { transform: translateX(100%); -webkit-transform: translateX(100%); } 100% { transform: translateX(-${itemsLength}00%); -webkit-transform: translateX(-${itemsLength}00%);}}`,
        //   0,
        // )
      })
    },
    handleClose () {
      this.$emit('close')
    },
    debounce (fn, wait) {
      var timer = null
      return function () {
        if (timer !== null) {
          clearTimeout(timer)
        }
        timer = setTimeout(fn, wait)
      }
    }
  }
}
</script>
```

## Less

```javascript
<style lang="less" scoped>
.notice-board-wrapper {
  // position: absolute;
  // top: 58px;
  // left: 0;
  background: #34a3c9;
  color: #fff;
  width: 100%;
  height: 30px;
  line-height: 30px;
  padding: 0 20px;
  display: flex;
  z-index: 2;
  .left {
    flex: 1;
    overflow: hidden;
    text-align: right;
    .animate {
      display: inline-block;
      white-space: nowrap;
      // hover暂停动画
      &:hover {
        animation-play-state: paused;
      }
      @media screen and (max-width: 960px) {
        animation: 14s wordsLoop linear infinite normal;
      }
      @media screen and (min-width: 961px) and (max-width: 1200px) {
        animation: 16s wordsLoop linear infinite normal;
      }
      @media screen and (min-width: 1201px) {
        animation: 18s wordsLoop linear infinite normal;
      }
    }
  }
  .right {
    width: 30px;
    text-align: center;
    font-size: 20px;
    cursor: pointer;
  }
}
</style>
```