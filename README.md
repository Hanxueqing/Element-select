## Element-UI中Select选择器详解

### 前言

最近开发的后台管理系统项目采用Vue+Element-UI技术架构，在使用Elment-UI中Select组件的时候遇到了比较多的操作难题，官网上关于这个组件的使用文档介绍的不是很详细，仅仅提供了一些基本用法，很多拓展场景都没有涉及到，在查阅了大量资料之后终于将目前的需求都完美解决了，这里整理一些Select组件的使用方案，希望能帮到有同样需求的同学。



项目已上传github，欢迎大家下载交流。

前端项目地址：https://github.com/Hanxueqing/Element-select



### 项目运行

```
# 克隆到本地
git clone git@github.com:Hanxueqing/Element-select.git

# 安装依赖
npm install

# 开启本地服务器localhost:8080
yarn serve

# 发布环境
yarn build
```



### 使用场景

#### 局部引入Select组件

注意这里引入Select组件的时候还需要引入Option，不然下拉列表渲染不出来，他们是两个单独的组件。

```
import { Select, Option} from 'element-ui'
Vue.use(Select)
Vue.use(Option)
```



#### 使用下拉菜单展示数据项

`v-model`的值为当前被选中的`el-option`的 value 属性值，输入框中显示的为label属性值。

```
            <el-select v-model="value" placeholder="请选择">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
                :disabled="item.disabled">
              </el-option>
          </el-select>
```

我们将要显示在下拉菜单中的数据写在options数组中，给label属性赋值姓名，让输入框显示我们选中的姓名。

```
<script>
  export default {
    data() {
      return {
        options: [{
          value: '选项1',
          company: '腾讯',
          label: '马化腾',
          school: '深圳大学'

        }, {
          value: '选项2',
          company: '美团',
          label: '王兴',
          school: '清华大学'

        }, {
          value: '选项3',
          company: '小米',
          label: '雷军',
          school: '武汉大学'

        }, {
          value: '选项4',
          company: '今日头条',
          label: '张一鸣',
          school: '南开大学'

        }, {
          value: '选项5',
          company: '红杉资本',
          label: '沈南鹏',
          school: '耶鲁大学'

        }],
        value: ''
      }
    }
  }
</script>
```

效果演示：

![](http://ww1.sinaimg.cn/large/006tNc79ly1g61catu9wcg30lo0kcnl5.gif)



#### 为select设定默认值

将默认值赋给v-model绑定的数据即可，此处我们给v-model绑定的是value，所以在data中给value赋值为"马化腾"为默认值。

```
value: '马化腾'
```



#### 在输入框按下回车，选择第一个匹配项

在`el-select`标签中添加`default-first-option`属性，需配合 `filterable` 或 `remote` 使用。



#### 在下拉菜单中，展示多项数据

目前我们在options数组中为每一个数据定义了三个字段，分别是公司、姓名和毕业院校，但是下拉列表中只能显示姓名，所以我们要自定义HTML模板，插入`el-option`中的slot。

```
<el-option
  v-for="item in options"
  :key="item.value"
  :label="item.label"
  :value="item.value"
  :disabled="item.disabled">
  <span style="float: left">{{ item.company }}</span>
  <span style="float: left">{{ item.label }}</span>
  <span style="float: left">{{ item.school }}</span>
</el-option>
```

效果演示：

![](http://ww4.sinaimg.cn/large/006tNc79ly1g61cq06j32g30ss0kc7wh.gif)



#### 给el-select添加唯一class名

我们修改了列表项的样式，但是页面中可能会使用多个select组件，直接修改会影响全局样式，在页面渲染的时候，el-select总是会被渲染为仅次于body层级的div，我们没有办法通过父级限制它，所以我们需要给它自身添加一个class名。

```
  .el-select-dropdown__item span{
    width:100px;
    text-align:center;
  }
```

在el-select上按照常规方法添加`class = "optionsContent"`，不生效，给el-select加父标签`<div class = "optionsContainter">`来包裹el-select也没用。

```html
            <div class = "optionsContainter">
              <el-select v-model="value" placeholder="请选择" class = "optionsContent">
                <el-option
                  v-for="item in options"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                  :disabled="item.disabled"
                >
                  <span style="float: left">{{ item.company }}</span>
                  <span style="float: left">{{ item.label }}</span>
                  <span style="float: left">{{ item.school }}</span>
                </el-option>
            </el-select>
            </div>
```

在控制台中查看渲染结果可以看到，class名和div根本没有被渲染出来。

![](http://ww3.sinaimg.cn/large/006tNc79ly1g61cx134xuj30qy0f8jvp.jpg)



这里需要将class改为`propper-class`

```
<el-select v-model="value" placeholder="请选择" popper-class = "optionsContent">
```

通过**popper-class**来自定义一个类，这样子在控制台可以看到，自定义的类渲染到页面上的效果，class名已经添加成功了：

![](http://ww2.sinaimg.cn/large/006tNc79ly1g61me4loccj30qs0bq41y.jpg)





#### 根据输入条件模糊查询

有的时候我们需要根据用户在input框中输入的内容提供对应的列表项。这时候需要为`el-select`添加`filterable`属性即可启用搜索功能。



效果演示：

![](http://ww1.sinaimg.cn/large/006tNc79ly1g61njpnhjfg30ss0kcqfe.gif)





当我们想实现输入拼音首字母检索的时候，就需要使用filter-method给它传入一个函数方法。

> 默认情况下，Select 会找出所有`label`属性包含输入值的选项。如果希望使用其他的搜索逻辑，可以通过传入一个`filter-method`来实现。`filter-method`为一个`Function`，它会在输入值发生变化时调用，参数为当前输入值。



给options中的每一个数据项添加szm字段，赋值为姓名拼音首字母

```
{
          value: '选项1',
          company: '腾讯',
          label: '马化腾',
          school: '深圳大学',
          szm:'mht'

        }
```



el-select标签中给filter-method绑定filter方法

```
<el-select v-model="value" placeholder="请选择" popper-class = "optionsContent" filterable :filter-method="filter">
```



在method中编写filter方法

```
filter(v) {
        // 对绑定数据赋值
        this.options = this.copy.filter((item) => {
          // 如果直接包含输入值直接返回true
          const val = v.toLowerCase()
          if (item.label.indexOf(val) !== -1) return true
          if (item.szm.substring(0, 1).indexOf(val) !== -1) return true
          if (item.szm.indexOf(val) !== -1) return true
        })
      }
```



在mounted中给copy数据赋值，保留数据源

```
mounted () {
      // 保留数据源
      this.copy = Object.assign(this.options)
    }
```



效果演示：

![](http://ww1.sinaimg.cn/large/006tNc79ly1g61npuqfjlg30ss0kcqgp.gif)





#### 在Select 组件头部插入内容

当你需要在select的input框中插入内容时就需要用到组件提供的slot具名插槽，name值为：prefix。

```
              <template slot = "prefix">
                <span class = 'prefixSlot'>互联网大佬</span>
              </template>
```



![](http://ww1.sinaimg.cn/large/006tNc79ly1g61o7cn52cj30m007saaf.jpg)





#### 在Option 下拉列表插入内容

如果我们需要在Option下拉列表中插入内容，官方也给我们提供了slot匿名插槽。比如，我们在数据项之前添加一个表头：

```
          <template>
            <div class = "optionHeader" v-show = 'optionVisible'>
              <span style="float: left">姓名</span>
              <span style="float: left;">卡号</span>
              <span style="float: left;">手机号</span>
            </div>
          </template>
```



![](http://ww2.sinaimg.cn/large/006tNc79ly1g61qzne789j30t60juacv.jpg)



#### 外部组件获得下拉列表选中项数据

我们需要将下拉列表中选中的数据展示在外部页面中，在`el-option`标签中给value值绑定`item.value`，那么v-model里的value将会得到选中对象的value，如果赋值为item，那将会得到选中的对象，可以在change事件里将返回的值分别赋值给需要的变量。

我们给`el-select`的chage事件绑定showMessage方法，参数是$event

```
<el-select v-model="value" placeholder="请选择" popper-class = "optionsContent" filterable :filter-method="filter" @change="showMessage($event)">
```

我们先打印一下获得的参数，可以获取到选中的对象

![](http://ww2.sinaimg.cn/large/006tNc79ly1g61rmg7w9tj30pk0bmwgl.jpg)



在methods中定义showMessage方法，给对应的数据赋值

```
      showMessage(e){
        console.log(e)
        this.v_company = e.company;
        this.v_label = e.label;
        this.v_school = e.school
      }
```



在页面中展示我们获得的数据

```
          <div class = "title">
            <span>公司：{{v_company}}</span>
            <span>姓名：{{v_label}}</span>
            <span>毕业院校：{{v_school}}</span>
          </div>
```



效果演示：

![](http://ww1.sinaimg.cn/large/006tNc79ly1g61roqeaamg30wc0q2kjl.gif)





#### 当输入框为空的时候不显示下拉列表，输入数据的时候根据输入内容显示对应列表项

现在默认是输入框获取焦点的时候就显示全部列表项，但是当我们涉及到一些保密功能时，不希望显示全部的列表项，比如选择会员，我们不能一开就将会员信息全部展示出来，所以一开始需要在data中设置一个数据`optionVisible:false`，控制下拉列表的显示与隐藏。

下拉列表的默认样式也需要做一些修改，去掉小尖头、边框、最小宽度和盒子阴影。

```
  .popper__arrow{
    display: none!important;
  }
  .el-select-dropdown{
    box-shadow: none!important;
    min-width: 0px;
    border:0!important;
  }
```



给`el-select`和`el-option`添加v-show指令`v-show = 'optionVisible'`，在el-select标签上绑定`@keyup.native = "showOption"`方法，编写showOption方法，判断输入框中的数据长度不为0时，显示下拉列表。

```
showOption(){
        let inputContent = document.getElementsByClassName('el-input__inner')[0].value
        console.log(inputContent.length)
        if(inputContent.length != 0){
          this.optionVisible = true;
        }
      }
```



效果演示：

![](http://ww4.sinaimg.cn/large/006tNc79ly1g61shlp81ug30wc0q2n4e.gif)