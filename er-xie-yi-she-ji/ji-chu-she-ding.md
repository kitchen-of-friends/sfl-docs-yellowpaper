---
description: 本章节为 M00 - 抽象模块（Abstract Module）的设计内容
icon: tent
---

# 基础设定

基础模块是整个数据协议最核心、最抽象的概念的部分，该模块定义了数据协议的ID类型、标准基础类和全局常量。

### 资源ID

朋厨社交协议中，一条数据称为“资源”，可以是一个用户信息、活动信息、报名信息等。资源的ID是资源的标识，由资源向数据库存储时生成，全局唯一。朋厨社交协议采用 MongoDB 加以实现，因此，朋厨的ID由12字节构成，例如“5f6cf6f6f6f6f6f6f6f6f6f6”。



### 资源状态

一条数据有多种状态，朋厨规定任意资源状态以 int 类型，“state”为名称的字段存储，并且状态采用以下的数值表示对应状态：

<table><thead><tr><th width="176">资源码</th><th>含义</th><th>是否常用</th></tr></thead><tbody><tr><td>1</td><td>用户新创建且正在编辑资源</td><td>是</td></tr><tr><td>100</td><td>资源正在审核中</td><td>是</td></tr><tr><td>200</td><td>审核通过，正常状态</td><td>是，大部分资源都是 200 上线状态</td></tr><tr><td>205</td><td>拼车中的状态，成功转200，不成功转 410</td><td></td></tr><tr><td>210</td><td>激励校验完毕的正常状态，例如用户领取过该资源</td><td></td></tr><tr><td>220</td><td>资源已满员，活动已报名满</td><td></td></tr><tr><td>250</td><td>资源处于“正在进行中”的状态</td><td></td></tr><tr><td>280</td><td>资源已经结束</td><td>是</td></tr><tr><td>300</td><td>用户选择封存资源，前端依旧可见</td><td></td></tr><tr><td>410</td><td>资源过期</td><td></td></tr><tr><td>420</td><td>自动审核失败</td><td>是</td></tr><tr><td>450</td><td>资源处于冻结状态（因举报），前端不可见</td><td>是</td></tr><tr><td>455</td><td>资源处于冻结状态（因管理员下架），前端不可见</td><td></td></tr><tr><td>500</td><td>用户选择永久删除，前端不可见</td><td>是</td></tr></tbody></table>

资源码满足以下逻辑：

* 1XX 都是编辑状态，用户前端不可见，管理前端不可见
* 2XX 都是开放状态，用户前端可见，管理前端可见
* 3XX 都是封存状态，用户前端可见，管理前端可见
* 4XX 都是临时状态，用户前端不可见，管理前端可见
* 5XX 都是永久状态，用户和管理前端不可见，开发后端可见



### 资源属性

#### WithIdAndAudit 有ID、可审计资源（基础类）

该基类让数据拥有ID，且可用记录创建人、修改人、创建时间、修改时间，所有深度社交协议资源都会继承该基类。

<table><thead><tr><th width="196">字段名</th><th>类型</th><th>含义</th></tr></thead><tbody><tr><td>name</td><td>string</td><td>名称</td></tr><tr><td>logo</td><td>string</td><td>logo 存储 url</td></tr><tr><td>content</td><td>string</td><td>内容/描述</td></tr><tr><td>img_list</td><td>[]string</td><td>图片列表</td></tr><tr><td>video_list</td><td>[]string</td><td>视频列表</td></tr></tbody></table>

#### WithDescribe 可描述资源（基础类）&#x20;

该基类让资源具有名字、Logo、描述型文本、图片列表、视频列表，如活动、组织、线上内容等资源都会继承该基类。

| 字段名         | 类型        | 含义          |
| ----------- | --------- | ----------- |
| name        | string    | 名称          |
| logo        | string    | logo 存储 url |
| content     | string    | 内容/描述       |
| img\_list   | \[]string | 图片列表        |
| video\_list | \[]string | 视频列表        |

#### WithLoc 有地理信息的资源（基础类）&#x20;

如一些资源需要带有地理位置信息，则需要继承该资源；该资源支持 “is\_virtual” 属性，代表那条数据无地理信息，当为 true 时，可忽略当条数据的地理请求。

| 字段名                  | 类型         | 含义                 |
| -------------------- | ---------- | ------------------ |
| is\_virtual          | bool       | 是否是线上的             |
| loc\_info.address    | string     | 地址信息               |
| loc\_info.name       | string     | 地点名称               |
| loc\_info.combine    | string     | 上述二者拼接在一起          |
| loc\_geo.type        | string     | 地理位置类型，一般都是“Point” |
| loc\_geo.coordinates | \[]float64 | 两个数值，第一个是经度，第二个是纬度 |

