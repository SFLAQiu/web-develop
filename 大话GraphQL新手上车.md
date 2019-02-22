## 大话GraphQL新手上车


![graphql](http://blog.thankbabe.com/imgs/graphql.png?v=2)

### GraphQL是什么？
GraphQL 既是一种用于API的查询语言也是一个满足你数据查询的运行时（来自：[官方解释](http://graphql.cn/learn/)）

理解起来就是，GraphQL有自己查询语法，发起的API请求中通过传递查询语句来告诉服务端需要哪些操作和具体数据字段，GraphQL定义了实现规范，各种的语言分别实现了GraphQL功能框架，通过框架可以对查询语法进行解释执行，然后返回数据输出给客户端

![graphql](http://blog.thankbabe.com/imgs/graphql-server.png?v=2)











---


### GraphQL的优势

> 以下所有查询和输出都是来自我的DEMO，DEMO的实现和源码Github地址下面会提到

**语法特性满足各种需求**

* 支持多操作：query->查询，mutation->修改，规范是写在查询语句前，默认不写就是query
* 支持参数，实现各种功能，比如：查询数据，排序，分页，... ...等
* 语法其他特性，别名，片段，定义变量，指令，... ...等


```
# 查询语句-有参数
query{
  student(id:86){
    id
    name
    sclass{
      id
      num
      level
      heads
    }
  }
}
```

```
# 输出
{
  "data": {
    "student": {
      "id": 86,
      "name": "Emma",
      "sclass": {
        "id": 9,
        "num": 8,
        "level": 3,
        "heads": 68
      }
    }
  }
}
```

```
# 修改
mutation {
  update(id: 86, name: "66666") {
    rt
    msg
  }
}

```

```
# 输出
{
  "data": {
    "update": {
      "rt": 1,
      "msg": "bingo"
    }
  }
}
```
---

**查询友好性，查询和输出关联**

看查询语句是不是感觉有点儿JSON的味道？查询语法类JSON格式，前后端都可以很容易上手，查询语句和输出数据有紧密的关联性，通过分析查询语句就知道输出的数据内容字段有哪些

---

**灵活性，请求你所要的数据，不多不少**

可以自定义查询语句来获取需要使用的字段，避免无用字段的输出，减少不必要数据块/数据字段查询逻辑


> 多字段

```
# 查询语句
query{
  students{
    id
    name
    classid
    sclass{
      id
      num
      level
      heads
    }
  }
}
```

```
# 输出
{
  "data": {
    "students": [
       {
        "id": 19,
        "name": "Savannah",
        "classid":22,
        "sclass": {
          "id": 22,
          "num": 6,
          "level": 4,
          "heads": 57
        }
      },
      {
        "id": 34,
        "name": "Ariana",
        "classid":33,
        "sclass": {
          "id": 33,
          "num": 3,
          "level": 4,
          "heads": 57
        }
      }
    ]
  }
}
```

> 去掉了不使用的字段输出，少了字段sclass，就可以不进行sclass数据查询

```
# 查询语句
query{
  students{
    id
    name
  }
}
```

```
# 输出
{
  "data": {
    "students": [
      {
        "id": 19,
        "name": "Savannah"
      },
      {
        "id": 34,
        "name": "Ariana"
      }
    ]
  }
}
```

---


**API演进，无需划分版本**

API版本迭代无需要进行版本号区分，添加字段不影响现有查询，请求发起者可以自己定义想要的查询信息

```
# Say No
http://api.xxx.com/student/v1/
http://api.xxx.com/student/v2/
# ... 
```

---
 
**自检性，可查询输出所有定义**

这个是GraphQL一个很Nice的特性，就是GraphQL服务API可以通过语句查询出它所支持的类型，开发可以不需要花时间写API文档，GraphQL直接帮助开发者快速了解API。


```
# 查询语句
{
  __type(name: "MStudentType") {
    kind
    name
    fields {
      name
      description
      type {
        name
      }
    }
  }
}


```

```
# 输出
{
  "data": {
    "__type": {
      "kind": "OBJECT",
      "name": "MStudentType",
      "fields": [
        {
          "name": "id",
          "description": "学号",
          "type": {
            "name": null
          }
        },
        {
          "name": "name",
          "description": "学生名",
          "type": {
            "name": null
          }
        },
        {
          "name": "age",
          "description": "年龄",
          "type": {
            "name": null
          }
        },
        {
          "name": "birthdate",
          "description": "生日",
          "type": {
            "name": null
          }
        },
        {
          "name": "sclass",
          "description": "班级信息",
          "type": {
            "name": "MClassType"
          }
        }
      ]
    }
  }
}

```

基于自检性，GraphQL开源了辅助工具GraphiQL，方便GraphQL接口调试和自动生成接口文档     
* GraphQL辅助工具：GraphiQL，可以调试查询语句，并对接口定义的schema进行文档可视化展示
  * 查询语句进行感知
  * 错误提示
  * 语句格式化
  * 执行查询
  * 查看接口定义的schema文档信息
    
> graphql-dotnet开源项目里的GraphiQL要接入自己开发GraphQL接口，还需要进行简单的修改调整，后面会说到

---


### .NET下的入门教程

* 构建ASP.NET MVC5 WebAPI 项目
* NutGet引入程序包
    * [GraphQL](https://github.com/graphql-dotnet/graphql-dotnet/) 
    * [GenFu](https://github.com/MisterJames/GenFu)，用于初始化测试数据
* 基于GraphQL简单实现一个学生查询API
    * 支持查询学生信息列表
    * 支持查询学生的班级信息
    * 支持查询学号对应的学生信息
    * 支持修改学生名字

定义【数据类】MStudent.cs(学生类)，MClass.cs(班级类)，MResult.cs(执行结果类)

```csharp

public class MStudent {
    /// <summary>
    /// 学号
    /// </summary>
    public int Id { get; set; }
    /// <summary>
    /// 名字
    /// </summary>
    public string Name { get; set; }
    /// <summary>
    /// 年龄
    /// </summary>
    public int Age { get; set; }
    /// <summary>
    /// 所在班级编号
    /// </summary>
    public int ClassId { get; set; }
    /// <summary>
    /// 生日
    /// </summary>
    public DateTime Birthdate { get; set; }
    /// <summary>
    /// 班级
    /// </summary>
    public MClass SClass { get; set; }
}

public class MClass {
    public int Id { get; set; }
    /// <summary>
    /// 年级
    /// </summary>
    public int Level { get; set; }
    /// <summary>
    /// 第几班
    /// </summary>
    public int Num { get; set; }
    /// <summary>
    /// 总人数
    /// </summary>
    public int Heads { get; set; }
}

public class MResult {
    /// <summary>
    /// 输出结果,0=失败，1=成功
    /// </summary>
    public int rt { get; set; }
    /// <summary>
    /// 说明信息
    /// </summary>
    public string msg { get; set; }
}

```

定义GraphType类 MStudentType,MClassType,MResultType 继承ObjectGraphType<TSourceType> ，TSourceType泛型对应到【数据类】     
构造函数里通过Field去添加可以被查询的数据字段，包括：描述以及字段内容获取的处理方法，等    

```csharp

public class MStudentType : ObjectGraphType<MStudent> {
    private static BStudent _bll { get; set; }
    public MStudentType() {
        if (_bll == null) _bll = new BStudent();
        Field(d => d.Id).Description("学号");
        Field(d => d.Name).Description("学生名");
        Field(d => d.Age).Description("年龄");
        Field(d => d.Birthdate).Description("生日");
        Field<MClassType>("sclass", resolve: d => {
            //缓存中已经存在就直接返回
            if (d.Source.SClass != null) return d.Source.SClass;
            //从DB/缓存中获取数据
            var classId = d.Source?.ClassId ?? 0;
            if (classId > 0) d.Source.SClass = _bll.GetClass(d.Source.ClassId);
            return d.Source.SClass;
        },description:"班级信息");
    }
}

public class MClassType : ObjectGraphType<MClass> {
    public MClassType() {
        Field(d => d.Level).Description("年级");
        Field(d => d.Heads).Description("人数");
        Field(d => d.Id).Description("编号");
        Field(d => d.Num).Description("班级");
    }
}

public class MResultType : ObjectGraphType<MResult> {
    public MResultType() {
        Field(d => d.rt);
        Field(d => d.msg);
    }
}

```


定义Schema的操作类(query/mutation)，继承 ObjectGraphType，有：StudentQuery，StudentMutation


```csharp

public class StudentQuery : ObjectGraphType {
        public StudentQuery(BStudent bll) {
            //查询-有参数id
            Field<MStudentType>("student", arguments: new QueryArguments(new QueryArgument<IntGraphType>() {
                Name = "id"
            }), resolve: d => {
                var id = d.Arguments["id"].GetInt(0, false);
                return bll.GetModel(id); ;
            });
            //查询-列表
            Field<ListGraphType<MStudentType>>("students", resolve: d => {
                return bll.GetStudents();
            });
        }
    }
}

public class StudentMutation : ObjectGraphType {
    public StudentMutation(BStudent bll) {
        Field<MResultType>("update", arguments: new QueryArguments(
            new QueryArgument<IntGraphType> {
                Name = "id"
            },
            new QueryArgument<StringGraphType> {
                Name = "name"
            }
        ), resolve: (d) => {
            var id = d.Arguments["id"].GetInt(0, false);
            var name = d.Arguments["name"].GetString("");
            if (id <= 0) return new MResult {
                rt = 0,
                msg = "非法学号"
            };
            if (name.IsNullOrWhiteSpace()) return new MResult {
                rt = 0,
                msg = "非法名字"
            };
            var isSc = bll.UpdateName(id, name);
            if (!isSc) return new MResult {
                rt = 0,
                msg = "更新失败"
            };
            return new MResult {
                rt = 1,
                msg = "bingo"
            };
        });
    }
}

```

在控制器里添加接口，构造Schema对象，根据查询条件解析执行返回结果输出    
Query = StudentQuery，Mutation = StudentMutation    

```csharp
/// <summary>
/// graphql demo 接口
/// </summary>
/// <returns></returns>
[HttpPost]
[Route("query")]
public object Test_Query() {
    var r = HttpContext.Current.Request;
    var query = r.GetF("query");
    var bll = new BStudent();
    var schema = new Schema { Query = new StudentQuery(bll), Mutation = new StudentMutation(bll) };
    var result = new DocumentExecuter()
        .ExecuteAsync(options => {
            options.Schema = schema;
            options.Query = query;
        }).GetAwaiter();
    var json = new DocumentWriter(indent: true).Write(result);
    return result.GetResult();
}

```
---

**GraphiQL工具的接入**

* Git Clone [graphql-dotnet](https://github.com/graphql-dotnet/graphql-dotnet)
* 安装NodeJS环境
* 命令工具CMD打开graphql-dotnet/src/GraphQL.GraphiQL/ 执行下面命令
    * npm install -g yarn
    * yarn install
    * yarn start
* 运行graphql-dotnet/src/GraphQL.GraphiQL 就可以启动：http://localhost:47080/
* 根据自己的情况调整：graphql-dotnet/src/GraphQL.GraphiQL/app/app.js 脚本，如下面贴出的代码
    * url=graphql接口地址
    * 如果需要投入生产使用，可以把接口地址进行url传入，或者支持输入框输入地址，不固定接口地址
* 每次调整完需要重新执行  yarn start，前端会使用webpack进行打包操作，执行完成后刷新页面即可


```javascript

//调整如下
import React from 'react';
import ReactDOM from 'react-dom';
import GraphiQL from 'graphiql';
import axios from 'axios';
import 'graphiql/graphiql.css';
import './app.css';

function graphQLFetcher(graphQLParams) {
    console.log(graphQLParams["query"]);
    return axios({
        method: 'post',
        url: "http://127.0.0.1:5656/query",//window.location.origin + '/api/graphql',
        data: "query=" + graphQLParams["query"],
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
    }).then(resp => resp.data);
}
ReactDOM.render(<GraphiQL fetcher={graphQLFetcher} />, document.getElementById('app'));

```  

![graphql](http://blog.thankbabe.com/imgs/graphiql.jpg?v=1)

### 总结

* 对于应用我觉得可以尝试使用到新项目需求中，或者现有合适应用场景进行重构，等服务运行稳定，并且开发上手后即可进行大范围的使用
* 对于RestFul和GraphQL的比较，我觉得没有最好的协议，只有最合适的场景

---
### 资源
* Demo源码：
  * Demo代码到我的Gtihub项目（[GraphQLDemo](https://github.com/SFLAQiu/GraphQLDemo)）
* 学习资料
  * [知乎-什么是GraphQL](https://www.zhihu.com/question/264629587)
  * [GraphQL语法入门](http://graphql.cn/learn/)
  * [GraphQL中文官网](http://graphql.cn)
  * [How To GraphQL](http://howtographql.cn/)
  * [GraphQL 搭配 Koa 最佳入门实践](https://juejin.im/post/5a49e5ccf265da430d585cfd)

---   
> 发布时间：2018-04-20