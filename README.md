# 微信公共帐号机器人(Weixin Robot)

A node.js robot for wechat.

[微信公众平台](http://mp.weixin.qq.com/cgi-bin/indexpage?t=wxm-index&lang=zh_CN)提供的[开放信息接口](http://mp.weixin.qq.com/cgi-bin/indexpage?t=wxm-callbackapi-doc&lang=zh_CN)的自动回复系统。

`weixin-robot` 是 [webot](https://github.com/node-webot/webot) 和 [wechat](https://github.com/node-webot/wechat) 的
高级包装。`webot` 负责定义回复规则，`wechat` 负责与微信服务器通信。

功能特色：

1. 方便灵活的规则定义，轻松实现文本匹配流程控制
2. 基于正则表达式的对话设定，配置简单，可以给一句话随机回复不同内容
3. 支持等待后续操作模式，如可以提示用户“需要我执行xxx操作吗？”

## 使用示例：

添加微信帐号，试试效果：

![豆瓣同城微信帐号二维码：douban-event](http://i.imgur.com/ijE19.jpg)
![微信机器人测试帐号：webot-test](http://i.imgur.com/6IcAJgH.jpg)

- webot-test 源码：[webot-example](https://github.com/node-webot/webot-example)
- douban-event 源码：[webot-douban-event](https://github.com/node-webot/webot-douban-event)

## 快速入门 | [FAQ](https://github.com/node-webot/weixin-robot/wiki/FAQ)

```javascript
var express = require('express');
var webot = require('weixin-robot');

var app = express();

// 指定回复消息
webot.set('hi', '你好');

webot.set('subscribe', {
  pattern: function(info) {
    return info.is('event') && info.param.event === 'subscribe';
  },
  handler: function(info) {
    return '欢迎订阅微信机器人';
  }
});

// 接管消息请求
webot.watch(app, { token: 'your1weixin2token' });

// 如果需要多个实例：
webot2 = webot();
webot2.set({
  '/hi/i': 'Hello',
  '/who (are|r) (you|u)/i': 'I\'m a robot.'
});
webot2.watch(app, {
  token: 'your1wechat2token3',
  path: '/wechat_en',
});

// 启动 Web 服务
// 微信后台只允许 80 端口
app.listen(80);

// 如果你不想让 node 应用直接监听 80 端口
// 可以尝试用 nginx 或 apache 自己做一层 proxy
// app.listen(process.env.PORT);
// app.enable('trust proxy');
```

如果一切顺利，你也搭建好了自己的机器人，欢迎到[此项目的 Wiki 页面](https://github.com/node-webot/weixin-robot/wiki/%E4%BD%BF%E7%94%A8%E6%AD%A4%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%BE%AE%E4%BF%A1%E5%B8%90%E5%8F%B7)添加你的帐号。

## 命令行工具

提供可执行文件 `webot` 用于发送测试消息。
使用 `npm` 安装 [webot-cli](https://github.com/node-webot/webot-cli)：

    npm install webot-cli -g

## 自定义菜单

**webot-cli** 提供处理微信自定义菜单的功能，安装好之后执行：

    webot help menu

## weixin-robot 0.3 -> weixin-robot 0.4

weixin-robot 0.4 版本弃用了部分老旧 API ，详见 [History.md](https://github.com/node-webot/weixin-robot/blob/master/History.md)

# API 参考

关于规则定义部分，请参考 [webot](https://github.com/node-webot/webot) 的文档。

## info 对象

webot rule 的 handler 接收到的 info 对象，有一些针对微信的高级属性。

### 星标消息

微信允许你在回复消息时标记一个 `FuncFlag` ，
可以在公共平台后台的「星标消息」中查看带标记的消息。
适合你的机器人不懂如何回复用户消息时使用。
在 `webot` 中，你只需在 handler 中给 `info.flag` 赋值 `true` 即可。

```javascript
// 把这句放到你的规则的最末尾
webot.set('fallback', {
  pattern: /.*/,
  handler: function(info) {
    info.flag = true;
    return ['唔.. 暂时听不懂您说的什么呢',
    '不好意思，我不太懂您说的什么意思',
    '哎呀，听不懂啦！', 
    '这个我不是很懂，不如我们聊点别的吧？']
  }
});
```

### 请求消息属性

你可以通过 `info.original` 拿到与[微信官方文档](http://mp.weixin.qq.com/wiki/index.php?title=%E6%B6%88%E6%81%AF%E6%8E%A5%E5%8F%A3%E6%8C%87%E5%8D%97#.E6.B6.88.E6.81.AF.E6.8E.A8.E9.80.81)中的 xml 一致的属性值：

    ToUserName      开发者微信号
    FromUserName    发送方帐号（一个OpenID）
    CreateTime      消息创建时间 （整型）
    MsgId           消息id
    MsgType         text / image / location / link / event

    // MsgType == text
    Content         文本消息内容

    // MsgType == image
    PicUrl          图片链接

    // MsgType == location
    Location_X      地理位置纬度(lat)
    Location_Y      地理位置经度(lng)
    Scale           地图缩放大小
    Label  地理位置信息

    // MsgType == link
    Title           消息标题
    Description     消息描述
    Url             消息链接

    // MsgType == event
    Event           事件类型，subscribe(订阅)、unsubscribe(取消订阅)、CLICK(自定义菜单点击事件)
    EventKey        事件KEY值，与自定义菜单接口中KEY值对应

    // MsgType == audio
    MediaId         媒体文件的 id
    Format          音频文件的格式

`webot` 的 `info` 把这些值包装为了更符合 js 命名规则的值，并根据 `MsgType` 的不同，
将额外参数存入了 `info.param` 对象。这样做能保证 `info` 对象的标准化，方便你在
不同平台使用相同的机器人。例如，地理位置消息( MsgType === 'location') 会被转化为：

```javascript
{
  uid: 'the_FromUserName',
  sp: 'the_ToUserName',
  id: 'the_MsgId',
  type: 'location',
  param: {
    lat: 'the_Location_X',
    lng: 'the_Location_Y',
    scale: 'the_Scale',
    label: 'the_Label'
  }
}
```

大部分属性值只是把首字母大写换成了小写。

更详细的属性值对应，请查看本模块源码。

### info.reply

你只需在 `rule.handler` 的返回值或 callbak 里提供回复消息的内容，
`webot.watch` 自带的 express 中间件会自动给 `info.reply` 赋值，
并将其打包成 XML 发送给微信服务器。

`info.reply` 支持的数据类型：

- {String}   直接回复文本消息，不能超过2048字节
- {Object}   单条 图文消息/音乐消息
- {Array}    多条图文消息

#### 回复文本消息

```javascript
info.reply = '收到你的消息了，谢谢'
```

#### 回复图文消息

    title        消息标题
    url          消息网址
    description  消息描述
    picUrl       消息图片网址


```javascript
info.reply = {
  title: '消息标题',
  url: 'http://example.com/...',
  picUrl: 'http://example.com/....a.jpg',
  description: '对消息的描述出现在这里',
}

// or

info.reply = [{
  title: '消息1',
  url: 'http://example.com/...',
  picUrl: 'http://example.com/....a.jpg',
  description: '对消息的描述出现在这里',
}, {
  title: '消息2',
  url: 'http://example.com/...',
  picUrl: 'http://example.com/....a.jpg',
  description: '对消息的描述出现在这里',
}]
```

### 回复音乐消息

    title             标题
    description       描述
    musicUrl          音乐链接
    hqMusicUrl        高质量音乐链接，wifi 环境下会优先使用该链接播放音乐

需指定 `reply.type` 为 `'music'`：

```javascript
info.reply = {
  type: 'music',
  title: 'Music 101',
  musicUrl: 'http://....x.mp3',
  hqMusicUrl: 'http://....x.m4a'
}
```

Have fun with wechat, and enjoy being a robot!

## LICENSE

(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
