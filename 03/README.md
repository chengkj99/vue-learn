# 实战

一般情况下，开始一个新项目之前会进行一次技术选型，根据团队属性和场景选择相应的技术:

* 框架 ( vue, react, angular )
* 开发语言 ( ES6, typescript )
* 打包工具（ webpack, rollup ）
* 组件库 ( elementUI, iview )
* 图表库 （ echarts, highcharts ）
* api请求方案 ( fetch, axios, ajax )
* css预编译 ( sass, scss, less )
* 状态管理 ( vuex, mobx )
* 样式方案/代码风格 ( airbnb, standard )
* 路由方案 ( vue-router )

有了 vue-cli, 以上这些基本都不用考虑了。

### 工程模板

现有的工程模板，辅助我们完成了搭建项目构建环境的流程，使用它们可以让我们直接专注于业务开发。

* [vuejs-templates](https://github.com/vuejs-templates)
* [vue-webpack-boilerplate-introduction](http://vuejs-templates.github.io/webpack/env.html)
* [vueAdmin-template](https://github.com/PanJiaChen/vueAdmin-template)

### 编译工具

Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 用于对模块的源代码进行转换。

所以我们需要各种 loader, 各种插件...

插件用于扩展 webpack 的功能, 可以完成更多 loader 不能完成的功能。如：CommonsChunkPlugin 创建一个公用的chunk，常用于将第三方 lib 抽取成公用js

* [webpack](https://github.com/webpack/webpack)
* [vue-loader](https://vue-loader.vuejs.org/#what-is-vue-loader)
* [babel-loader](https://github.com/babel/babel-loader)
* [sass-loader](https://github.com/webpack-contrib/sass-loader)
* ...

### vue 技术栈

* [vue-router](https://router.vuejs.org/)
* [vuex](https://vuex.vuejs.org/zh/guide/)
* [axios](https://github.com/axios/axios)
* [elementUI](http://element-cn.eleme.io/)

### 其他工具推荐

* [lodash](https://lodash.com/docs/4.17.10)
* [js-cookie](https://github.com/js-cookie/js-cookie)
* [NProgress](https://github.com/rstacruz/nprogress)
* [moment](https://github.com/moment/moment)
* ...

### 开发工具

vue-devtools是一款基于chrome游览器的插件，用于调试vue应用，这可以极大地提高我们的调试效率。

* [vue-devtools](https://github.com/vuejs/vue-devtools)

### 规范

* [vuejs-component-style-guide](https://github.com/pablohpsilva/vuejs-component-style-guide)

## 开始

### 安装

vue-cli生成的项目需要继续安装一些依赖和库。vueAdmin-template 大多数的模块都已经装好了。

```
npm install vuex -S

npm install element-ui -S

npm install axios -S

...
```

### 项目结构

vue-cli 生成的项目视情况补充： `api/`、`constants/`、`utils/`、`transforms/`、`components/layout`

vueAdmin-template 视情况补充 `constants/`、`transforms/`

可以通过 vue-devtools 概览 `vueAdmin-template` 的项目结构

### 实战：产品管理模块

开发一个功能模块的需要完成： 路由配置、 api接口配置、store 状态管理(可选)、 入口组件(逐步完善)、 功能组件(逐步完善)

> 逐步完善： 先搭个功能架子，再一一补充功能细节。

####

#### 添加路由配置 `router/index.js`

左侧导航是根据路由来生成的，
```
{
  path: '/product',
  component: Layout,
  children: [
    {
      path: '',
      name: 'Product',
      component: () => import('@/views/product/index'),
      meta: { title: 'Product', icon: 'table' }
    }
  ]
},
```


#### api `api/product.js`

根据接口文档，实现前端的接口调用函数。

```
import axios from 'axios'

export function getProductList(query) {
  return axios.get(`/api/product/list?query=${query}`)
}

export function addProduct(value) {
  return axios.post(`/api/product/add`, {
    contacts: '王宗义',
    phone: '13651226099',
    mail: 'Wangzongyi001@sina.com',
    imgUrl: 'empty url',
    owner: 'Wangzongyi',
    status: 'disable',
    ...value
  })
}

export function changeStatus(id, status) {
  return axios.put(`/api/product/status/${id}`, { status })
}

export function deleteProduct(id) {
  return axios.delete(`/api/product/${id}`)
}

```

配置代理转发，跨域 `config/index.js`

```
proxyTable: {
      '/api': {
        target: 'http://192.168.1.8:1323',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
```


#### 功能入口组件 `product/index.vue`

```
<template>
  <div class="product">
    <div class="filter-content">
      <el-row type="flex" class="row-bg" justify="space-between">
        <el-col :span="18">
          <el-button size="small" type="primary" plain @click="addProduct">新增产品</el-button>
        </el-col>
        <el-col :span="6">
          <el-input
            size="small" v-model="query" placeholder="请输入查询的名称"
            @keyup.enter.native="queryProduct"
            suffix-icon="el-icon-search">
          </el-input>
        </el-col>
      </el-row>
    </div>

    <product-list
      :value="productList"
      @change-status="handleChangeStatus"
      @delete="handleDelete"
      v-loading="isLoadingList">
    </product-list>
    <el-dialog
      title="新增产品"
      :visible.sync="dialogVisible"
      width="50%">
      <product-form
        ref="productForm"
        :is-loading="isLoadingAddProduct"
        @submit="handleSubmit"
        @cancel="handleCancel">
      </product-form>
    </el-dialog>
  </div>
</template>

<script>
import { getProductList, changeStatus, deleteProduct, addProduct } from '@/api/product'
import ProductForm from './form'

import ProductList from './list'
export default {
  name: 'product',
  components: {
    ProductList,
    ProductForm
  },
  data() {
    return {
      productList: [],
      query: '',
      dialogVisible: false,
      isLoadingList: false,
      isLoadingAddProduct: false
    }
  },
  methods: {
    fetData(query = '') {
      this.isLoadingList = true
      getProductList(query).then(
        res => {
          this.productList = res.data.data
          this.isLoadingList = false
        }
      )
    },
    handleChangeStatus(id, status) {
      changeStatus(id, status).then(
        () => {
          this.$message.success('改变状态成功')
          this.fetData()
        },
        () => {
          this.$message.fail('操作失败')
        }
      )
    },
    handleDelete(id) {
      deleteProduct(id).then(
        () => {
          this.$message.success('删除成功')
          this.fetData()
        },
        () => { this.$message.fail('删除失败') }
      )
    },
    addProduct() {
      this.dialogVisible = true
    },
    handleSubmit(form) {
      this.isLoadingAddProduct = true
      addProduct(form).then(
        () => {
          this.$message.success('添加产品成功')
          this.fetData()
          this.dialogVisible = false
          this.isLoadingAddProduct = false
        },
        () => {
          this.$message.fail('添加失败')
        }
      )
    },
    handleCancel() {
      this.dialogVisible = false
      this.$refs.productForm.$refs.form.resetFields()
    },
    queryProduct() {
      this.fetData(this.query)
    }
  },
  created() {
    this.fetData()
  }
}
</script>

<style lang='scss'>
.product {
  display: block;
  position: relative;
  padding: 10px;
  .filter-content {
    position: relative;
    margin: 10px 0 20px 0;
  }
}
</style>


```


#### table 列表组件

```
<template>
  <div class="product-list">
  <el-table
    :data="value"
    style="width: 100%">
    <el-table-column
      label="ID"
      width="180">
      <template slot-scope="scope">
        <span>{{ scope.row.id }}</span>
      </template>
    </el-table-column>
    <el-table-column
      label="产品名称"
      width="180">
      <template slot-scope="scope">
        <span>{{ scope.row.name }}</span>
      </template>
    </el-table-column>
    <el-table-column
      label="厂商"
      width="180">
      <template slot-scope="scope">
        <span>{{ scope.row.firm_model }}</span>
      </template>
    </el-table-column>
    <el-table-column
      label="参数"
      width="180">
      <template slot-scope="scope">
        <span>{{ scope.row.parameter }}</span>
      </template>
    </el-table-column>
    <el-table-column
      label="状态"
      width="180">
      <template slot-scope="scope">
        <el-tag :type="humanizeStatus(scope.row.status).type">
          {{ humanizeStatus(scope.row.status).value }}
        </el-tag>
      </template>
    </el-table-column>
    <el-table-column label="操作">
      <template slot-scope="scope">
        <el-dropdown size="mini" split-button type="primary" @command="handleCommand">
          改变状态
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item :command="`${scope.row.id}|enable`">启用</el-dropdown-item>
            <el-dropdown-item :command="`${scope.row.id}|disable`">禁用</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <el-button
          size="mini"
          type="danger"
          @click="handleDelete(scope.row.id)">删除</el-button>
      </template>
    </el-table-column>
  </el-table>
  </div>
</template>

<script>

export default {
  name: 'product-list',
  props: ['value'],
  methods: {
    humanizeStatus(status) {
      if (status === 'enable') {
        return { type: 'success', value: '可用' }
      }
      if (status === 'disable') {
        return { type: 'danget', value: '不可用' }
      }
      return { type: 'info', value: '-' }
    },
    handleCommand(command) {
      const [id, status] = command.split('|')
      this.$emit('change-status', id, status)
    },
    handleDelete(id) {
      this.$confirm('此操作将永久删除该产品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$emit('delete', id)
      })
    }
  }
}
</script>

<style lang='scss'>
.product-list {
  display: block;
  position: relative;
}
</style>

```

#### form 表单组件

```
<template>
  <div class="product-form">
    <el-form ref="form" :model="form" :rules="rules" label-width="80px" size="small">
      <el-form-item label="名称" prop="name">
        <el-input v-model="form.name" placeholder="请输入名称"></el-input>
      </el-form-item>
      <el-form-item label="原价" prop="original_price">
        <el-input v-model="form.original_price" placeholder="请输入产品原价"></el-input>
      </el-form-item>
      <el-form-item label="厂商型号" prop="firm_model">
        <el-input v-model="form.firm_model" placeholder="请输入厂商型号"></el-input>
      </el-form-item>
      <el-form-item label="参数" prop="parameter">
        <el-input v-model="form.parameter" placeholder="请输入参数"></el-input>
      </el-form-item>
      <el-form-item label="功能" prop="functional_use">
        <el-input v-model="form.functional_use" placeholder="请输入功能"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="desc">
        <el-input v-model="form.desc" placeholder="请输入描述"></el-input>
      </el-form-item>
      <el-form-item>
        <div class="btn-wrapper">
          <el-button @click="cancel">取消</el-button>
          <el-button type="primary" @click="onSubmit" :loading="isLoading">立即创建</el-button>
        </div>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
const initialForm = {
  name: '',
  original_price: '',
  firm_model: '',
  parameter: '',
  functional_use: '',
  desc: ''
}

export default {
  name: 'product-form',
  props: ['isLoading'],
  data() {
    return {
      form: { ...initialForm },
      rules: {
        name: [
          { type: 'string', required: true, message: '请输入名称' }
        ],
        original_price: [
          { type: 'string', required: true, message: '请输入产品原价' }
        ],
        firm_model: [
          { type: 'string', required: true, message: '请输入厂商型号' }
        ],
        parameter: [
          { type: 'string', required: true, message: '请输入参数' }
        ],
        functional_use: [
          { type: 'string', required: true, message: '请输入功能' }
        ],
        desc: [
          { type: 'string', required: true, message: '请输入描述' }
        ]
      }
    }
  },
  methods: {
    onSubmit() {
      this.$refs.form.validate(valid => {
        if (!valid) return
        this.$emit('submit', this.form)
      })
    },
    cancel() {
      this.$emit('cancel')
    }
  }
}
</script>

<style lang='scss'>
.product-form {
  position: relative;
  display: block;
  .btn-wrapper {
    text-align: right;
  }
}
</style>

```

### 了解： vuex

#### 添加路由配置 `router/index.js`

```
{
  path: '/vuex',
  component: Layout,
  children: [
    {
      path: '',
      name: 'vuex',
      component: () => import('@/views/vuex-demo/index'),
      meta: { title: 'VuexDemo', icon: 'user' }
    }
  ]
},
```

#### 实现 vuex-demo 模块的 store `store/`

全局 store 配置 `store/index.js`
```
import Vue from 'vue'
import Vuex from 'vuex'
import app from './modules/app'
import user from './modules/user'
import vuexDemo from './modules/vuex-demo'
import getters from './getters'

Vue.use(Vuex)

const store = new Vuex.Store({
  modules: {
    app,
    user,
    vuexDemo
  },
  getters
})

export default store
```
实现 vuex-demo 的store `store/modules/vuex-demo.js`
```
const vuexDemo = {
  state: {
    todoList: [
      { text: 'Learn Vue' },
      { text: 'Learn Javascript' },
      { text: 'Learn PHP' },
      { text: 'Learn Golang' }
    ]
  },
  mutations: {
    updateTodoList: (state, todoList) => {
      state.todoList = todoList
    }
  },
  actions: {
    updateTodoList: ({ commit }, todoList) => {
      commit('updateTodoList', todoList)
    }
  }
}
export default vuexDemo

```

#### 创建 vuex-demo 入口文件 `vuex-demo/index.vue`

```
<template>
  <div class="vuex-demo">
    <el-input @keyup.enter.native="addTodo" v-model="value"></el-input>
    <ul>
      <li v-for="(todo, index) in todoList" :key="index">
        <span>{{ todo }}</span>
        <el-button size="mini" @click="remove(index)">remove</el-button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'vuex-demo',
  data() {
    return {
      todoList: this.$store.state.vuexDemo.todoList,
      value: ''
    }
  },
  methods: {
    remove(index) {
      this.todoList.splice(index, 1)
      this.$store.dispatch('updateTodoList', this.todoList)
    },
    addTodo() {
      this.todoList.push({ text: this.value })
      this.$store.dispatch('updateTodoList', this.todoList)
    }
  }
}
</script>

<style lang='scss'>
.vuex-demo {
  position: relative;
  display: block;
}
</style>

```

