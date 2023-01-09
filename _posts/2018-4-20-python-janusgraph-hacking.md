---
title:          JanusGraph 踩坑记录
date:           2018-04-20 21:00:00 -0700
categories:     [Database, Practice]
tags:       [database, graph]
comments: true
---

### 使用Python操作JanusGraph踩坑记录

> 最近在折腾图数据库，稍微记录下使用过程中的一些坑。
>
> 常用的图数据库有Neo4j、Titan、OrientDB等。今年年初的时候用过一阵子Neo4j，整个生态圈非常好，但是社区版功能太局限，也不支持分布式的多机部署，用于大型项目比较鸡肋。它的企业版价格每年最低$36,000，太贵了就没考虑。Titan是一个事务型数据库，各方面表现都不错并且是开源的，可定制程度高，支持多个不同的存储后端、索引后端和分析平台，但是已经有3年左右没人维护了，也没有考虑。OrientDB没有用过，据说性能不太行。
>
> 这篇博文里提到的JanusGraph是Titan的升级版，是基于Titan开发的，社区也比较活跃。但大部分文档都是用gremlin来操作数据库的，嵌入到其他项目中或多或少会带来不便。Gremlin是Apache TinkerPop官方的图遍历语言，可以把它理解成图数据库里的SQL，不过语法比较神奇，这里不作具体介绍。本文主要记录用python来操作JanusGraph过程中的一些坑。

* 启动 gremlin-server

    Notes: 不要直接启动bin目录下的gremlin-server.sh，会缺少初始化、elasticsearch和cassandra等配置。

    ```bash
    cd /path_to/janusgraph-0.2.0-hadoop2
    bin/janusgraph.sh start
    ```

* 使用其他语言与Gremlin交互

    - 推荐使用官方的库gremlinpython
    - 注意版本问题，对照https://github.com/JanusGraph/janusgraph/releases/中提到的`Tested Compatibility`安装对应的gremlinpython版本，gremlinpython的版本号就是tinkerpop的版本号。
    因为tinkerpop更新比janusgraph快，不注意的话会导致版本不兼容报错如下：
    ```text
    568695 [gremlin-server-worker-1] WARN  org.apache.tinkerpop.gremlin.server.handler.WsGremlinBinaryRequestDecoder  - Gremlin Server is not configured with a serializer for the requested mime type [application/vnd.gremlin-v3.0+json] - using org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0 by default
    568832 [gremlin-server-worker-1] WARN  org.apache.tinkerpop.gremlin.driver.ser.AbstractGraphSONMessageSerializerV1d0  - Request [PooledUnsafeDirectByteBuf(ridx: 172, widx: 172, cap: 206)] could not be deserialized by org.apache.tinkerpop.gremlin.driver.ser.AbstractGraphSONMessageSerializerV1d0.
    568837 [gremlin-server-worker-1] WARN  org.apache.tinkerpop.gremlin.server.handler.OpSelectorHandler  - Invalid OpProcessor requested [null]
    org.apache.tinkerpop.gremlin.server.op.OpProcessorException: Invalid OpProcessor requested [null]
    ```
    安装对应JanusGraph 0.2.0版本的gremlinpython:

    ```bash
    pip3 install gremlinpython==3.2.6
    ```

* 链接JanusGraph数据库

    ```python
    from gremlin_python.structure.graph import Graph
    from gremlin_python.driver.driver_remote_connection import DriverRemoteConnection
    graph = Graph()
    g = graph.traversal().withRemote(DriverRemoteConnection('ws://localhost:8182/gremlin', 'g'))
    ```

* Vertex

    * 插入

        - 需要注意，gremlinpython所有的函数返回的实际上都是iterator，需要进一步调用next()等RemoteConnection Submission方法来执行具体的操作。而在gremlin标准的console中执行的所有函数可以省略next()等方法因为console会自动对**返回值**进行iterate。

        * 插入一个vertex，拥有三个属性name, attr_name和attr_value

        ```python
        v1 = g.addV().property("name", "wine").property("attr_name", "color").property("attr_value", 'red').next()
        ```

    * 查询

        * 查询name为‘141’绿色工程的vertex的attr_name数值

        ```python
        vertex = g.V().has("name", "wine").values('attr_name').next()
        ```

    * 删除

        Notes: 调用drop方法后再调用next会报错因为无法iterate下去了，需要用其他的submission方式如toList()

        ```python
        res = g.V().drop().toList()	# remove all the vertexes
        res = g.V().properties('name').drop().toList() # remove the property "name" for all vertexes
        ```

* Edge

    * 插入

      * 插入一条从foo连接到bar的edge，该edge的label为knows

      Notes: `GraphTraversalSource`类本身没有addE的方法，需要通过withSideEffect函数来生成两个vertex之间的`GraphTraversal`类实例，然后才能调用addE方法

      ```python
      vFrom = g.addV().property('name', 'foo').next()
      vTo = g.addV().property('name', 'bar').next()
      edge = g.withSideEffect('t', vTo).V(vFrom).addE('knows').to('t').next()
      ```

    * 查询

      * 查询所有的edge，可在调用next方法之前通过hasLabel等方法过滤查询结果

      ```python
      edges = g.E().next()
      ```

    * 删除

      ```python
      res = g.E().drop().toList()	# remove all the edges
      res = g.E().hasLabel('knows').drop().toList() # remove all the edges with label 'knows'
      ```

* Notes
    * 目前查下来缺少标准文档，有些接口是试出来的，更多的例子可以参考https://gist.github.com/okram/f193d5616563a69ad5714a42c504276f。
    * 此外，gremlinpython整体方法调用模式基本和原生的gremlin一致，可以直接参考gremlin的官方文档http://tinkerpop.apache.org/docs/3.2.6/reference/#gremlin-python