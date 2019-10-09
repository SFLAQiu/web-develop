
在参与各种app业务开发的过程中，大部分都会遇到需要对某些功能/界面/数据可以灵活的管理后台控制，客户端根据配置变化而变化，不需要发版本就可以解决这些需求，大致功能需求就是需要提供一个后台功能，能够给产品/运营童鞋进行配置管理，然后通过服务端接口输出给客户端进行逻辑/渲染使用，这里针对这种场景，分享一个相对通用的解决方案


#### 项目背景

当前项目中针对这种配置的需求，每次都需要开发人员重新开发后台表单，然后修改配置接口针对配置进行输出，因为这个功能的开发要归宿到很早以前，也不知道当初为啥要这么做，现在存在的问题就是不容易维护和拓展，以及重复开发的成本

#### 整理需求
* 配置管理后台
    * 支持版本控制
    * 支持客户端类型（安卓/IOS/所有）
    * 表单可配置
* 配置输出接口
    * 增量下发
    * 保证高可用，高稳定，高性能
* 客户端
    * 接口下发配置数据进行缓存 
    
#### 技术背景

* 管理后台：php服务端+jquery+bootstrap
* 接口项目：php服务端


#### 技术过程

* 前端技术选型：
    * vuejs
    * element ui

* 核心问题，如何后台配置生成表单（开发人员来配置）？

初步计划是通过配置表单的JSON生成element ui的表单，进行了一些调研，也找到可以通过配置JSON生成element ui表单的js库，感觉灵活性差了些，而且当时还不支持富文本，感觉后续拓展也是大问题，所以弃用，后面尝试自己来实现，通过vuejs+element ui组件相对简单的方式实现了这个配置表单的功能，能够支持基本需求，具体看后面代码（简单粗暴）

* 接口数据增量下发，以及客户端获取配置时机和缓存策略

客户端每次启动的时候去获取一次配置，缓存【配置数据】，新增配置添加到缓存，已经存在进行替换   
接口输出【配置数据】的同时在响应头上【timestamp】= 带上当前请求的服务器时间戳  
客户端获取数据，缓存【配置数据】&【timestamp】   
客户端下次请求的头上带上【timestamp】= 缓存的时间戳，第一次请求可以不用    
服务端接收到请求的时候获取客户端的【timestamp】，过滤配置的时候校验最后更新时间>=【timestamp】进行输出【配置数据】   



* 保障高可用，高稳定，高性能，容错

配置数据进行多级缓存，第一级缓存【redis】，第二级缓存【服务器内存】（php apcu）  
接口优先从【服务器内存】中获取，如果不存在从【redis】 并同步到【服务器内存】，不存在从【mysql】 并同步到【redis】，正常后台编辑完就同步到redis，【服务器内存】就进行短暂性的缓存（3s），保障在高并发的情况下可以快速下发，弊端就是数据变化的时候会延迟N/s后更新

客户端在获取缓存配置的时候如果不存在需要自己有个默认配置，极端情况下无法获取配置的容错机制，保障功能的正常运行


#### 解决方案

配置管理列表界面：
![配置管理](http://blog.thankbabe.com/imgs/config_center_manage.png?v=222)

配置添加和表单JSON配置界面（开发人员操作）：
![配置管理](http://blog.thankbabe.com/imgs/config_center_add.png?v=222)

配置数据表单界面（产品/运营童鞋操作）：
![配置管理](http://blog.thankbabe.com/imgs/config_center_data_edit.png?v=222)


前端框架/库：
* [vuejs](https://cn.vuejs.org/)
* [element ui](https://element.eleme.cn/#/zh-CN) 饿了么UI
* [jsoneditor](https://github.com/josdejong/jsoneditor) json编辑组件
* [VueQuillEditor](https://github.com/surmon-china/vue-quill-editor) vuejs富文本组件

主要的代码内容，如下： 

表设计：
```sql
-- 配置中心表
CREATE TABLE `config_center` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `title` varchar(100) NOT NULL DEFAULT '' COMMENT '标题',
  `code` varchar(60) NOT NULL DEFAULT '' COMMENT '标识',
  `platform` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0=所有，1=IOS，2=安卓',
  `template` tinyint(4) NOT NULL DEFAULT '1' COMMENT '模板标识',
  `form_json` text NOT NULL COMMENT '表单JSON',
  `form_data` text NOT NULL COMMENT '表单数据',
  `description` varchar(255) NOT NULL DEFAULT '' COMMENT '描述',
  `app_version` varchar(15) NOT NULL DEFAULT '' COMMENT 'app版本',
  `app_version_compare` varchar(10) NOT NULL DEFAULT '' COMMENT 'app版本比较符号',
  `operator` varchar(20) NOT NULL DEFAULT '' COMMENT '编辑人',
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '更新时间',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态，1=有效，-1=删除',
  PRIMARY KEY (`id`),
  KEY `index_code` (`code`),
  KEY `index_update_at_platform` (`update_at`,`platform`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

```
表单配置JOSN内容：
```js
[
    {
        el: "input",
        type: "textarea",
        name: "名字",
        field: "name",
        value: "6666",
        rule: [
            {
                required: true,
                message: "请输入活动名称",
                trigger: "blur"
            }
        ]
    },
    {
        el: "input-number",
        type: "",
        name: "数字",
        field: "number",
        value: 1,
        min:1,
        max:1000,
        rule: [
            {
                required: true,
                message: "数字",
                trigger: "blur"
            }
        ]
    },
    {
        el: "input",
        type: "text",
        name: "描述",
        field: "desc",
        value: "",
        rule: [
            {
                required: true,
                message: "请输入活动名称",
                trigger: "blur"
            }
        ]
    },
    {
        el: "editor",
        type: "",
        name: "富文本",
        field: "editor",
        value: "",
        rule: [
            {
                required: true,
                message: "请输入内容",
                trigger: "blur"
            }
        ]
    },
    {
        el: "date",
        type: "datetimerange",
        name: "日期范围",
        field: "datetime",
        value: ["2019-01-01 10:00:00", "2019-03-01 08:00:00"],
        rule: [
            {
                required: true,
                message: "必须",
                trigger: "blur"
            }
        ]
    },
    {
        el: "switch",
        type: "",
        name: "开关",
        field: "open",
        value: false,
        rule: [
            {
                required: true,
                message: "必须",
                trigger: "blur"
            }
        ]
    },
    {
        el: "date",
        type: "datetime",
        name: "活动时间",
        field: "datet",
        value: "2019-01-01"
    },
    {
        el: "slider",
        type: "",
        name: "范围",
        field: "fw",
        value: 0,
        max: 500
    },
    {
        el: "color",
        type: "",
        name: "颜色",
        field: "color",
        value: ""
    },
    {
        el: "radio",
        type: "",
        name: "类型",
        field: "type",
        value: 0,
        options: [
            {
                label: "类型1",
                value: 1
            },
            {
                label: "类型2",
                value: 2
            },
            {
                label: "类型3",
                value: 3
            }
        ]
    },
    {
        el: "select",
        type: "",
        name: "食品",
        field: "foods",
        value: "黄金糕",
        options: [
            {
                value: 1,
                label: "黄金糕"
            },
            {
                value: 2,
                label: "双皮奶"
            },
            {
                value: 3,
                label: "蚵仔煎"
            },
            {
                value: 4,
                label: "龙须面"
            },
            {
                value: 5,
                label: "北京烤鸭"
            }
        ]
    },
    {
        el: "checkbox",
        type: "",
        name: "城市",
        field: "city",
        value: [0],
        options: [
            {
                value: 1,
                label: "上海"
            },
            {
                value: 2,
                label: "深圳"
            },
            {
                value: 3,
                label: "北京"
            }
        ]
    }
];
```

vuejs + element ui 表单模板主要代码（简单粗暴）
```html
<el-form size="small" :rules="rules" ref="form" :model="form" label-width="80px">
    <el-form-item v-for="(item,index) in formData" :key="item.key" :label="item.name" :prop="item.field">
        <!--input 输入框-->
        <el-input v-if="item.el==='input'" :type="item.type" style="width:400px" v-model="form[item.field]"></el-input>
    
        <!--input 数字输入框-->
        <el-input-number v-if="item.el==='input-number'" :min="item.min" :max="item.max" v-model="form[item.field]"></el-input-number>
    
        <!--datetime 时间-->
        <el-date-picker v-if="item.el==='date'" :type="item.type" v-model="form[item.field]" placeholder="选择日期时间">
        </el-date-picker>
    
        <!--switch 开关-->
        <el-switch v-if="item.el==='switch'" v-model="form[item.field]" active-text="" inactive-text="">
        </el-switch>
    
        <!--滑块-->
        <el-slider v-if="item.el==='slider'" v-model="form[item.field]" :max="item.max?item.max:100"></el-slider>
    
        <!--颜色选择-->
        <el-color-picker v-if="item.el==='color'" v-model="form[item.field]"></el-color-picker>
    
        <!--单选-->
        <el-radio v-if="item.el==='radio'" v-for="(option,index) in item.options" :key="option.key" v-model="form[item.field]" :label="option.value">
            {{ option.label }}
        </el-radio>
    
        <!--多选-->
        <el-checkbox-group v-if="item.el==='checkbox'" v-model="form[item.field]">
            <el-checkbox v-for="(option,index) in item.options" :key="option.value"  :label="option.value">
                {{ option.label }}
            </el-checkbox>
        </el-checkbox-group>
    
        <!--选择器-->
        <el-select v-if="item.el==='select'" v-model="form[item.field]" placeholder="请选择">
            <el-option v-for="option in item.options" :key="option.value" :label="option.label" :value="option.value">
            </el-option>
        </el-select>
    
        <!-- 富文本-->
        <quill-editor v-if="item.el==='editor'" v-model="form[item.field]"></quill-editor>
    </el-form-item>                
</el-form>

```
js代码：
```js

//富文本组件
Vue.use(VueQuillEditor);
$vm = new Vue({
    el: "#app",
    data: {
        template: "1",
        form: {},
        rules: {},
        formData: {},
    },
    methods: {
        useTemplate: function () {
            switch (this.template) {
                case "1": {
                    var formJson = [];
                    if (this.config['form_json']) {
                        formJson = this.config['form_json'];
                    } else if (templateOneJson) {
                        formJson = templateOneJson;
                    }
                    editorJson(formJson);
                    this.createForm(formJson);
                    return
                }
            }
        },
        //预览
        review: function () {
            var jsonData = editor.get();
            this.createForm(jsonData);
        },
        //根据配置的JSON，解析出构造表单需要的Vue数据
        getData: function (json) {
            var data = {
                //表单数据
                form: {},
                //表单验证规则
                rules: {},
                //表单控件配置
                formData: {}
            };
            //构造数据
            for (var index in json) {
                var item = json[index];
                data.form[item.field] = item.value;
                if (item.rule) data.rules[item.field] = item.rule;
            }
            data.formData = json;
            return data;
        },
        //创建表单Vue对象
        formVue: function (data) {
            Vue.set($vm, "form", data.form);
            Vue.set($vm, "rules", data.rules);
            Vue.set($vm, "formData", data.formData);
            // $vm.$forceUpdate();
        },
        //根据配置JSON生成Form表单
        createForm: function (json) {
            var data = this.getData(json);
            console.log(data);
            this.formVue(data);
        }
    }
});

```

前端部分因为基于原有项目技术背景拓展，用最原始的link引入方式，而且没有拉到前端同学参与，前端部分如果可以把后台功能进行前后端分离，然后基于组件化封装那就最好不过了，存后端童鞋折腾想想就好，low了点，能用哈，不过不影响基本实现思路可借鉴参考

#### 总结

当你在开发产品需求时候，除了要解决眼前的问题，是否有思考过之前或者将来也会遇到很多类似的问题。把你的解决方案从解决一个问题扩展到解决一类问题是一项非常重要的能力，也往往是区分新人与资深技术人员的一条分界线

---   

> 发布时间：2018-10-08