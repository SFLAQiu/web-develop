## Redis实战之限制操作频率

最近沉迷于业务开发无法自拔 🤣，有一段时间没有更新博文了，后续博文内容计划把一些业务场景下的实战方案，或者比较好的设计思路进行分享，就不像之前围绕着一个主题，消耗很多的时间去整理相关内容(憋大招)，后续可能一篇的内容量就没那么丰富，但是尽可能针对一个点进行更细化，或者更深入的分析，通过不断分享和自我复盘，进行经验的沉淀，同时提高博文分享的频率 🤙




### 场景

##### 场景1

    留言功能限制，30秒 内只能评论 10次，超出次数不让能再评论，并提示：过于频繁
  
##### 场景2

    点赞功能限制，10秒 内只能点赞 10次，超出次数后不能再点赞，并禁止操作 1个小时，提示：过于频繁，被禁止操作1小时
    
##### 场景3

    上传记录功能，限制一天只能上传 100次，超出次数不让能再上传，并提示：超出今日上线
    
### 抽离本质

在业务开发的过程中，我们不断的参与各种业务场景的方案设计，往往很容易碰到很类似的场景，只不过当前所属的业务模块不一样，其实这些需求的本质是解决同一个问题，当遇到这种场景的时候，我们需要根据自己经验分析抽离出需求的本质问题，实现一个通用的解决方案，让自己的解决方案更有价值，这可能就是区别于你是有灵魂的工程师还是cp（copy paste）最强王者吧。

分析上面3个业务场景，可以从中发现其中有相似的逻辑，称它为同类的问题，现在我们就是要抽离这个问题，设计一个通用的解决方案，勾画相同逻辑流程图：

![流程图](http://blog.thankbabe.com/imgs/redis_limit_flow.png?v=4)


通过分析上面的需求场景，抽离出他们都需要的那些条件：

* 限制对象：用户
* 限制操作（评论，点赞，记录， ...）
* 时间范围X秒内
* 限制操作数Y次
* 超出后禁止操作时间Z（秒/具体时间）
* 超出后不让再操作，并提示

![脑图](http://blog.thankbabe.com/imgs/redis_limit.png?v=3)

> （最小时间单位用秒：天/小时/分钟都可换算成秒，用秒可以解决更多的场景）




如果把功能抽离成一个通用函数是不是大概是这样：

```php
<?php
/**
 * 频率限制
 * @param string $action 操作动作
 * @param int $userId 发起操作的用户ID
 * @param int $time 时间范围X秒内
 * @param int $number 限制操作数Y次
 * @param array $expire 超出封印时间Z ['type'=>1,'ttl'=>过期时间/秒] ['type'=>2,'ttl'=>具体过期时间戳] 二选一
 * @return bool
 * @throws \Exception
 */
public static function frequencyLimit(string $action, int $userId, int $time, int $number, $expire = [])
{
    // todo 根据用户操作动作时间范围，进行频率的控制和失效释放
}

```


### 解决方案落地

功能中需要对用户发起的操作和时间，以及累计次数进行存储，并且需要失效过期的清理，如果这个时候我们依赖mysql做存储，想想都觉的挺痛苦，这里主角：redis 终于登场了，基于redis特性，incr的原子操作和key 支持过期机制，内存存储的效率优势，可以相对简单灵活并且又高效的完成目的。

这里简单实现个通用功能的代码：

```php
<?php
/**
 * 频率限制
 * @param string $action 操作动作
 * @param int $userId 发起操作的用户ID
 * @param int $time 时间范围X秒内
 * @param int $number 限制操作数Y次
 * @param array $expire  超出封印时间Z ['type'=>1,'ttl'=>过期时间/秒] ['type'=>2,'ttl'=>具体过期时间戳] 二选一
 * @return bool
 * @throws \Exception
 */
public function frequencyLimit(string $action, int $userId, int $time, int $number, $expire = [])
{
    if (empty($action) || $userId <= 0 || $time <= 0 || $number <= 0) {
        throw new \Exception('非法参数');
    }
    $key = 'act:limit:' . $action . ':' . $userId;
    $r = RedisClient::connect();
    //获取当前累计次数
    $current = intval($r->get($key));
    if ($current >= $number) return false;
    //累计并返回最新值
    $current = $r->incr($key);
    //第一次累加，设置控制操作频率的有效时间
    if ($current === 1) $r->expire($key, $time);
    //未超出限制次数先放过
    if ($current <= $number) return true;
    //超出后根据需要重新设置过期失效时间 $current === $number 判断保证只重新设置一次
    $type = empty($expire['type']) ? 0 : intval($expire['type']);
    $ttl = empty($expire['ttl']) ? 0 : intval($expire['ttl']);
    if ($current === $number && $ttl > 0 && in_array($type, [1, 2])) {
        if ($type === 1) $r->expire($key, $ttl);
        if ($type === 2) $r->expireAt($key, $ttl);
    }
    return false;
}
//场景1

/**
 * 评论限制
 * @param int $userId
 * @return bool|string
 */
public function doComment(int $userId)
{
    try {
        $pass = FrequencyLimit::doHandle('comment', $userId, 30, 10);
        if (!$pass) return '过于频繁';
        // todo 评论逻辑
        return true;
    } catch (\Exception $e) {
        return $e->getMessage();
    }
}

//场景2
/**
 * 点赞限制
 * @param int $userId
 * @return bool|string
 */
public function doLike(int $userId)
{
    try {
        $pass = FrequencyLimit::doHandle('like', $userId, 10, 10, ['type' => 1, 'ttl' => 1 * 60 * 60]);
        if (!$pass) return '过于频繁，被禁止操作1小时';
        // todo 点赞逻辑
        return true;
    } catch (\Exception $e) {
        return $e->getMessage();
    }
}

//场景3

/**
 * 上传限制
 * @param int $userId
 * @return bool|string
 */
public function doUpload(int $userId)
{
    try {
        $expire = strtotime(date('Y-m-d', strtotime(+1 . 'days')));
        $pass = FrequencyLimit::doHandle('upload', $userId, 1 * 24 * 60 * 60, 100, ['type' => 2, 'ttl' => $expire]);
        if (!$pass) return '超出今日上线';
        // todo 上传逻辑
        return true;
    } catch (\Exception $e) {
        return $e->getMessage();
    }
}

//场景N
```

> 编码上可以根据你设计这个通用方案的复杂度进行进一步抽象，如抽象成频率限制的功能类 等

### 总结
* 对相似的业务场景进行分析，发现本质问题并设计通用的解决方案
* 让解决方案更有价值，做一个有灵魂的开发者
* 熟练掌握redis，充分利用它的特性和优势

---

> 发布时间：2019-06-04