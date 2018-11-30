<template>
  <div id="app">
    <div class="container-fluid">
      <div class="row">
        <div id="side-buttons" class="col-md-2 bg-left min-height ">
          <Input suffix="ios-search" placeholder="搜索组件"/>
          <Divider orientation="left" :style="{'fontSize':'12px'}">组件</Divider>
          <Collapse
            v-model="value"
            :accordion="isAccordion"
          >
            <Panel name="source">
              {{info.source}}
              <Source slot="content" v-for="item in ['a','b','c','d']" :id="item"></Source>
            </Panel>
            <Panel name="data">
              {{info.data}}
              <Data slot="content" v-for="item in ['a','b','c','d']" :id="item">
              </Data>
            </Panel>
            <Panel name="feature">
              {{info.feature}}
              <Feature slot="content" v-for="item in ['a','b','c','d']" :id="item">
              </Feature>
            </Panel>
            <Panel name="classification">
              {{info.classification}}
              <Classification slot="content" v-for="item in ['a','b','c','d']" :id="item">
              </Classification>
            </Panel>
            <Panel name="regression">
              {{info.regression}}
              <Regression slot="content" v-for="item in ['a','b','c','d']" :id="item">
              </Regression>
            </Panel>
            <Panel name="model">
              {{info.model}}
              <Model slot="content" v-for="item in ['a','b','c','d']" :id="item">
              </Model>
            </Panel>
            <Panel name="other">
              历史流程图
            </Panel>
          </Collapse>
        </div>
        <div class="col-md-10 bg-success min-height diagramContainer " id="drop-bg">
          <Card
            style="position: fixed;bottom: 10px;width: 200px;right:80px;background: aliceblue"
            dis-hover
          >
            <Row>
              <!--<i-col span="6"><Button icon="ios-expand">放大</Button></i-col>-->
              <!--<i-col span="6"><Button icon="ios-contract">缩小</Button></i-col>-->
              <i-col span="12">
                <Button type="primary" icon="md-play">运行</Button>
              </i-col>
              <i-col span="12">
                <Button type="primary" icon="md-share-alt ">发布</Button>
              </i-col>
            </Row>
          </Card>
        </div>
      </div>
    </div>

    <!--<Affix :offset-bottom="20">-->
    <!--<span class="demo-affix">Fix at the bottom 20px</span>-->
    <!--</Affix>-->
    <!--右键点击出来的菜单-->
    <div id="menu">
      <Dropdown @on-click="handleClick" slot="list">
        <DropdownMenu>
          <DropdownItem name="config">
            <Icon size="18" type="ios-create-outline"/>
            配置
          </DropdownItem>
          <Dropdown
            :visible="isShowMenu"
            :transfer="true"
            trigger="custom"
            placement="right-start"
            @on-click="showDataMark"
          >
            <DropdownItem name="result">
              <Icon size="18" type="ios-podium-outline"/>
              查看
              <Icon type="ios-arrow-forward"></Icon>
            </DropdownItem>
            <DropdownMenu slot="list">
              <DropdownItem v-for="item in dataNameList" :name="item.id">{{item.dataName}}</DropdownItem>
            </DropdownMenu>
          </Dropdown>
          <DropdownItem name="delete" style="color: #ed4014">
            <Icon size="18" type="ios-trash-outline"/>
            删除
          </DropdownItem>
        </DropdownMenu>
      </Dropdown>
    </div>
    <!--配置抽屉组件-->
    <Drawer
      title="数据配置"
      v-model="isDrawer"
      width="300"
      :mask-closable="false"
      :styles="styles"
    >
      <Form
        ref="formValidate"
        :model="formValidate"
        :rules="ruleValidate"
        :label-width="80">
        <FormItem label="配置名称:" prop="configName">
          <Input v-model="formValidate.configName" disabled/>
        </FormItem>
        <FormItem label="数据表:" prop="tableName">
          <Select
            placeholder="---请选择---"
            clearable
            v-model="formValidate.tableName"
            :label-in-value="true"
            @on-change="getFileds"
          >
            <Option v-for="name in tableNames" :value="name" :key="name">
              {{ name }}
            </Option>
          </Select>
        </FormItem>
        <FormItem label="目标字段:" prop="fieldName">
          <Select
            placeholder="---请选择---"
            clearable
            v-model="formValidate.fieldName"
            :label-in-value="true"
          >
            <Option v-for="item in fieldList" :value="item.field_name" :key="item.field_name">
              {{ item.field_name }}
            </Option>
          </Select>
        </FormItem>
        <FormItem label="模型类型:" prop="modelType">
          <Select
            placeholder="---请选择---"
            clearable
            v-model="formValidate.modelType"
            :label-in-value="true"
          >
            <Option v-for="type in modelTypes" :value="type.label" :key="type.label">
              {{ type.label }}
            </Option>
          </Select>
        </FormItem>
        <FormItem label="数据名称:" prop="dataName">
          <Input v-model="formValidate.dataName"/>
        </FormItem>
        <div class="demo-drawer-footer">
          <Button style="margin-right: 8px" @click="dataReset('formValidate')">重置</Button>
          <Button type="primary" @click="dataHandle('formValidate')" :loading="dataHandleLoading">运行</Button>
        </div>
      </Form>
    </Drawer>
  </div>
</template>
<script>
  import Source from './Source'
  import Data from './Data'
  import Feature from './Feature'
  import Classification from './Classification'
  import Regression from './Regression'
  import Model from './Model'
  import Mustache from 'mustache/mustache.min'
  import uuid from '../js/uuid.min'
  import visoConfig from '../config'

  export default {
    name: 'flow-chart',
    components: {Source, Data, Feature, Classification, Regression, Model},
    data() {
      const validateTable = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请选择表名'));
        } else {
          callback();
        }
      };
      const validateField = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请选择目标字段'));
        } else {
          callback();
        }
      };
      const validateModel = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('请选择模型类型'));
        } else {
          callback();
        }
      };
      const validateName = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('数据名称不能为空'));
        } else {
          let regex = /^[a-zA-Z0-9_]{1,}$/;
          if (!value.match(regex)) {
            return callback(new Error('数据名称只能有字母、数字、下划线组成'));
          } else {
            callback()
          }
        }
      };
      return {
        value: "组件面板",
        info: {
          source: '数据源组件',
          data: '数据组件',
          feature: '特征组件',
          classification: '分类算法组件',
          regression: '回归算法组件',
          model: '模型组件'
        },
        dataHandleLoading: false,
        isDrawer: false,
        formValidate: {
          configName: '',
          tableName: '',
          fieldName: '',
          modelType: '',
          dataName: ''
        },
        ruleValidate: {
          tableName: [{validator: validateTable, trigger: 'change'}],
          fieldName: [{validator: validateField, trigger: 'change'}],
          modelType: [{validator: validateModel, trigger: 'change'}],
          dataName: [{validator: validateName, trigger: 'blur'}],
        },
        styles: {
          height: 'calc(100% - 55px)',
          overflow: 'auto',
          paddingBottom: '53px',
          position: 'static'
        },
        //左侧组件ID
        id: null,
        //组件生成ID
        currentId: null,
        //当前组件信息
        componentInfo: null,
        position: {
          top: 0,
          left: 0
        },
        isAccordion: true,
        modelTypes: [
          {
            value: "fl",
            label: "分类"
          },
          {
            value: "hg",
            label: "回归"
          }
        ],
        isShowMenu: false,//是否显示查看菜单
        dataConfig: null,//当前选中的配置信息
        sourceList: [],//数据源组件组
        tableNames: [],//数据配置下的表名
        fieldList: [],//表名下的字段组,
        dataNameList: [],//配置源对应的数据组
      }
    },
    //在模板渲染成html前调用，即通常初始化某些属性值，然后再渲染成视图
    created() {
      //刷新一次页面 只有第一次刷新才能使用 待查明原因
      // if (location.href.indexOf("#reloaded") == -1) {
      //   location.href = location.href + "#reloaded";
      //   location.reload();
      // }

    },
    //在模板渲染成html后调用，通常是初始化页面完成后，再对html的dom节点进行一些需要的操作
    mounted() {
      console.log('进入初始化方法1')
      //此处需要初始化
      jsPlumb.ready(this.main)
      jsPlumb.importDefaults({
        ConnectionsDetachable: false
      })
    },
    watch: {
      value: function () {
        if (this.value.length === 1) {
          switch (this.value[0]) {
            case 'source'://初始化数据源组

              break
            case 'data'://当选出数据源组件时 初始化数据组件
              break
            default:
          }
        }

        jsPlumb.ready(this.main)
        jsPlumb.importDefaults({
          ConnectionsDetachable: false
        })
      },
      //监控componentInfo以便改变数据
      componentInfo: function () {
        //重置表单
        this.dataReset("formValidate")
        let temp = this
        if (temp.componentInfo != null && temp.componentInfo != undefined) {
          temp.formValidate.configName = temp.componentInfo.textContent.trim()
          //根据配置名称获取 当前数据源配置
          temp.sourceList.forEach(function (value) {
            if (value.config_name == temp.formValidate.configName) {
              temp.dataConfig = value
            }
          })
        }
      }
    },
    methods: {
      //显示数据运行结果
      showDataMark(id) {
        if (id == null || id == undefined || id === 'result') {
          return
        }

      },
      //整页加载运行
      handleSpinCustom() {
        let temp = this
        temp.$Spin.show({
          render: (h) => {
            return h('div', [
              h('Icon', {
                'class': 'demo-spin-icon-load',
                props: {
                  type: 'ios-loading',
                  size: 18
                }
              }),
              h('div', '数据正在处理中')
            ])
          }
        });

        // setTimeout(() => {
        //     temp.$Spin.hide();
        // }, 3000);
      },
      //重置输入框
      dataReset(name) {
        this.$refs[name].resetFields();
      },
      //开始运行
      dataHandle(name) {
      },
      //获取字段信息
      getFileds() {

      },

      clickConfig() {
        this.isDrawer = true
      },
      clickResult() {
      },
      clickDelete() {
        this.isShowMenu = false
        this.emptyNode(this.currentId)
        document.querySelector("#menu").style.width = 0
      },
      //处理右键菜单功能
      handleClick(name) {

        this.dataNameList = []

        if (name === 'config') {
          this.clickConfig()
        } else if (name === 'result') {
          this.isShowMenu = true
          this.clickResult()
        } else if (name === 'delete') {
          this.clickDelete()
        }
      },
      // 获取基本配置
      getBaseNodeConfig: function () {
        return Object.assign({}, visoConfig.baseStyle)
      },
      // 让元素可拖动
      addDraggable: function (id) {
        jsPlumb.draggable(id, {
          containment: 'parent',
          grid: [10, 10]
        })
      },
      // 设置入口点
      setEnterPoint: function (id) {
        let config = this.getBaseNodeConfig()
        config.isSource = false
        config.maxConnections = -1
        jsPlumb.addEndpoint(id, {
          anchors: 'Top',
          uuid: id + '-in'
        }, config)
      },
      // 设置出口点
      setExitPoint: function (id, position, template) {
        let config = this.getBaseNodeConfig()
        config.isTarget = false
        config.maxConnections = template ? -1 : 1

        jsPlumb.addEndpoint(id, {
          uuid: id + '-out',
          anchors: position || 'BottomCenter'

        }, config)
        jsPlumb.repaintEverything();
        console.log('id2',id)
        console.log('创建出口点',jsPlumb.getEndpoints(id))
      },
      setExitMenuItem: function (id) {
        let temp = this
        $('#' + id).find('li').each(function (key, value) {
          temp.setExitPoint(value.id, 'Right')
        })
      },
      // 初始化节点设置
      initSetNode: function (template, id) {
        //使得当前可拖动
        this.addDraggable(id)
        if (template === 'tpl-feature') {
          this.setEnterPoint(id)
          this.setExitPoint(id, null, template)
        } else if (template === 'tpl-source') {
          this.setExitPoint(id, null, template)
        } else {
          this.setEnterPoint(id)
          this.setExitPoint(id)
        }
      },
      // 渲染html
      renderHtml: function (type, position) {
        switch (type) {
          case "tpl-feature":
            let tempHtml = [
              // '<body id="tpl-feature" >',
              // '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;" >',
              // '   <div class="panel panel-default panel-node panel-info">',
              // '       <div  id="{{id}}-head" data-id="{{id}}" class="panel-heading"><i class="fa fa-navicon" aria-hidden="true">特征组件一</i> ',
              // '           <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              // '       </div>',
              // '       <ul class="list-group">',
              // '           <li id="{{id}}|classification" class="list-group-item panel-node-list" >分类算法',
              // '           </li>',
              // '           <li id="{{id}}|regression" class="list-group-item panel-node-list" >回归算法',
              // '           </li>',
              // '       </ul>',
              // '   </div>',
              // '</div>',
              // '</body>'
              '<body id="tpl-feature" >',
              '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;"  >',
              '   <a class="btn btn-default" href="#" role="button">',
              '       <i class="fa fa-clone" aria-hidden="true"></i>',
              '       {{textContent}}',
              '       <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              '   </a>',
              '</div>',
              '</body>'
            ].join('')
            Mustache.parse(tempHtml);
            return Mustache.render(tempHtml, position)
          case 'tpl-source':
            tempHtml = [
              '<body id="tpl-source" >',
              '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;"  >',
              '   <a class="btn btn-default" href="#" role="button">',
              '       <i class="fa fa-database" aria-hidden="true"></i>',
              '        {{textContent}}',
              '       <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              '   </a>',
              '</div>',
              '</body>'
            ].join('')
            //构建HTML
            Mustache.parse(tempHtml);
            //传值
            return Mustache.render(tempHtml, position)
          case 'tpl-data':
            tempHtml = [
              '<body id="tpl-data" >',
              '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;" >',
              '   <a class="btn btn-default" href="#" role="button">',
              '       <i class="fa fa-bars" aria-hidden="true"></i>',
              '       {{textContent}}',
              '       <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              '   </a>',
              '</div>',
              '</body>'
            ].join('')
            Mustache.parse(tempHtml);
            return Mustache.render(tempHtml, position)
          case 'tpl-classification':
            tempHtml = [
              '<body id="tpl-classification" >',
              '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;" >',
              '   <a class="btn btn-default" href="#" role="button">',
              '       <i class="fa fa-location-arrow" aria-hidden="true"></i>',
              '       {{textContent}}',
              '       <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              '   </a>',
              '</div>',
              '</body>'
            ].join('')
            Mustache.parse(tempHtml);
            return Mustache.render(tempHtml, position)
          case 'tpl-regression':
            tempHtml = [
              '<body id="tpl-regression" >',
              '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;" >',
              '   <a class="btn btn-default" href="#" role="button">',
              '       <i class="fa fa-location-arrow" aria-hidden="true"></i>',
              '        {{textContent}}',
              '       <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              '   </a>',
              '</div>',
              '</body>'
            ].join('')
            Mustache.parse(tempHtml);
            return Mustache.render(tempHtml, position)
          case 'tpl-model':
            tempHtml = [
              '<body id="tpl-model" >',
              '<div id="{{id}}"  class="pa" style="left:{{left}}px;top: {{top}}px;" >',
              '   <a class="btn btn-default" href="#" role="button">',
              '       <i class="fa fa-lightbulb-o" aria-hidden="true"></i>',
              '        {{textContent}}',
              '       <span class="delete-node pull-right" data-type="deleteNode" data-id="{{id}}">X</span>',
              '   </a>',
              '</div>',
              '</body>'
            ].join('')
            Mustache.parse(tempHtml);
            return Mustache.render(tempHtml, position)
        }
        return Mustache.render($('#' + type).html(), position)
      },
      getMousePos: function (event) {
        let e = event || window.event;
        let scrollX = document.documentElement.scrollLeft || document.body.scrollLeft;
        let scrollY = document.documentElement.scrollTop || document.body.scrollTop;
        let x = e.pageX || e.clientX + scrollX;
        let y = e.pageY || e.clientY + scrollY;
        // alert('x: ' + x + '\ny: ' + y);
        return {'x': x, 'y': y};
      },
      //右键点击打开菜单
      showChoose(position) {
        let temp = this
        $('#' + position.id).mousedown(function (e) {
          if (e.which == 3) {
            //获取我们自定义的右键菜单
            let menu = document.querySelector("#menu");
            //加上滚动条的高度
            // let scrollTop=document.documentElement.scrollTop||document.body.scrollTop;
            //根据事件对象中鼠标点击的位置，进行定位
            menu.style.left = temp.getMousePos(e).x  + 'px';
            menu.style.top = temp.getMousePos(e).y - 90 + 'px';
            // //改变自定义菜单的宽，让它显示出来
            menu.style.width = '95px';
          }
        })
      },
      //阻止浏览器默认右键点击事件并且用户点击左键关闭菜单
      closeChoose(areaId) {
        let temp = this
        $(areaId).bind("contextmenu", function () {
          return false;
        })
        //屏蔽自带右键菜单
        document.oncontextmenu = function (e) {
          return false;
        }
        $(areaId).mousedown(function (e) {
          if (e.which == 1) {
            temp.isShowMenu = false
            document.querySelector("#menu").style.width = 0
          }
        })
      },
      // 放入拖动节点
      dropNode: function (template, position, textContent) {
        let areaId = '#drop-bg'
        position.left -= $('#side-buttons').outerWidth()
        position.left = Math.ceil(position.left)
        position.top = Math.ceil(position.top)
        position.id = uuid.v1()
        position.textContent = textContent
        let html = this.renderHtml(template, position)
        $(areaId).append(html)
        console.log(position)
        //不同的组件事件不同
        if (template == 'tpl-source') {
          //给当前变量赋值
          this.currentId = position.id
          this.componentInfo = position
          this.showChoose(position)
        } else if (template == 'tpl-data') {

        }

        this.initSetNode(template, position.id)
        //将关闭菜单变为可触发事件
        this.closeChoose(areaId)
      },
      eventHandler: function (data) {
        if (data.type === 'deleteNode') {
          this.emptyNode(data.id)
        }
      },
      // 删除一个节点以及连线
      emptyNode: function (id) {
        jsPlumb.remove(id)
      },
      connectionBeforeDropCheck: function (info) {
        return info.sourceId.split("|")[0] !== info.targetId
      },
      //删除连线
      deleteLine: function (conn) {
        // if (confirm('确定删除所点击的链接吗？')) {
          jsPlumb.deleteConnection(conn)
        // }
      },
      main: function () {
        let temp = this
        $("#drop-bg").droppable({
          scope: 'ss',
          drop: function (event, ui) {
            temp.dropNode(ui.draggable[0].dataset.template, ui.position, ui.draggable[0].textContent)
          }
        })

        $('#app').on('click', function (event) {
          event.stopPropagation()
          event.preventDefault()
          temp.eventHandler(event.target.dataset)
        })

        // 单点击了连接线上的X号
        jsPlumb.bind('dblclick', function (conn, originalEvent) {
          temp.deleteLine(conn)
        })
        // 链接建立后的检查
        // 当出现自连接的情况后，要将链接断开
        jsPlumb.bind('beforeDrop', function (info) {
          return temp.connectionBeforeDropCheck(info)
        })
      }
    },

  }
</script>
<style>
  .demo-drawer-footer {
    width: 100%;
    position: absolute;
    bottom: 0;
    left: 0;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    background: #fff;
  }

  /*css代码*/
  #menu {
    width: 0; /*设置为0 隐藏自定义菜单*/
    overflow: hidden; /*隐藏溢出的元素*/
    box-shadow: 0 1px 1px #888, 1px 0 1px #ccc;
    position: absolute; /*自定义菜单相对与body元素进行定位*/
    background-color: white;
  }

  .demo-spin-icon-load {
    animation: ani-demo-spin 1s linear infinite;
  }

  .menu {
    width: 160px;
    line-height: 25px;
    padding: 0 10px;
  }

  .min-height {
    height: 1200px;
  }

  .bg-left {
    background: #e6f5ce;
  }

  .btn {
    z-index: 2;
  }

  .pa {
    position: absolute;
  }

  .fixed-node {
    position: absolute;
    top: 80px;
    left: 150px;
  }

  #end-node {
    left: 150px;
    top: 700px;
  }

  .panel-node {
    width: 150px;
    display: inline-block;
  }

  .panel-node-list {
    padding: 10px 10px;
  }

  .delete-node {
    cursor: pointer;
    width: 20px;
    display: inline-block;
    text-align: center;
  }

  .delete-node:hover {
    color: red
  }

  .diagramContainer {
    padding: 20px;
    width: 80%;
    border: 1px solid #cfd2eb;
    background-image: url('../static/41F58PIC8Wg_1024.jpg');
    background-repeat: repeat;
  }

  .panel {
    margin-bottom: 20px;
    background-color: #fff;
    border: 1px solid transparent;
    border-radius: 4px;
    -webkit-box-shadow: 0 1px 1px rgba(0, 0, 0, .05);
    box-shadow: 0 1px 1px rgba(0, 0, 0, .05)
  }

  .panel-default {
    border-color: #ddd
  }

  .panel-info {
    border-color: #bce8f1
  }

  .panel-heading {
    padding: -10px -15px;
    border-bottom: 1px solid transparent;
    border-top-left-radius: 3px;
    border-top-right-radius: 3px
  }

  .pull-right {
    float: right
  }

  .list-group {
    padding-left: 0;
    margin-bottom: 20px
  }

  .list-group-item {
    position: relative;
    display: block;
    padding: 10px 15px;
    margin-bottom: -1px;
    background-color: #fff;
    border: 1px solid #ddd
  }

  .list-group-item:first-child {
    border-top-left-radius: 4px;
    border-top-right-radius: 4px
  }

  .list-group-item:last-child {
    margin-bottom: 0;
    border-bottom-right-radius: 4px;
    border-bottom-left-radius: 4px
  }

  .btn-default {
    color: #333;
    background-color: #e6e6e6;
    border-color: #adadad
  }
</style>
