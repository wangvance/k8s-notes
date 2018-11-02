# Node-Problem-Detector
https://github.com/kubernetes/node-problem-detector
## 架构
### 代码组织
---
- /node-problem-detector
  - /cmd
    - /logcounter
    - /options
    - node-problem-detetor.go
  - /pkg
    - /condition
    - /custompluginmonitor
    - /logcounter
    - /problemclient
    - /problemdetector
    - /systemlogmonitor

### 接口实现
---
#### main
  - NewNodeProblemDetectorOptions
  - NewLogMonitorOrDie
  - NewCustomPluginMonitorOrDie
  - NewClientOrDie
  - NewProblemDetector


#### options 实现
  - AddFlags, 通过 fs 组件实现 flag 解析（比较强大）；
  - 通过 slice 将日志路径保存到 config 数组中；
  - 每个日志路径启动一个 monitor 做监控；

#### 两类 monitor
  - systemlogmonitor
    - 特别关注 tomb 的引进，管理 goroutine 的 lifecycle。
  - custompluginmonitor
    - 与 systemlogmonitor 类似

#### kube-client
  - 如有配置 override；如没有采用 in-cluster
  - 创建 eventRecorder

#### NewProblemDetector 启动
  - 启动各个 monitor
  - groupChannel 将所有 monitor 的 channel 集合起来？？？
  - select channel
    - event： client.Eventf
    - condition: UpdateCondition

#### systemlogmonitor 详解
  - Start
    - logCh: watcher.Watch()
    - monitorLoop
      - if logCh, then parseLog
        - 一旦有新的日志，log monitor 就会将 log 信息与 rules 进行匹配
        - 一旦有匹配的日志，则产生 status
          - generateStatus

**generateStatus**
  - 从 logs 获取 message
  - if event: 填充 types.Temp
  - if condition: 填充 conditions
  - 更新 types.Status，返回
