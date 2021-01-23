# flink
## 应用场景
- data pipeline 管道
  - 传统 periodic ETL(Extract-transform-laod)
    - 存储系统之间转换数据
  - Data Pipeline
    - 实时事件和存储系统之间持续追加
    - flink->es
- data analytics 报表/大屏
  - 传统 batch analytics
  - streaming analytics
- data driven  风控/预计
  - 传统 transactional application
  - event-driven application
    - trigger
      - action
## 基础概念
- Stream
  - unbound data
  - bound data
    - 有结尾
- state
  - stateless
  - statefull
- time
  - event time
  - ingestion time
  - processing time

