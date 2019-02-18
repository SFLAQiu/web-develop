
![image](http://blog.thankbabe.com/imgs/php.png?v=1)

### 前言

近期因公司内部转岗，开始参与PHP项目进行后端开发，一直都是强类型写的比较多，弱类型语言也有接触了一些，如：nodejs，python，做一些辅助服务，数据采集的事情，刚好内部有这个机会进行可以学以致用，加上之前对后端的理解和经验，很容易上手，这里记录下开发过程遇到的些问题解决方案和自己对PHP的理解，以及项目中的部分架构

> 当前已经进入PHP7的版本，做了很多的调整，尤其在性能上有很大的提升




### 面向对象

![image](http://blog.thankbabe.com/imgs/pt2.png?v=3)

PHP框架内置很多强大函数，超级全局变量，魔术函数，魔术变量，可以通过提供的内置函数对PHP项目进行拓展，数据类型操作，http信息获取等，通过安装拓展添加各种功能支持，框架内置函数调用大部分还是偏向面向过程，通过调用函数，传入要操作的类型数据和依赖数据，这里刚开始有些不习惯，面向对象的开发中习惯直接 类型变量/对象 点出函数。

现在PHP开发可以选择使用面向过程也可以用面向对象，最早PHP版本不支持面向对象特性，PHP5开始对OOP有良好的支持，很多PHP开发者没有系统性的学习OOP相关知识，包括工龄长的PHP开发者或者老的项目很多还是偏向面向过程开发，所以会接触到很多偏向面向过程开发的项目

在项目开发过程中遇到些偏应用业务开发的项目，看似有用到类，但是并没用到面向对象的特性对业务进行抽象，如：项目中每个业务功能有个php文件对应一个类，类里里大部分都是逻辑function，然后通过拓展autoload，实现自动include php文件，比如通过L函数传入要调用的类名，构造出PHP文件路径，进行include，然后返回类实例对象，只是通过类文件来区分功能函数，并没有使用到面向对象的特性进行封装，还是偏向面向过程思路在开发

PHP5开始对OOP提供了良好支持，基本已经和java，C# 面向对象语法相似，可以使用命名空间，封装interface，abstract，多态：implements，extends，PHP7还支持多继承trait，方便封装些公用的功能，通过PSR4规范，引入composer 实现的autoload，可以很好的进行OOP开发

> PHP开发还是比较灵活，可以面向过程也可以面向对象，根据具体的业务场景设计

使用composer psr4
* 在项目中添加composer.json文件，根据自己需求配置
```
{
  "autoload": {
    "psr-4": {
      "Library\\": "library/"
    }
  }
}
```
* 在composer.json文件所在目录下输入命令，就会自动 download vendor/composer autoload 相关文件
```
composer install
```
* php中的入口index include autoload.php
```
include_once "vendor/autoload.php";
```
* 注意，配置修改，内容变更的时候需要执行
```
composer dump-autoload -o
```
---

### 弱类型问题

##### 编码问题：

在刚学习PHP语法的时候比较不习惯的就是弱类型，不用去定义变量类型，参数类型，返回值类型，对于习惯强类型的童鞋开始会有些不习惯，不定义类型心里怪怪的，总感觉哪里会导致些错误，而且弱类型在编码的过程中IDE不会有类型错误的一些提示，只有在运行的时候报错了才能知道这里错误了，错误提示滞后。尤其是从DB查询数据返回的是一个stdclass/array，获取到的数据没有对应一个实体类，无法知道具体数据有哪些字段，需要通过查询的sql语句，然后通过查看表结构才能知道数据字段信息，这点很难受，影响开发效率

PHP现在已经支持typehint，通过定义类型可以对部分确定的类型变量，参数，返回类型进行强类型的定义，尤其需要定义表数据Model类，这样得到数据对象后通过->可以感知出所有数据字段，方便后续拓展开发和维护
> 根据场景使用，不能说因为自己习惯使用强力型就把所有类型定义都写成强类型

```php
/**
 * Class MJop
 * @property int $id 工作ID
 * @property string $name 工作名字
 * @property int $salary 薪水
 */
class MJop
{
}

/**
 * Class MWorker
 * @property string $name 员工名字
 * @property int $age 年龄
 * @property MJop $jop 工作
 */
class MWorker
{
}

class Worker
{
    /**
     * 获取员工信息
     * @param int $id
     * @return MWorker|stdClass
     */
    public function get(int $id): stdClass
    {
        // mysql select
        return new stdClass();
    }
}

class Logic
{
    /**
     * 获取员工描述
     * @param int $workId
     * @return string
     */
    public function Desc(int $workId): string
    {
        $worker = new Worker();
        $mWorker = $worker->get($workId);
        return '名字：' . $mWorker->name . '，年龄:' . $mWorker->age . '，工作：' + $mWorker->jop->name . '，薪水:' . $mWorker->jop->salary;
    }
}

```
通过定义变量类型得到代码感知
```php
/** @var Logic $logic */
$logic=new Logic();
```

##### 弱类型比较一个头两个大：

因为PHP是弱类型原因，在做类型比较的时候，往往会因为一个不小心就掉坑里，下面列出类型函数和类型比较的表格
> 就问你，看到这些表格怕不怕，心中有一万只草泥马奔腾而过，瞬间变成幽怨的小眼神

![image](http://blog.thankbabe.com/imgs/pt1.png?v=1)

使用 PHP 函数对变量 $x 进行比较

表达式 | gettype() | empty() | is_null() | isset() | boolean : if($x)
---|---|---|---|---|---
$x = ""; | string | TRUE | FALSE | TRUE | FALSE
$x = null; | NULL | TRUE | TRUE | FALSE | FALSE
var $x; | NULL | TRUE | TRUE | FALSE | FALSE
$x is undefined | NULL | TRUE | TRUE | FALSE | FALSE
$x = array(); | array | TRUE | FALSE | TRUE | FALSE
$x = false; | boolean | TRUE | FALSE | TRUE | FALSE
$x = true; | boolean | FALSE | FALSE | TRUE | TRUE
$x = 1; | integer | FALSE | FALSE | TRUE | TRUE
$x = 42; | integer | FALSE | FALSE | TRUE | TRUE
$x = 0; | integer | TRUE | FALSE | TRUE | FALSE
$x = -1; | integer | FALSE | FALSE | TRUE | TRUE
$x = "1"; | string | FALSE | FALSE | TRUE | TRUE
$x = "0"; | string | TRUE | FALSE | TRUE | FALSE
$x = "-1"; | string | FALSE | FALSE | TRUE | TRUE
$x = "php"; | string | FALSE | FALSE | TRUE | TRUE
$x = "true"; | string | FALSE | FALSE | TRUE | TRUE
$x = "false"; | string | FALSE | FALSE | TRUE | TRUE

松散比较 ==

 类型 | TRUE | FALSE | 1 | 0 | -1 | "1" | "0" | "-1" | NULL | array() | "php" | ""
---|---|---|---|---|---|---|---|---|---|---|---|---
TRUE | TRUE | FALSE | TRUE | FALSE | TRUE | TRUE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE
FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | TRUE | TRUE | FALSE | TRUE
1 | TRUE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
0 | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | TRUE | FALSE | TRUE | TRUE
-1 | TRUE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE
"1" | TRUE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
"0" | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE
"-1" | TRUE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE
NULL | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | TRUE | TRUE | FALSE | TRUE
array() | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | TRUE | FALSE | FALSE
"php" | TRUE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE
"" | FALSE | TRUE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | TRUE

严格比较 ===

类型 | TRUE | FALSE | 1 | 0 | -1 | "1" | "0" | "-1" | NULL | array() | "php" | ""
---|---|---|---|---|---|---|---|---|---|---|---|---
TRUE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
1 | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
0 | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
-1 | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
"1" | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE
"0" | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE | FALSE
"-1" | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE | FALSE
NULL | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE | FALSE
array() | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE | FALSE
"php" | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | FALSE
"" | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE

* [参考官方类型比较文档](http://php.net/manual/zh/types.comparisons.php)


刚接触PHP看到这几个表格的时候会有点儿晕，开发的时候需要特别注意下类型比较，对等比较尽量用'==='，一些函数类型已经能够确定不会传递多类型参数，就可以强制类型进行限制，后面熟练度上来再看这个表格就感觉也还好，常用的类型之间的比较已经深深的进入到脑海中


##### 其他问题：
* IDE没办法给定义的变量进行错误提示，因为没有定义类型IDE也不清楚定义变量的类型，没办法做错误提醒，往往需要在运行的时候输出到页面上才能发现问题
* [PHP弱类型引发的漏洞实例](https://www.freebuf.com/articles/web/166543.html)

---

#### PHP灵活性

上面说了这么多弱类型下的问题，这里说下弱类型的优点，弱类型一个明显的优势就是灵活   
PHP动态特性，可以动态实例化，动态添加属性，动态调用函数，等，通过这些特性可以用简单的代码封装出强大的功能

![image](http://blog.thankbabe.com/imgs/pt3.jpg?v=3)

简单举栗子：
```php
<?php

class Developer
{
    public $name;
    public $hair;

    /**
     * 介绍
     */
    public function introduce()
    {
        $desc = '名字：' . $this->name . '，发量：' . $this->hair;
        if (!empty($this->age)) $desc = $desc . '，年龄:' . $this->age;
        echo $desc . "\r\n";
    }

    /**
     * 数据逻辑处理
     * @param $condition
     * @return bool|string
     */
    public function handle($condition)
    {
        if (is_int($condition)) {
            // ... ... 逻辑
            return '数字处理结果';
        } else if (is_string($condition)) {
            // ... ... 逻辑
            return '字符串处理结果';
        } else if (is_array($condition)) {
            // ... ... 逻辑
            return '数组处理结果';
        } else {
            return false;
        }
    }
}

// -----动态添加对象属性-----
$xm = new Developer();
$xm->name = '码圣';
$xm->hair = 80;

// 方式1 - 变量作为属性名
// $fieldAge = 'age';
// $developer->$fieldAge = 20;

// 方式2 - 直接设置属性值
$xm->age = 20;


// -----动态调用对象函数-----

// 变量作为函数名调用
$fn = 'introduce';
if (method_exists($xm, $fn)) $xm->$fn();


// -----动态实例化-----

// 方式1 - 变量作为类名进行实例化
$className = 'Developer';

/** @var Developer $xf */
$xf = new $className();
$xf->name = '小方';
$xf->hair = 30;
$xf->introduce();


// ------属性遍历------
foreach ($xf as $key => $val) {
    echo $key . '=' . $val . "\r\n";
}


// ------参数类型和返回值支持多类型------

$rs = $xf->handle(null);
if ($rs === false) {
    echo '处理失败';
} else {
    echo $rs;
}

// ------函数变量------

$fn = function () {
    echo 'do something';
};

$fn();

```

---

#### 独特特性

##### 内存不常驻

PHP WEB服务端开发，服务器部署多依赖fastcgi进程管理器，static变量和C#包括java生命周期不一样，C#/java 的WEB应用服务进程静态变量是常驻在内存里并且共享，PHP大多使用nginx部署fastcgi进程管理，服务器接收请求的进程是彼此独立的，请求响应完了就回收资源，不存在常驻。

当然PHP也是可以内存常驻的，cli(命令行模式)下内存是常驻，swoole框架开发部署的WEB应用服务也是内存常驻

##### 错误级别

以往在C#开发的时候，执行遇到错误会直接抛出异常,try catch 可以捕获错误异常，出现异常不会继续执行后面的内容，PHP会比较不一样，根据不同的错误级别不一样的执行机制

PHP 有几个错误严重性等级。三个最常见的的信息类型是错误（error）、通知（notice）和警告（warning）。它们有不同的严重性: E_ERROR、E_NOTICE和E_WARNING。错误是运行期间的严重问题，通常是因为代码出错而造成，必须要修正它，否则会使 PHP停止执行。通知是建议性质的信息，是因为程序代码在执行期有可能造成问题，但程序不会停止。 警告是非致命错误，程序执行也不会因此而中止。

PHP 可以控制错误是否在屏幕上显示（开发时比较有用）或隐藏记录日志（适用于正式环境）
更改错误报告行为:
```
# 方式1：配置php.ini
error_reporting=E_ALL &  ~E_NOTICE
```
```php
//方式2：函数调用设置报错级别
error_reporting(E_ALL & ~E_NOTICE);
```
行内错误抑制：
错误控制操作符 @ 来抑制特定的错误。将这个操作符放置在表达式之前，其后的任何错误都不会出现。

```php
<?php echo @$var['sflyq'];
```

php的Error与Exception捕获问题：

Error是检测到的这个问题极有可能使程序无法继续运行，而Exception则是虽然有问题但是程序继续运行不受影响。在php7以前的版本中Error类型是不能被捕获的，仅仅可以捕获Exception类型。php7以后Error与Exception都继承了Throwable接口，使得Error被捕获成为可能，在php7以下的版本也可以捕获Error

* register_shutdown_function 注册一个 callback ，它会在脚本执行完成或者 exit() 后被调用。
* set_error_handler 自己定义的方式来处理运行中的错误
* set_exception_handler 设置默认的异常处理程序，用于没有用 try/catch 块来捕获的异常

##### 连接池

涉及数据库开发过程中一般都会用到连接池，通过使用连接池减少每次需要重新建立连接的时间消耗提高数据操作效率，在高并发业务场景下效果尤为明显，因为目前大部分PHP应用服务都是使用fastcgi的进程管理，每个请求服务器会分配进程去处理，返回结果后进程资源就会自动回收，因为这个因素无法建立连接池

方式1：  
fastcgi模式下目前比较合理的方式就是通过单例模式，保证在当前请求操作下的数据连接只创建一个对象

方式2：   
可以通过swoole拓展实现数据连接池服务，传递sql到服务里执行返回数据，swoole内存常驻，应用客户端连接断开连接池服务进程资源不会自动回收


---
#### 多线程 协程

##### 多线程

线程(thread) 是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一个线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务

* pthread扩展
    * 适用于cli 
    * 单独配置php-cli.ini
* swoole
    * 是一个底层网络库
* gearman
    * 实现了一个 Master-Worker 的模型
    * 分布式任务分发
    * [教程](http://www.cnblogs.com/jkko123/p/6493282.html)
* workerman
    * php 实现的一个网络库 

##### 协程

线程是由操作系统内核进行调度的，我们无法干预，协程是用户态程序，相当于应用程序自己进行了调度。

因为它是用户态程序，所以相当于多个协程会运行在一个线程中。

要注意的是，只有内核对线程的调度才能够利用cpu的多核资源，让程序做到并行，所以在一个线程中的多个协程，是无法做到并行的。

用户态和内核态：   
简单一句话，程序执行时，如果执行的是我们编写的应用程序的代码，这些代码就是运行在用户态的；当代码中调用了系统调用后，接下来内核中的代码就会执行，内核中的代码就是运行在内核态的

* swoole 4.0 全新的协程内核

---

#### 应用服务器架构


服务部署：  
* php7+nginx+php-fpm
* gitlab代码托管，自动发布环境
* ELK日志服务
    * Elasticsearch 搜索引擎
    * Logstash/Filebeat 用户日志处理
    * Kibana 用于对存储在Elasticsearch里的结构化数据做可视化展现
* mysql
    * 根据业务分布式集群 
* redis
    * codis 分布式部署 
* mongodb
* kafaka
    * 日志记录，BI处理

代码托管和测试环境：
> 均使用阿里云服务器，代码托管自建gitlab服务，从开发分支合并到gitlab环境分支自动部署到对应环境服务器上   

* 测试环境 
    * test-app.sflyq.com
    * 测试数据库
    * 公司内网访问
    * gitlab release
* 预发环境
    * yf-app.sflyq.com
    * 生产数据库
    * 公司内网访问
    * gitlab simulation
* 生产环境
    * app.sflyq.com 
    * gitlab master
* 本地测试环境
    * 通过docker部署 

---

#### 最近阶段感悟

![image](http://blog.thankbabe.com/imgs/pt5.jpeg?v=7)

从一个熟悉的语言到另一个相对陌生的语言，语言只是工具，在适合的场景下使用适合的工具，从自己熟悉的业务到陌生的业务，离开自己的舒服区，拥抱变化才能成长

在相同的后端领域切换语言学习成本还是比较低的，主要是对后端开发的思路，经验是可以共用的，只是换了个语言去实现

当公司发展到一定的规模，岗位职能区分的很细，做应用开发的童鞋接触不到服务器架构，没有机会接触职能以外的技术，工作内容除了完成业务需求开发，还是业务需求开发，这样常年开发下去对个人成长的局限性很高，需要自己在工作之余进行拓展，对公司内部有兴趣的技术进行了解和学习，耐心等待机会的到来

在结尾重点说下作为开发应该有的工作态度，感觉大部分开发参与项目普遍责任感和带入感不强，需求过来没有多想，啪啪啪就是一梭子代码，按照产品的逻辑流程码了整个业务功能，功能测试上线可以正常运行没有问题，然后功成身退，两耳不闻天下事，作为开发在参与项目把自己摆在什么样的位置决定这你是什么样的工作态度

从项目角度出发应该把自己所有参与的项目当成自己的孩子，需要主动关注和关心项目的数据情况和后续发展，伴随着孩子成长了，慢慢就有了成就感

从技术角度出发需要把项目需求功能开发当成造房子，需要分析业务需求提供合理的设计方案，适当的抽象和使用设计模式，只有在开发的时候把地基打稳了才能保证后续的维护和拓展，避免技术债

![image](http://blog.thankbabe.com/imgs/pt4.png?v=7)

> 2019年开年第一篇，祝大家和自己新的一年里猪事顺利，大吉大利!