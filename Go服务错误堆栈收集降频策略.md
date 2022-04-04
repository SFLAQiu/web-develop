

### 背景：

Go服务在高并发请求下，当服务出现异常，会出现大量的错误日志调用栈跟踪，并上报到UDP日志服务，导致I/O彪高的问题。

多数的情况下，我们只需要通过几条错误日志的分析即可定位问题，也不需要看过多的重复的错误日志，针对这种场景下的问题，对堆栈日志的收集进行时间间隔内限量，超出限量的部分进行概率性的跳过忽略。

当前项目，日志指定的日志级别 >= `error` 之上的级别都需要输出调用堆栈

```go
// 实例化Zap日志对象
logger = zap.New(core).WithOptions(zap.AddCaller()).WithOptions(zap.AddStacktrace(zapcore.ErrorLevel))
```

#### ELK日志：

有调用栈日志：

![](http://blog.thankbabe.com/imgs/log1.png)

![](http://blog.thankbabe.com/imgs/log2.jpg)

无调用栈跟踪

![](http://blog.thankbabe.com/imgs/log3.png)

### 设计：

- 通过计数器进行时间间隔计数
    - 实现封装
- 堆栈日志收集进行概率性跳过策略
    - 通过配置文件配置和启用策略

### 项目：

- core
    - feature-log-limit-test
        - 时间间隔计数器：
            - `/utils/hcounter/timeCounter.go`
        - 日志堆栈跳过策略：
            - `/log/stackSkip.go`

### 相关代码：

`/utils/hcounter/timeCounter.go`

- 计数器计数通过 atomic 原子操作，（原子计数性能比锁高）
- 间隔重置计数器和时间通过 锁 确保并发不会重复重置

```go
package hcounter

import (
	"sync"
	"sync/atomic"
	"time"
)

var mutex sync.Mutex

// TimeCounter 时间间隔计数器
// 计数器计数通过 atomic 原子操作，（原子计数性能比锁高）
// 间隔重置计数器和时间通过 锁 确保并发不会重复重置
//
type TimeCounter struct {
	counter  uint32        // 计数器，从0开始
	abortTs  int64         // 截止时间，当前时间+间隔时间
	Max      uint32        // 默认限制数量，默认：100
	Delta    uint32        // 计数器累加值，默认：1
	Interval time.Duration // 间隔时间，默认：1分钟
}

type TimeCounterOptions func(tl *TimeCounter)

func NewTimeCounter(options ...TimeCounterOptions) *TimeCounter {
	tl := &TimeCounter{}
	// 初始化
	tl.init()
	// 赋能
	for _, opt := range options {
		opt(tl)
	}
	return tl
}

// 初始化，设置默认值
func (tl *TimeCounter) init() {
	if tl.Max <= 0 {
		tl.Max = 100
	}
	if tl.Delta <= 0 {
		tl.Delta = 1
	}
	if tl.Interval <= 0 {
		tl.Interval = 1 * time.Minute
	}
}

// 初始设置新的截止时间
func (tl *TimeCounter) initAbort() {
	var timestamp int64
	if tl.abortTs <= 0 {
		timestamp = time.Now().Add(tl.Interval).Unix()
	} else {
		timestamp = time.Unix(tl.abortTs, 0).Add(tl.Interval).Unix()
	}
	// 更新截止时间
	atomic.StoreInt64(&tl.abortTs, timestamp)
}

// 初始设置计数器数值
func (tl *TimeCounter) initCounter() {
	// 初始化计数器，从0开始
	atomic.StoreUint32(&tl.counter, 0)
}

// 锁操作
func (tl *TimeCounter) doLockHandle(doFn func()) {
	mutex.Lock()
	defer mutex.Unlock()
	doFn()

}

// CheckPass 校验是否通过
func (tl *TimeCounter) CheckPass() bool {
	nowTs := time.Now().Unix()
	// 校验是否过了截止时间
	if nowTs > tl.abortTs {
		// 重置截止时间和计数器 - 使用锁操作
		tl.doLockHandle(func() {
			// 锁里再检验，避免重复设置
			if nowTs <= tl.abortTs {
				return
			}
			// 初始化截止时间
			tl.initCounter()
			// 获取新的截止时间更新
			tl.initAbort()
		})
	}
	// 获取当前计数器值是否超过
	if tl.counter > tl.Max {
		return false
	}
	// 计数器累加
	newCounter := atomic.AddUint32(&tl.counter, tl.Delta)
	// 判断累加计数器是否超过
	if newCounter > tl.Max {
		return false
	}
	return true
}

func WithMax(max uint32) TimeCounterOptions {
	return func(tl *TimeCounter) {
		tl.Max = max
	}
}

func WithDelta(delta uint32) TimeCounterOptions {
	return func(tl *TimeCounter) {
		tl.Delta = delta
	}
}

func WithInterval(interval time.Duration) TimeCounterOptions {
	return func(tl *TimeCounter) {
		tl.Interval = interval
	}
}
```

`/log/stackSkip.go`

- 堆栈日志跳过策略

```go
package log

import (
	ycfg "github.com/olebedev/config"
	"math/rand"
	"time"
)

const MaxNum = 1000

// StackSkip 堆栈日志限制配置
type StackSkip struct {
	Prob           float64 // 跳过概率，小数不能大于1，支持3位小数（可以根据MaxNum修改，这次多位小数）
	CounterMax     int     // 时间间隔，最大允许堆栈日志数量
	IntervalSecond int     // 时间间隔，*/秒
	skipNum        int     // 跳过数值：随机 MaxNum = n ，n<skipNum，就不进行日志堆栈获取
	hc             *hcounter.TimeCounter
}

func NewStackLimitCfg(logCfg *ycfg.Config) *StackSkip {
	if logCfg == nil {
		return nil
	}
	cfg, _ := logCfg.Get("StackSkip")
	if cfg == nil {
		return nil
	}
	// 配置获取
	sl := &StackSkip{}
	sl.Prob = cfg.UFloat64("Prob", 0.5)
	sl.CounterMax = cfg.UInt("CounterMax", 100)
	sl.IntervalSecond = cfg.UInt("IntervalSecond", 60)
	sl.skipNum = int(MaxNum * sl.Prob)
	if sl.Prob < 0 || sl.CounterMax < 0 || sl.IntervalSecond < 0 || sl.skipNum < 0 || sl.skipNum > MaxNum {
		return nil
	}

	// 初始化时间间隔计数器
	sl.hc = hcounter.NewTimeCounter(
		hcounter.WithMax(uint32(sl.CounterMax)),
		hcounter.WithDelta(1),
		hcounter.WithInterval(time.Duration(sl.IntervalSecond)*time.Second),
	)
	return sl
}

// NeedStack 是否需要堆栈日志
func (sl StackSkip) NeedStack() bool {
	// hc=nil 不限制
	if sl.hc == nil {
		return true
	}

	// 是否达到启动概率性策略
	pass := sl.hc.CheckPass()
	if pass {
		return true
	}

	// 使用概率性策略，根据配置概率，概率性略过一些不获取堆栈日志
	i := rand.Intn(MaxNum)
	if i < sl.skipNum {
		return false
	}
	return true
}
```

运用：

```go
func SyncUDPLog(ls LogStruct) {
	// ...

	// 日志堆栈概率性跳过（解决并发下大量错误日志输出堆栈信息导致I/O过高）
	if ls.NeedStack && logStackSkip != nil {
		ls.NeedStack = logStackSkip.NeedStack()
	}

	if !ls.NeedStack {
		// 无堆栈上报
	} else {
		// 有堆栈上报
	}
```

### 项目使用：

增加配置才生效，可以根据需求配置间隔时间，允许最大数量，超出最大数量后，根据配置的概率，跳过忽略掉堆栈日志

config/{{env}}/log.yaml

```yaml
StackSkip:
  #时间间隔，*/秒
  IntervalSecond: 20
  #时间间隔，最大允许堆栈日志数量
  CounterMax: 10
  #跳过概率，小数不能大于1，支持3位小数
  Prob: 0.8
```