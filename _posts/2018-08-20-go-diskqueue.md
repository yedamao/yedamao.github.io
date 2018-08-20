---
layout: post
title: "go-diskqueue"
---

# go-diskqueue

## 介绍
![](https://f.cloud.github.com/assets/187441/1698990/682fc358-5f76-11e3-9b05-3d5baba67f13.png)

[go-diskqueue](https://github.com/nsqio/go-diskqueue)是nsq内部使用的基于文件的磁盘队列库。nsqd内部topic/channel消息积压超过阀值后，后续进入的消息会被存储在各自的diskQueue中.

## 内部细节

### 元数据

```golang
// diskQueue implements a filesystem backed FIFO queue
type diskQueue struct {
    ...
    // run-time state (also persisted to disk)
    readPos      int64 // 队列读offset
    writePos     int64 // 队列写offset
    readFileNum  int64 // 读取文件序号
    writeFileNum int64 // 写入文件序号
    depth        int64 // 队列长度
    ...
}
```
diskQueue实例化时会先从元数据文件(```"%s.diskqueue.meta.dat"```)读出之前记录的运行时元数据, ioLoop goroutine从元数据记录的位置开始出入队操作.

示例元数据文件内容:

```
3323
0,0
0,348915
```

### 文件消息编码
消息格式:

```
    // parts:   dataLen       data
    // bits:  | - - - - | - - ... - - |
    // len:       4        dataLen
```

### 消息写入文件队列 (Put)

调用方调用Put([]byte)将消息传入writeChan, ioLoop从writeChan读取消息，写入文件队列.

写入过程(writeOne):

```
    // 如果writeFile为空，打开writeFile
    // Seek 至 d.wirtePos
    //           |
    //           V
    // 按编码格式编码消息流
    //           |
    //           V
    //  将消息流写入文件
    //           |
    //           V
    // 更新元信息
    // 如果writePos大于单个文件最大限制,
    // writeFileNum++, 重置writePos.关闭当前文件
```

### 读取文件队列消息 (ReadChan)
diskQueue通过ReadChan()将内部同步阻塞通道readChan暴露给外部使用， 外部程序循环从readChan获取消费队列消息。

ioLoop中读取过程(readOne): 

```
    // 如果readFile为空，打开readFile
    // Seek 至 d.readPos, 设置reader
    //           |
    //           V
    //   按编码格式读取消息流
    //           |
    //           V
    //       更新元信息
    //           |
    //           V
    // 如果nextReadPos大于单个文件最大限制,
    // nextReadFileNum++, 重置nextReadPos.关闭当前文件
```


###  ioLoop goroutine
diskQueue实例化完成后，ioLoop goroutine负责diskQueue内部主要处理逻辑:

```golang
    for {
        // dont sync all the time :)
        // 尝试fsync写入文件
        //           |
        //           |
        //           V
        // 尝试从文件队列读取一条消息 到dataRead,
        // 如果读取到消息 r=d.readChan, 否则r = nil,
        //           |
        //           |
        //           V
        select {
        // the Go channel spec dictates that nil channel operations (read or write)
        // in a select are skipped, we set r to d.readChan only when there is data to read
        case r <- dataRead:
            // 如果没有读到数据, r = nil, select语句会跳过
            // 如果有读到数据，尝试将数据送入readChan, 更新元数据
            count++
            // moveForward sets needSync flag if a file is removed
            d.moveForward()

        case dataWrite := <-d.writeChan:
            // 尝试读取写入文件, 写入文件队列
            count++
            d.writeResponseChan <- d.writeOne(dataWrite)

        case <-syncTicker.C:
            if count == 0 {
                // avoid sync when there's no activity
                continue
            }
            d.needSync = true

        case <-d.emptyChan:
            d.emptyResponseChan <- d.deleteAllFiles()
            count = 0
        case <-d.exitChan:
            goto exit
        }
    }
```
