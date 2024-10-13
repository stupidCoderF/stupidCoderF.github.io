# NodeJs 事件循环

NodeJs的异步事件会放入三个不同的队列进行维护

1. Timer 队列：事件循环到达Timer阶段时，会检查setTimeout,setInterview 是否到期，到期则执行回调，未到期则进入Poll阶段
2. Poll 队列
3. Check 队列

Timer 队列主要维护的异步事件回调有：setTimeout，setInterview

Poll 队列主要维护的异步事件回调有：IO操作，文件读写，数据库操作，网络请求

Check 队列主要维护的异步事件回调有：setImmediate

当所有队列中都存在待处理的任务时事件循环会按照 Timer => Poll => Check 的顺序循环执行队列中的任务

## nextTick

每循环一次Timer => Poll => Check称之为一个Tick

process.nextTick 是所有异步任务中最快执行的

process.nextTick 表示该任务在下一次Tick前执行（即追加在本次循环的尾部）

事件循环在空闲的情况（即所有队列均无要执行的事件）下，会暂停在Poll队列等待文件IO完成

## 微任务队列

微任务队列主要包含Promise的回调函数

微任务队列的执行时机在nextTick之后 Tick之前

## 示例

以下代码的执行结果无法保证：

```javascript
setImmediate(() => {
    console.log('immediate');
})
// setTimeout的最小时间为1ms
setTimeout(() => {
    console.log('timeout');
}, 0)
```

原因是

1. 如果主线程执行完同步代码的时间>=1ms时，则在事件循环开始时定时器已经到期，则优先执行定时器回调
2. 如果主线程执行完同步代码的时间<1ms时，则在事件循环开始时定时器未到期，则跳过Timer阶段，Poll队列无任务，直接执行Check队列
