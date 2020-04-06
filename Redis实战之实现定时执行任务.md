## Redis实战之实现定时执行任务

#### 需求
* 异步执行任务
* 支持定时执行
* 支持取消任务
* 保障快速执行

#### 技术背景
* 基于redis实现
* php


#### 实现

基于redis的 sorted set + hash，实现定时执行任务的Demo

**sorted set 介绍**： 
*  redis有序集合，且不允许重复的成员，不同的是每个元素都会关联一个double类型的分数
*  redis正是通过分数来为集合中的成员进行从小到大的排序，有序集合的成员是唯一的,但分数(score)却可以重复
 
**思路**：
* 使用sortset类型，将member[成员] =【任务标识】，score[分数] =【定时时间戳】
* 使用hash类型，将【任务标识】对应的任务数据JSON存到hash中 key =【任务标识】，value =【任务数据JSON】
* 解决及时消耗，可以运行多个进程进行并行执行

![Before VS After](http://blog.thankbabe.com/imgs/yield_task.png?v=666)

php实现代码DEMO
```php
<?php
class DoTest
{
    private const YIELD_KEY = 'yield:list';
    private const YIELD_DATA_KEY = 'yield:data';

    public function run()
    {
        $bbj = RedisClient::instance()->bbj();
        //获取排序（低到高）中第一个task_id
        $data = $bbj->zRange(self::YIELD_KEY, 0, 0, true);
        if (empty($data)) {
            echo "无数据" . PHP_EOL;
            return null;
        }
        $mem = array_keys($data)[0];
        $ts = array_values($data)[0];
        $now = time();
        //校验是否到时
        if ($ts > $now) {
            echo "还未到时间，无需操作" . PHP_EOL;;
            return null;
        }
        //移除集合，多进程并执行到时任务(只能被成功移除一次)
        $row = $bbj->zRem(self::YIELD_KEY, $mem);
        if (empty($row)) {
            echo "已经被剔除" . PHP_EOL;;
            return false;
        }
        //获取当前要执行的任务数据JSON
        $dataJson = $bbj->hGet(self::YIELD_DATA_KEY, $mem);
        //todo 执行定时任务业务逻辑
        var_export($data);
        var_export($dataJson);
        //使用完后删除任务数据JSON
        $bbj->hdel(self::YIELD_DATA_KEY, $mem);
        return true;
    }

    public function add(int $time, $i, string $content)
    {
        $data = [
            'msg' => $content . $time
        ];
        $bbj = RedisClient::instance()->bbj();
        $dataJson = json_encode($data);
        $taskId = $time . '_' . $i;
        $isSc = $bbj->zAdd(self::YIELD_KEY, $time, $taskId);
        if ($isSc) $isSc = $bbj->hSet(self::YIELD_DATA_KEY, $taskId, $dataJson);
        var_export($isSc);
    }

    public function addFeature($i)
    {
        $time = Carbon::now()->addMinute()->timestamp;
        $this->add($time, $i, '未来执行内容');
    }

    public function addCurrent($i)
    {
        $time = time();
        $this->add($time, $i, '马上执行内容');
    }
    
    /**
     * 取消定时任务，根据任务ID
     * @param $taskId
     */
    public function removeYield($taskId)
    {
        $bbj = RedisClient::instance()->bbj();
        $bbj->zRem(self::YIELD_KEY, $taskId);
        $bbj->hDel(self::YIELD_DATA_KEY, $taskId);
    }
}

```
监控定时任务队列
```php
<?php
$dt = new DoTest();
while (true) {
    $rt = $dt->run();
    if (is_null($rt)) {
        sleep(1);
    }
}

```

添加当前执行和未来执行任务
```php
<?php
$dt = new DoTest();
while (true) {
     for ($i = 0; $i < 100000; $i++) {
        // 添加立即执行当前任务
        $dt->addCurrent($i);
        // 添加待执行未来任务
        $dt->addFeature($i);
    }
}

```
取消定时任务
```php
<?php
$dt = new DoTest();
$dt->removeYield('taskId');
```


#### 解决场景
* 定时短信发送/email发送
* 定时执行??任务

> 发布时间：2019-08-08