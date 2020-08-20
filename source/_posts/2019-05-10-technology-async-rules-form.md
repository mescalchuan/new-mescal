---
layout: post
title: IVIEW 动态校验规则组件
date: 2019-05-10 09:23:19 +0300
description: 
catalog: true
header-img: /img/16.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - vue
categories: 技术博客
ascription: technology
author: # Add name author (optional)
---

### async-rules-form
动态配置 iview 的表单规则

#### 初衷
实现 `iview` 的`Form` 组件可以动态修改校验规则的功能。

#### 需求场景
例如：当用户年龄下拉框选择“未成年”时，身份证号为非必填项，否则为必填项。

#### 原理
为该字段生成两份 不同规则的 `FormItem`，通过`v-if`切换。目前仅支持两个校验规则。

#### API
属性 | 说明 | 类型 | 默认值
----|-----|------|------
prop | 对应表单域 `model` 里的字段，同 `iview.FormItem` | String | -
label | 标签文本，同 `iview.FormItem` | String | -
label-width | 表单域标签的的宽度钮，同 `iview.FormItem` | Number | -
label-for | 指定原生的 `label` 标签的 `for` 属性，配合控件的 `element-id` 属性，可以点击 `label 时聚焦控件`，同 `iview.FormItem` | String | -
rules | 表单验证规则，结构为 `{ rules1: FormItem.rules, rules2: FormItem.rules }` 。其中，`FormItem.rules` 额外添加了 `condition` 字段，作为规则的生成条件，该字段返回 `Boolean`，必填 | Object | -
show-message | 是否显示校验错误信息，同 `iview.FormItem` | Boolean | `true`

#### 示例
```
<template>
    <Form>
        <FormItem label="用户年龄" prop="isUnderAge" :rules="ageRules">
            <Select v-model="form.isUnderAge">
                <Option value="0" key="0">未成年</Option>
                <Option value="1" key="1">成年</Option>
            </Select>
        </FormItem>
        <async-rules-form label="身份证号" prop="IDNumber" :rules="{
                rules1: {
                    type: 'string',
                    required: false,
                    message: '未成年可以不输入身份证号',
                    condition: form.isUnderAge == 0
                },
                rules2: {
                    type: 'string',
                    required: true,
                    message: '成年人必须输入身份证号',
                    condition: form.isUnderAge == 1
                },
            }"
        >
            <Input v-model="form.IDNumber" slot="form-element"/>
        </async-rules-form>
    </Form>
</template>
<script>
    import asyncRulesForm from './async-rules-form'
    export default {
        ...
        components: {
            asyncRulesForm
        }
        ...
    }
</script>

```
