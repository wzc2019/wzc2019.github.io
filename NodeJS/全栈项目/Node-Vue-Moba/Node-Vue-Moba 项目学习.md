

 # Node-Vue-Moba 项目学习

[toc]

## 一、项目来源

本项目学习资料来自 B 站、全站之巅的教学视频：TODO

该学习项目已经在 GitHub 开源，有开源的地址：TODO

本篇是个人学习复现该项目的所做的一份笔记。

需要注意的一点养成，完成了一部分之后主动提交一次到 Git 的习惯。



学习方法：

- 下载原版

## 二、环境搭建

### 2.1 相关准备 

相关的环境以及使用的模块：

- NodeJS ：一个 JS 的服务端运行环境

  - axios : 一个基本的 HTTP 库
  - express：TODO
  - mongoose：TODO
  - cros：TODO

- Vue ：一个前端框架
  - Vue-CLI：Vue 的快速启动模板、脚手架
  - Vue-Router ：Vue 路由管理
  - Element ：前端 UI 界面
- MongoDB ：一个非关系型数据库
- Git ： 项目管理工具

### 2.2 项目初始化

需要在项目下面创建三个文件夹，实现三端：

- admin：后台管理界面
- web：前台用户界面
- server：后端服务器

#### 2.2.1 server 子项目

1. 通过 `npm init -y` 去创建一个 `Node.js` 项目
2. 创建一个 `index.js` 作为程序的入口
3. 在 `package.json` 中添加一个脚本 `"serve":"nodemon index.js",` 用于后台启动一个守护进程
4. 安装 `nodemon`

#### 2.2.2 admin 子项目

1. 通过 `vue create admin `  去创建一个 `Vue-CLI` 项目

  2. 通过 `npm run serve` 可以启动服务
  3. 在该项目下使用  `vue add element`  添加一个 `element` 组件
  4. 同理，添加 `Vue-Router`

#### 2.2.3 web 子项目

​	同理，使用  `vue create web` 先创建项目

#### 2.2.4 其他文件

- gitigonore：git 需要忽略掉的项目的文件
- LICENSE：相关的一些协议

- readme：项目指南

## 三、分类模块

在之前的项目中，我们通过项目初始化，构造了下面的一个目录结构：

<img src="Node-Vue-Moba 项目学习.assets/image-20200119173635074.png" alt="image-20200119173635074" style="zoom:67%;" />

### 3.1 创建分类

#### 3.1.1 使用模板代码

1. 在 `src/views/` 目录下创建一个 `Main.vue` ,直接使用下面的 Element 一个示例代码

<img src="Node-Vue-Moba 项目学习.assets/image-20200119174153170.png" alt="image-20200119174153170" style="zoom:67%;" />



2. 修改边框栏

```js
<el-aside width="200px" style="background-color: rgb(238, 241, 246)">
    <el-menu router :default-openeds="['1', '3']">
      <el-submenu index="1">
        <template slot="title"><i class="el-icon-message"></i>分类管理</template>
        <el-menu-item-group>
          <template slot="title">分类</template>
          <el-menu-item index="/categories/create">创建分类</el-menu-item>
          <el-menu-item index="/categories/list">分类列表</el-menu-item>
        </el-menu-item-group>
      </el-submenu>
    </el-menu>
  </el-aside>
```

3. 在主体中，添加一个路由容器

```js
<el-main>
<router-view></router-view>
</el-main>
```



#### 3.1.2 添加路由

在 `src/router/index.js` 文件中,修改成如下的代码

修改相关的路由,引入 Main 组件，并添加一个创建目录的子路由

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Main from '../views/Main.vue'
import CategoryEdit from '../views/CategoryEdit.vue'
Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'main',
    component: Main,
    children:[
      {path:'/categories/create',component:CategoryEdit}
    ]
  },
]
```

#### 3.1.3  创建菜单界面

在 `/src/views `目录下，添加一个 `CategoryEdit.vue` 文件，然后

1. 添加如下 UI 界面：

```js
<template>
    <div class="about">
        <h1>新建分类</h1>
        <el-form label-width="100px">
        <el-form-item label="名称">
        <el-input v-model="model.name"></el-input>
        </el-form-item>
        <el-form-item>
            <el-button type="primary" native-type="submit">保存</el-button>
        </el-form-item>
        </el-form>
    </div>
</template>
```

2. 添加 data、method 
   - data 中的 model：用于双向绑定表单数据
   - method 中的 save：用于表单 post 提交

```js
<script>
export default {
    data(){
        return {
            model:{}
        }
    },
    methods: {
        save(){
            this.$http.post()
        }
    },
}
</script>
```

   

#### 3.1.4 服务端代码

##### 3.1.4.1 程序入口

1.  编写根目录下的 `index.js`

```js
const express = require('express')
const app = express()

app.use(require('cors')())
app.use(express.json())

require('./routes/admin')(app)
require('./plugins/db')(app)
app.listen(3000,()=>{
    console.log('http://localhost:3000');
})
```

引入各种需要的模块，启动 `express `、`cors`、`express.json` 中间件以及 路由和数据库组件。

然后监听 3000 端口。

##### 3.1.4.2 路由模块

在 `routes\admin` 目录下创建一个 `index.js` 文件

```js
module.exports = app=>{
    const express = require('express')
    const router = express.Router()
    const Category = require('../../models/Category')
    router.post('/categories',async(req,res)=>{
        const model = await Category.create(req.body)
        res.send(model)
    })
    app.use('/admin/api',router)
}
```

取得 `router` 、`Category` 模型，等待 model 对象，然后回显。



##### 3.1.4.3 模型 Category

在 `\models` 目录下，创建一个 `Category.js` 的模型文件，

一个映射到 Mongo 数据库的 ORM 对象：

```js
const mongoose = require('mongoose')

const schema = new mongoose.Schema({
    name : {type:String}
})

module.exports = mongoose.model('Category',schema)
```

##### 3.1.4.4 数据库模块

在 `\plugins` 目录下，创建一个 `db.js` 的数据库连接文件：

连接到 mongodb。

```js
module.exports = app =>{
    const mongoose = require("mongoose")
    mongoose.connect('mongodb://127.0.0.1:27017/node-vue-moba',{
        useNewUrlParser:true
    })
}
```

### 3.2 分类列表

#### 3.2.1 页面编写

在这个页面中包含一个表单，然后包含 items 数组数据。

在 Vue 创建之后，通过 fetch 方法获取到数据。

```js
<template>
  <div class="about">
    <h1>分类列表</h1>
    <el-table :data="items" style="width: 100%">
      <el-table-column prop="_id" label="ID" width="260"></el-table-column>
      <el-table-column prop="name" label="分类名称" width="180"></el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
    data(){
        return {
            items:[]
        }
    },
    methods: {
        async fetch(){
            const res = await this.$http.get('categories');
            this.items = res.data
        }
    },
    created() {
        this.fetch()
    },

}
</script>
```

#### 3.2.2 添加路由

在 `src/router` 添加 ` {path:'/categories/list',component:CategoryList}` 子路由。

#### 3.2.3 服务端代码

在服务端 `route/admin`  的 `index.js` 添加一条监听 get 的路由。

```js
router.get('/categories',async (req,res)=>{
    const items = await Category.find().limit(10)
    res.send(items)
})
```



### 3.3 分类编辑

编辑较为简单，就简要说明

1. 在 List 页面下添加一个编辑按钮

```js
<el-button type="text" size="small" @click = "$router.push(`/categories/edit/${scope.row._id}`)">编辑</el-button>
```

2. 在 Edit 页面中添加一个 prop 属性

   ```js
   props: {
       id: {}
     },
   ```

3. 添加相关的 if 判断：

   ```js
    // let res
           if(this.id){
               await this.$http.put(`categories/${this.id}`, this.model);
               //res =await this.$http.put("categories", this.model);
           } else{ 
               await this.$http.post("categories", this.model);
               //res =await this.$http.post("categories", this.model);
           }
   ```

4. 在服务端，添加两条路由：

   ```js
       router.get('/categories/:id',async (req,res)=>{
           const item = await Category.findById(req.params.id)
           res.send(item)
       })
       router.put('/categories/:id',async (req,res)=>{
           const model = await Category.findByIdAndUpdate(req.params.id,req.body)
           res.send(model)
       })
   ```

   

### 3.4 分类删除

1. 添加一个删除按钮，指向remove 函数

   ```js
   <el-button type="text" size="small" @click="remove(scope.row)">删除</el-button>
   ```

2. 编写一个 remove 函数：

   ```js
       async remove(row) {
         this.$confirm(`是否确定要删除分类 "${row.name}" ?`, "提示", {
           confirmButtonText: "确定",
           cancelButtonText: "取消",
           type: "warning"
         }).then(async () => {
           await this.$http.delete(`categories/${row._id}`);
           this.$message({
             type: "success",
             message: "删除成功!"
           });
           this.fetch()
         });
       }
   ```

3. 编写后端代码，添加 

   ```js
       router.delete('/categories/:id',async (req,res)=>{
           const model = await Category.findByIdAndDelete(req.params.id,req.body)
           res.send({
               success:true
           })
       })
   ```

### 3.5 子分类

假设我们现在需要多级分类的功能，就是一个分类下面有子分类。

#### 3.5.1 编辑页面

增加一个下拉菜单，选择已有的分类模型，作为上级分类，使用 v-model 进行数据双向绑定。

同时使用 v-for 进行列表循环：

- ：key 是一个给 Vue 用的索引值
- ：label 选项标签名
- ：value 真实绑定的值

```js
<el-form-item label="上级分类">
        <el-select v-model="model.parent">
            <el-option v-for="item in parents" :key="item._id"
            :label="item.name" :value="item._id"></el-option>
        </el-select>
      </el-form-item>
```

添加相关函数获取菜单列表：

```js
    async fetchparent(){
        const res = await this.$http.get(`categories`)
        this.parents = res.data
    }
```

同时添加 parents 数组：

```js
data() {
    return {
      model: {},
      parents:[]
    };
  },
```

在 created 中添加，获取上级目录的函数调用：

```
 created() {
      this.fetchparent()
      this.id && this.fetch()
  },
```

#### 3.5.2 Category 模型修改 

添加一个自身的引用：

```js
const mongoose = require('mongoose')

const schema = new mongoose.Schema({
    name : {type:String},
    parent : {type:mongoose.SchemaTypes.ObjectId,ref:'Category'},
})

module.exports = mongoose.model('Category',schema)
```

#### 3.5.3 修改服务端

加入 populate 去获得 parent 对象而不是 ID :

```js
    router.get('/categories',async (req,res)=>{
        const items = await Category.find().populate('parent').limit(10)
        res.send(items)s
    })
```

#### 3.5.4 修改列表页面

加入一列显示上级分类的名称：

```js
<el-table-column prop="parent.name" label="上级分类"></el-table-column>
```

### 3.6 通过 CRUD 接口

1. 修改相关的后端接口加上 rest 前缀

2. 添加中间件，动态获取资源对象取到不同的模型

   ```js
    app.use('/admin/api/rest/:resource',async (req,res,next) =>{
           const modelName = require('inflection').classify(req.params.resource)
           req.Model = require(`../../models/${modelName}`)
           next()
       },router)
   ```

3. 对一些内容，特殊化处理

   ```js
   const queryOptions = {}
   if (req.Model.modelName === 'Category'){
       queryOptions.populate = 'parent'
   }
   const items = await req.Model.find().setOptions(queryOptions).limit(10)
   ```

### 3.x 问题及解决

出现了一个 sass-loader 出错的问题：

<img src="Node-Vue-Moba 项目学习.assets/image-20200120000313611.png" alt="image-20200120000313611" style="zoom:67%;" />

直接百度，得到下面的答案

[web前端_Vue框架_关于Can't resolve 'sass-loader' in...错误的解决办法](https://blog.csdn.net/qq_24058693/article/details/80056557)

##  四 、装备模块

因为已经用上了通过 CRUD 接口，所以其他界面的配置会很简单。

模仿分类的写法，很容易写出装备的编辑和列表界面，并添加一个 `item.js`  模型即可。

#### 4.1 图片上传：编辑界面

下面重点是图片上传：

1. 在编辑界面中添加一个：

   - 设置 :action 当提交的时候提交到 baseURL/upload 的地址
   - 设置 onsuccess ，在上传成功后调用 afterUpLoad 函数

   ```js
   <el-form-item label="图标">
       <el-upload
       class="avatar-uploader"
       :action="$http.defaults.baseURL +'/upload'"
       :show-file-list="false"
       :on-success="afterUpLoad"
       >
       <img v-if="model.icon" :src="model.icon" class="avatar" />
       <i v-else class="el-icon-plus avatar-uploader-icon"></i>
       </el-upload>
   </el-form-item>
   ```

   afterUpLoad 函数：设置图标的 url 地址，使其上传后可以显示

   ```js
       afterUpLoad(res){
         this.$set(this.model,'icon',res.url)
       }
   ```

#### 4.2 图片上传：后端接口

使用 `multer` 模块。

使用静态托管托管 `/uploads` 下的静态资源：

```js
app.use('/uploads',express.static(__dirname + '/uploads'))
```

监听 `upload`  请求，当文件发送过来后回应文件信息并添加一个静态的 URL 路径。

```js
    const multer =require('multer')
    const upload = multer({dest:__dirname + '/../../uploads'})
    app.post('/admin/api/upload',upload.single('file'),async (req,res)=>{
        const file =req.file
        file.url = `http://localhost:3000/uploads/${file.filename}`
        res.send(file) 
    })
```

#### 4.3 图片上传：列表界面

在列表菜单中，添加显示列图标，这里使用模板自定义：

```js
<el-table-column prop="icon" label="图标" width="180">
        <template slot-scope="scope">
          <img :src="scope.row.icon" style="height:3rem;" alt="">
        </template>
</el-table-column>
```

#### 4.4 效果展示

1. 编辑界面

<img src="Node-Vue-Moba 项目学习.assets/image-20200121123402746.png" alt="image-20200121123402746" style="zoom:67%;" />

2. 物品列表界面

<img src="Node-Vue-Moba 项目学习.assets/image-20200121123441077.png" alt="image-20200121123441077" style="zoom:50%;" />

## 五、英雄模块

如装备所示，相应先添加一个相仿的界面。

### 5.1 英雄模型

根据官网攻略站，对应的信息，

<img src="Node-Vue-Moba 项目学习.assets/image-20200121204602691.png" alt="image-20200121204602691" style="zoom:50%;" />

撰写相关数据库模型，建议边写边测试，

```js
const mongoose = require('mongoose')

const schema = new mongoose.Schema({
    name : {type:String},
    icon : {type:String},
    title :{type:String},
    categories :[{type:mongoose.SchemaTypes.ObjectId,ref:'Category'}],
    scores :{
        difficult:{type:Number},
        skills:{type:Number},
        attack:{type:Number},
        survive:{type:Number},

    },
    skills:[{
        icon:{type:String},
        name:{type:String},
        description:{type:String},
        tips:{type:String},


    }],
    items1:[{
        type: mongoose.SchemaTypes.ObjectId,ref:'Item'
    }],
    items2:[{
        type: mongoose.SchemaTypes.ObjectId,ref:'Item'
    }],
    usageTips:{type:String},
    battleTips:{type:String},
    teamTips:{type:String},
    partners:[{
        hero:{type:mongoose.SchemaTypes.ObjectId,ref:'Hero'},
        description:{type:String},
    }]
})

module.exports = mongoose.model('Hero',schema)
```

### 5.2 英雄编辑界面

#### 5.2.1 基本属性面板

编写一块基本信息面板

```js
 <el-tab-pane label="基础信息">
            <el-form-item label="名称">
              <el-input v-model="model.name"></el-input>
            </el-form-item>

            <el-form-item label="头像">
              <el-upload
                class="avatar-uploader"
                :action="$http.defaults.baseURL +'/upload'"
                :show-file-list="false"
                :on-success="afterUpLoad"
              >
                <img v-if="model.icon" :src="model.icon" class="avatar" />
                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
              </el-upload>
            </el-form-item>

            <el-form-item label="称号">
              <el-input v-model="model.title"></el-input>
            </el-form-item>
            <el-form-item label="类型">
              <el-select v-model="model.categories" multiple>
                <el-option v-for="item of categories" :key="item._id" :label="item.name" :value="item._id">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="难度">
              <el-rate style="margin-top:0.6rem" :max="10" show-score v-model="model.scores.difficult"></el-rate>
            </el-form-item>
            <el-form-item label="技能">
              <el-rate style="margin-top:0.6rem" :max="10" show-score v-model="model.scores.skills"></el-rate>
            </el-form-item>
            <el-form-item label="攻击">
              <el-rate style="margin-top:0.6rem" :max="10" show-score v-model="model.scores.attack"></el-rate>
            </el-form-item>
            <el-form-item label="生存">
              <el-rate style="margin-top:0.6rem" :max="10" show-score v-model="model.scores.survive"></el-rate>
            </el-form-item>
            
            <el-form-item label="顺风出装">
              <el-select v-model="model.items1" multiple>
                <el-option v-for="item of items" :key="item._id" :label="item.name" :value="item._id">
                </el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="逆风出装">
              <el-select v-model="model.items2" multiple>
                <el-option v-for="item of items" :key="item._id" :label="item.name" :value="item._id">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="使用技巧">
              <el-input type="textarea" v-model="model.usageTips"></el-input>
            </el-form-item >
            <el-form-item label="战斗技巧">
              <el-input type="textarea" v-model="model.battleTips"></el-input>
            </el-form-item >
            <el-form-item label="团战技巧">
              <el-input type="textarea" v-model="model.teamTips"></el-input>
            </el-form-item >
          </el-tab-pane>
```

#### 5.2.2 技能面板

还有一份技能面板

```js
<el-tab-pane label="技能" name="skills">
            <el-button size="small" @click="model.skills.push({})"> <i class="el-icon-plus"></i> 添加技能</el-button>
            <el-row type="flex" style="flex-wrap:wrap">
              <el-col :md="12" v-for="(item,i) in model.skills" :key="i">
                <el-form-item label="名称" >
                  <el-input v-model="item.name"></el-input>
                </el-form-item>
                <el-form-item label="图标" v-model="item.icon">
                  <el-upload
                    class="avatar-uploader"
                    :action="$http.defaults.baseURL +'/upload'"
                    :show-file-list="false"
                    :on-success="res => $set(item,'icon',res.url)"
                  >
                    <img v-if="item.icon" :src="item.icon" class="avatar" />
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                  </el-upload>
                </el-form-item>
                <el-form-item label="描述" >
                  <el-input v-model="item.description" type="textarea"></el-input>
                </el-form-item>
                <el-form-item label="小提示" >
                  <el-input v-model="item.tips" type="textarea"></el-input>
                </el-form-item>
                <el-form-item>
                  <el-button size="small" type="danger" @click="model.skills.splice(i,1)">
                   删除</el-button>
                </el-form-item>
              </el-col>
            </el-row>
          </el-tab-pane>
```

### 5.3 效果展示

1. 基本面板

   <img src="Node-Vue-Moba 项目学习.assets/image-20200121223319093.png" alt="image-20200121223319093" style="zoom:33%;" />

2. 技能面板

   <img src="Node-Vue-Moba 项目学习.assets/image-20200121223402130.png" alt="image-20200121223402130" style="zoom:50%;" />

## 六、文章模块

文章包含分类、标题、详情主体等。

其中详情主体需要用到富文本编辑器来充分展示文章细节。

### 6.1 富文本编辑器 

这里使用了 `Vue2-editor`  第三方组件，使用`useCustomImageHandler @image-added="handleImageAdded"` 进行定义。

`<vue-editor useCustomImageHandler @image-added="handleImageAdded" v-model="model.body"></vue-editor>`

改变函数的默认提交图片的行为，

```js
async handleImageAdded(file, Editor, cursorLocation, resetUploader) {
      // An example of using FormData
      // NOTE: Your key could be different such as:
      // formData.append('file', file)

      const formData = new FormData();
      formData.append("file", file);
      const res = await this.$http.post('upload',formData);
      Editor.insertEmbed(cursorLocation, "image", res.data.url);
          resetUploader();
      
    }
```



## 七、广告位模块

广告位包含一个名称，下面可能有多条广告轮播。

广告位模块也相对比较简单，添加一个如以前英雄模块所示的结构

```js
 <el-button size="small" @click="model.items.push({})"> <i class="el-icon-plus"></i> 添加广告</el-button>
            <el-row type="flex" style="flex-wrap:wrap">
              <el-col :md="12" v-for="(item,i) in model.items" :key="i">
                <el-form-item label="跳转链接" >
                  <el-input v-model="item.url"></el-input>
                </el-form-item>
                <el-form-item label="图片" v-model="item.image">
                  <el-upload
                    class="avatar-uploader"
                    :action="$http.defaults.baseURL +'/upload'"
                    :show-file-list="false"
                    :on-success="res => $set(item,'image',res.url)"
                  >
                    <img v-if="item.image" :src="item.image" class="avatar" />
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                  </el-upload>
                </el-form-item>
                <el-form-item>
                  <el-button size="small" type="danger" @click="model.items.splice(i,1)">
                   删除</el-button>
                </el-form-item>
              </el-col>
            </el-row>
```



## 八、管理员及登录

### 8.1 管理员模型

建立一个管理员模型，里面包含用户名和密码

```js
const mongoose = require('mongoose')

const schema = new mongoose.Schema({
    name : {type:String},
    password : {type:String,
        select:false,
        set(val){
        return require('bcrypt').hashSync(val,10)
    }},
})

module.exports = mongoose.model('AdminUser',schema)
```
而然管理界面不会和其他增删改查模块近似，这里就不多赘述。

### 8.2 登录接口

获取前端页面传送过来的用户数据，
1. 根据用户名找用户
2. 校验密码
3. 返回 token
token 是一个用来确认验证的一个标志。
```js
app.post('/admin/api/login',async (req,res)=>{
        const {name,password} =req.body
        // 1. 根据用户名找用户
        const AdminUser = require('../../models/AdminUser')
        const user = await AdminUser.findOne({name}).select('+password');
        if (!user){
            return res.status(422).send({
                message:'用户不存在'
            })
        }
        // 2. 校验密码
        const isVaild = require('bcrypt').compareSync(password,user.password)
        if(!isVaild){
            return res.status(422).send({
                message:'密码错误'
            })
        }
        // 3. 返回 token
        const jwt =require('jsonwebtoken')
        const token = jwt.sign({
            id:user._id,
        },app.get('secret'))
        res.send({token})
    })
```

### 8.3 登录界面
实现一个如图的登录界面显示，
<img src="Node-Vue-Moba 项目学习.assets/image-20200122183235803.png" alt="image-20200122183235803" style="zoom:50%;" />

```js
<template>
    <div class="login-container">
        <el-card header="请先登录" class="login-card">
            <el-form @submit.native.prevent="login">
                <el-form-item label="用户名">
                    <el-input v-model="model.name"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input v-model="model.password"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" native-type="submit">登录</el-button>
                </el-form-item>
            </el-form>
            
        </el-card>
    </div>
</template>
```

下面是一个登录处理函数，登录后得到一个token 保存在 localStorage 中，由浏览器记录。
```js
async login(){
            const res =await this.$http.post('login',this.model)
            localStorage.token = res.data.token
            this.$router.push('/')
            this.$message({
                type:'success',
                message:'登录成功'
            })
        }
```

### 8.4 服务端校验

