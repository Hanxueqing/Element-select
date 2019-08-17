<template>
  <el-container class="wrapper">
    <el-header
      height="80px"
      :style="{ 'background-color': primaryColor }">
      <img
        src="./assets/element-logo.svg"
        alt="element-logo"
        class="header-logo">
      <ul class="header-operations">
        <li @click="showThemeDialog">{{ langConfig.header.switch[lang] }}</li>
        <li
          class="header-download"
          :class="{ 'is-available': canDownload }"
          @click="downloadZip">
          {{ langConfig.header.download[lang] }}
        </li>
        <li @click="showHelpDialog">{{ langConfig.header.help[lang] }}</li>
        <li>
          <span
            @click="switchLang('/zh-CN')"
            :class="{ 'is-active': lang === '/zh-CN' }"
            class="header-lang">
            中文
          </span>
          <span>/</span>
          <span
            @click="switchLang('/en-US')"
            :class="{ 'is-active': lang === '/en-US' }"
            class="header-lang">
            En
          </span>
        </li>
      </ul>
    </el-header>
    <el-container>
      <el-aside class="menu">
        <el-menu default-active="1">
          <el-menu-item index="1">{{ langConfig.menu.pageOne[lang] }}</el-menu-item>
          <el-menu-item index="2">{{ langConfig.menu.pageTwo[lang] }}</el-menu-item>
          <el-menu-item index="3">{{ langConfig.menu.pageThree[lang] }}</el-menu-item>
        </el-menu>
      </el-aside>
      <el-main class="content">
        <el-breadcrumb>
          <el-breadcrumb-item>{{ langConfig.breadcrumb.main[lang] }}</el-breadcrumb-item>
          <el-breadcrumb-item>{{ langConfig.breadcrumb.project[lang] }}</el-breadcrumb-item>
          <el-breadcrumb-item>{{ langConfig.breadcrumb.page[lang] }}</el-breadcrumb-item>
        </el-breadcrumb>
        <el-form inline :model="query" label-position="right" label-width="60px" class="query-form">
          <div class = "title">
            <span>公司：{{v_company}}</span>
            <span>姓名：{{v_label}}</span>
            <span>毕业院校：{{v_school}}</span>
          </div>
          <el-form-item :label="langConfig.query.name[lang]" prop="name">
            <el-select v-model="value" placeholder="请选择" popper-class = "optionsContent" filterable :filter-method="filter" @change="showMessage($event)" @keyup.native = "showOption" default-first-option>
              <template slot = "prefix">
                <span class = 'prefixSlot'>互联网大佬</span>
              </template>
              <template>
                <div class = "tableHeader" v-show = 'optionVisible'>
                  <span style="float: left">公司</span>
                  <span style="float: left;">姓名</span>
                  <span style="float: left;">毕业院校</span>
                </div>
              </template>
              <el-option
                v-show = 'optionVisible'
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item"
                :disabled="item.disabled"
              >
                <span style="float: left">{{ item.company }}</span>
                <span style="float: left">{{ item.label }}</span>
                <span style="float: left">{{ item.school }}</span>
              </el-option>
          </el-select>
          </el-form-item>
          <el-form-item :label="langConfig.query.date[lang]" prop="date">
            <el-date-picker
              v-model="query.date"
              type="daterange"
              :placeholder="langConfig.query.dateHolder[lang]">
            </el-date-picker>
          </el-form-item>
          <el-form-item>
            <el-button type="primary">{{ langConfig.query.search[lang] }}</el-button>
          </el-form-item>
        </el-form>
          <el-table
           ref="multipleTable"
            :data="tableData"
            tooltip-effect="selection-changedark"
            @selection-change="handleSelectionChange"
            highlight-current-row
            :row-class-name="rowStyle"
          >
            <el-table-column
              type="selection"
              width="55">
            </el-table-column>
            <el-table-column
              label="日期"
              width="120">
              <template slot-scope="scope">{{ scope.row.date }}</template>
            </el-table-column>
            <el-table-column
              prop="name"
              label="姓名"
              width="120">
            </el-table-column>
            <el-table-column
              prop="address"
              label="地址"
              show-overflow-tooltip>
            </el-table-column>
          </el-table>
          <div style="margin-top: 20px">
            <el-button @click="toggleSelection([tableData[1], tableData[2]])">切换第二、第三行的选中状态</el-button>
            <el-button @click="toggleSelection()">取消选择</el-button>
          </div>
      </el-main>
    </el-container>
    <el-dialog
      center
      :visible.sync="themeDialogVisible"
      :title="langConfig.header.switch[lang]"
      width="400px">
      <el-form
        :model="colors"
        :rules="rules"
        ref="form"
        class="theme-form"
        label-position="top"
        label-width="70px">
        <el-form-item :label="langConfig.form.theme[lang]" prop="primary">
          <el-color-picker v-model="colors.primary" ></el-color-picker>
        </el-form-item>
        <el-form-item class="color-buttons">
          <el-button type="primary" @click="submitForm">{{ langConfig.form.switch[lang] }}</el-button>
          <el-button @click="resetForm">{{ langConfig.form.reset[lang] }}</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <el-dialog :visible.sync="helpDialogVisible" :title="langConfig.help.title[lang]">
      <div v-html="langConfig.help.content[lang]" class="help"></div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="helpDialogVisible = false">{{ langConfig.help.ok[lang] }}</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>

<script>
  import generateColors from './utils/color'
  import objectAssign from 'object-assign'
  import blobUtil from 'blob-util'
  import langConfig from './lang'
  import tableData from './table-data'
  import JSZip from 'jszip'
  import FileSaver from 'file-saver'
  import { use } from 'element-ui/lib/locale'
  import zhLocale from 'element-ui/lib/locale/lang/zh-CN'
  import enLocale from 'element-ui/lib/locale/lang/en'

  export default {
    name: 'app',

    data () {
      const colorValidator = (rule, value, callback) => {
        if (!value) {
          return callback(new Error(this.langConfig.validate.required[this.lang]))
        } else if (!/^#[\dabcdef]{6}$/i.test(value)) {
          return callback(new Error(this.langConfig.validate.format[this.lang]))
        } else {
          callback()
        }
      }
      return {
        optionVisible:false,
        v_company:"",
        v_label:"",
        v_school:"",
        options: [{
          value: '选项1',
          company: '腾讯',
          label: '马化腾',
          school: '深圳大学',
          szm:'mht'

        }, {
          value: '选项2',
          company: '美团',
          label: '王兴',
          school: '清华大学',
          szm:'wx'

        }, {
          value: '选项3',
          company: '小米',
          label: '雷军',
          school: '武汉大学',
          szm:'lj'

        }, {
          value: '选项4',
          company: '今日头条',
          label: '张一鸣',
          school: '南开大学',
          szm:'zym'

        }, {
          value: '选项5',
          company: '红杉资本',
          label: '沈南鹏',
          school: '耶鲁大学',
          szm:'snp'

        }],
        value: '马化腾',
        tableData: [{
          id:0,
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          id:1,
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          id:2,
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          id:3,
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          id:4,
          date: '2016-05-08',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          id:5,
          date: '2016-05-06',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          id:6,
          date: '2016-05-07',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }],
        multipleSelection: [],
        colors: {
          primary: '#409eff'
        },
        rules: {
          primary: [
            { validator: colorValidator, trigger: 'blur' }
          ]
        },
        originalStylesheetCount: -1,
        originalStyle: '',
        langConfig,
        primaryColor: '#409eff',
        themeDialogVisible: false,
        helpDialogVisible: false,
        zip: null,
        styleFiles: [],
        fontFiles: ['element-icons.ttf', 'element-icons.woff'],
        fonts: [],
        canDownload: false,
        query: {
          name: '',
          date: []
        }
      }
    },

    computed: {
      lang () {
        const lang = this.$route.path
        use(lang === '/zh-CN' ? zhLocale : enLocale)
        this.query.date = []
        return lang
      }
    },

    methods: {
      switchLang (lang) {
        this.$router.push(lang)
      },

      showThemeDialog () {
        this.themeDialogVisible = true
      },

      showHelpDialog () {
        this.helpDialogVisible = true
      },

      resetZip () {
        this.zip = new JSZip()
        this.fonts.forEach((font, index) => {
          // console.log(font, index)
          this.zip.file(`fonts/${this.fontFiles[index]}`, font.data)
        })
      },

      writeNewStyle () {
        let cssText = this.originalStyle
        Object.keys(this.colors).forEach(key => {
          cssText = cssText.replace(new RegExp('(:|\\s+)' + key, 'g'), '$1' + this.colors[key])
        })

        if (this.originalStylesheetCount === document.styleSheets.length) {
          const style = document.createElement('style')
          style.innerText = cssText
          document.head.appendChild(style)
        } else {
          document.head.lastChild.innerText = cssText
        }
      },

      submitForm () {
        this.$refs.form.validate(valid => {
          if (valid) {
            this.themeDialogVisible = false
            this.primaryColor = this.colors.primary
            this.colors = objectAssign({}, this.colors, generateColors(this.colors.primary))

            this.canDownload = true
            this.writeNewStyle()
          } else {
            return false
          }
        })
      },

      downloadZip () {
        if (!this.canDownload) return
        this.resetZip()
        this.styleFiles.forEach(file => {
          let cssText = file.data
          Object.keys(this.colors).forEach(key => {
            cssText = cssText.replace(new RegExp('(:|\\s+)' + key, 'g'), '$1' + this.colors[key])
          })
          const css = blobUtil.createBlob([cssText], { type: 'text/css' })
          this.zip.file(file.name, css)
        })
        this.zip.generateAsync({ type: 'blob' })
          .then(blob => {
            FileSaver.saveAs(blob, `element-${this.primaryColor}.zip`)
          })
      },

      resetForm () {
        this.$refs.form.resetFields()
      },

      getStyleTemplate (data) {
        const colorMap = {
          '#3a8ee6': 'shade-1',
          '#409eff': 'primary',
          '#53a8ff': 'light-1',
          '#66b1ff': 'light-2',
          '#79bbff': 'light-3',
          '#8cc5ff': 'light-4',
          '#a0cfff': 'light-5',
          '#b3d8ff': 'light-6',
          '#c6e2ff': 'light-7',
          '#d9ecff': 'light-8',
          '#ecf5ff': 'light-9'
        }
        Object.keys(colorMap).forEach(key => {
          const value = colorMap[key]
          data = data.replace(new RegExp(key, 'ig'), value)
        })
        return data
      },

      getFile (url, isBlob = false) {
        return new Promise((resolve, reject) => {
          const client = new XMLHttpRequest()
          client.responseType = isBlob ? 'blob' : ''
          client.onreadystatechange = () => {
            if (client.readyState !== 4) {
              return
            }
            if (client.status === 200) {
              const urlArr = client.responseURL.split('/')
              resolve({
                data: client.response,
                url: urlArr[urlArr.length - 1]
              })
            } else {
              reject(new Error(client.statusText))
            }
          }
          client.open('GET', url)
          client.send()
        })
      },

      getIndexStyle () {
        this.getFile('//unpkg.com/element-ui/lib/theme-chalk/index.css')
          .then(({ data }) => {
            this.originalStyle = this.getStyleTemplate(data)
          })
      },

      getSeparatedStyles () {
        this.getFile('//unpkg.com/element-ui/lib/theme-chalk/')
          .then(({ data }) => {
            return data.match(/href="[\w-]+\.css"/g).map(str => str.split('"')[1])
          })
          .then(styleFiles => {
            return Promise.all(styleFiles.map(file => {
              return this.getFile(`//unpkg.com/element-ui/lib/theme-chalk/${file}`)
            }))
          })
          .then(files => {
            this.styleFiles = files.map(file => {
              return {
                name: file.url,
                data: this.getStyleTemplate(file.data)
              }
            })
          })
      },

      getFontFiles () {
        Promise.all(this.fontFiles.map(font => {
          return this.getFile(`//unpkg.com/element-ui/lib/theme-chalk/fonts/${font}`, true)
        }))
          .then(fonts => {
            this.fonts = fonts
          })
      },
      handleSelectionChange(val) {
       var arr = [];
       val.forEach((val, index) => {
　　　　　　this.tableData.forEach((v, i) => {
　　　　　　　// id 是每一行的数据id
            if(val.id == v.id){
              arr.push(i)
            }
          })
        })   
        this.multipleSelection = arr;
      },
      rowStyle({row, rowIndex}){
        let arr =  this.multipleSelection;
        // console.log(arr)
        for(let i = 0; i < arr.length; i++){
          if(rowIndex === arr[i]){
            // console.log("v",i)
            // console.log("index",rowIndex)
            return 'rowStyle'
          }
        } 
      },
      filter(v) {
        // 对绑定数据赋值
        this.options = this.copy.filter((item) => {
          // 如果直接包含输入值直接返回true
          const val = v.toLowerCase()
          if (item.label.indexOf(val) !== -1) return true
          if (item.szm.substring(0, 1).indexOf(val) !== -1) return true
          if (item.szm.indexOf(val) !== -1) return true
        })
      },    
      showOption(){
        let inputContent = document.getElementsByClassName('el-input__inner')[0].value
        console.log(inputContent.length)
        if(inputContent.length != 0){
          this.optionVisible = true;
        }
      },
      showMessage(e){
        console.log(e)
        this.v_company = e.company;
        this.v_label = e.label;
        this.v_school = e.school
      }
    },

    created () {
      this.getIndexStyle()
      this.getSeparatedStyles()
      this.getFontFiles()
    },

    mounted () {
      // 保留数据源
      this.copy = Object.assign(this.options)
      document.addEventListener('keydown',this.FNine)
      this.$nextTick(() => {
        this.originalStylesheetCount = document.styleSheets.length
      })
    }
  }
</script>
<style lang = "scss">
  .rowStyle{
    background-color:#f0f9eb!important;
  }
  .el-select-dropdown__item span{
    width:100px;
    text-align:center;
  }
  .el-input--prefix .el-input__inner {
    padding-left: 110px;
  }
  .prefixSlot{
    height: 36px;
    width: 90px;
    display: block;
    line-height: 36px;
    border-right: 1px solid #f1f1f1;
  }
  .tableHeader{
    background:rgb(64, 158, 255);
    color:#fff;
    height: 40px;
    line-height: 40px;
    font-size:14px;
    font-family:HiraginoSansGB-W3;
    font-weight:600;
    padding: 0 20px;
  }
  .tableHeader span{
    width:100px;
    text-align: center;
  }
  .title{
    margin-left:20px;
    width:400px;
  }
  .title span{
    display:block;
    width:150px;
    margin-bottom: 20px;
    font-weight: 800;
  }
  .popper__arrow{
    display: none!important;
  }
  .el-select-dropdown{
    box-shadow: none!important;
    min-width: 0px;
    border:0!important;
  }
  .el-scrollbar__view{
    padding:0;
    background: #ebf3ff;
  }
</style>