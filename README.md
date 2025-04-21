# cnvuekeyboard

一个基于 Vue3 和 simple-keyboard 的中文/英文虚拟键盘组件，支持拼音输入、大小写切换、符号输入、分页候选词等功能，适合触屏和桌面场景。

## 特性

- 支持中英文输入、拼音联想
- 支持大小写切换（Shift）
- 支持符号输入
- 候选词分页显示
- 触屏友好，样式美观

## 安装

```bash
npm install simple-keyboard cnchar cnchar-input cnchar-words cnchar-poly cnchar-idiom


## 安装

```bash
<template>
  <cnvuekeyboard
    :input="input"
    @onChange="handleInput"
    @onLangChange="handleLangChange"
  />
</template>

<script setup>
import cnvuekeyboard from './src/components/SimpleKeyboard.vue'
import { ref } from 'vue'
const input = ref('')
function handleInput(val) {
  input.value = val
}
function handleLangChange(lang) {
  // 语言切换回调
}
</script>
