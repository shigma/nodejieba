[![Build Status](https://github.com/yanyiwu/nodejieba/actions/workflows/test.yml/badge.svg)](https://github.com/yanyiwu/nodejieba/actions/workflows/test.yml)
[![Financial Contributors on Open Collective](https://opencollective.com/nodejieba/all/badge.svg?label=financial+contributors)](https://opencollective.com/nodejieba) [![Author](https://img.shields.io/badge/author-@yanyiwu-blue.svg?style=flat)](https://github.com/yanyiwu/) 
[![Platform](https://img.shields.io/badge/platform-Linux,macOS,Windows-green.svg?style=flat)](https://github.com/yanyiwu/nodejieba)
[![Performance](https://img.shields.io/badge/performance-excellent-brightgreen.svg?style=flat)](https://github.com/yanyiwu/blog/blob/posts2023archive/_posts/2015-06-14-jieba-series-performance-test.md) 
[![License](https://img.shields.io/badge/license-MIT-yellow.svg?style=flat)](http://yanyiwu.mit-license.org)
[![NpmDownload Status](http://img.shields.io/npm/dm/nodejieba.svg)](https://www.npmjs.org/package/nodejieba)
[![NPM Version](https://img.shields.io/npm/v/nodejieba.svg?style=flat)](https://www.npmjs.org/package/nodejieba)
[![Code Climate](https://codeclimate.com/github/yanyiwu/nodejieba/badges/gpa.svg)](https://codeclimate.com/github/yanyiwu/nodejieba)

- - -

# NodeJieba "结巴"分词的Node.js版本

## 介绍 

`NodeJieba`是"结巴"中文分词的 Node.js 版本实现，
由[CppJieba]提供底层分词算法实现，
是兼具高性能和易用性两者的 Node.js 中文分词组件。

## 特点

+ 词典载入方式灵活，无需配置词典路径也可使用，需要定制自己的词典路径时也可灵活定制。
+ 底层算法实现是C++，性能高效。
+ 支持多种分词算法，各种分词算法见[CppJieba]的README.md介绍。
+ 支持动态补充词库。
+ 支持TypeScript，提供完整的类型定义。

对实现细节感兴趣的请看如下博文：

+ [Node.js的C++扩展初体验之NodeJieba] 
+ [由NodeJieba谈谈Node.js异步实现] 

## 安装

```sh
npm install nodejieba
```

## 快速开始

```js
var nodejieba = require("nodejieba");
var result = nodejieba.cut("南京市长江大桥");
console.log(result);
//["南京市","长江大桥"]
```

更多示例请参考 [demo](https://github.com/yanyiwu/nodejieba-demo)

### 词典载入可灵活配置

如果没有主动调用词典函数时，
则会在第一次调用cut等功能函数时，自动载入默认词典。

如果要主动触发词典载入，则使用以下函数主动触发。

```js
nodejieba.load();
```

以上用法会自动载入所有默认词典，
如果需要载入自己的词典，而不是默认词典。
比如想要载入自己的用户词典，则使用以下函数：

```js
nodejieba.load({
  userDict: './test/testdata/userdict.utf8',
});
```

字典载入函数load的参数项都是可选的，
如果没有对应的项则自动填充默认参数。
所以上面这段代码和下面这代代码是等价的。

```js
nodejieba.load({
  dict: nodejieba.DEFAULT_DICT,
  hmmDict: nodejieba.DEFAULT_HMM_DICT,
  userDict: './test/testdata/userdict.utf8',
  idfDict: nodejieba.DEFAULT_IDF_DICT,
  stopWordDict: nodejieba.DEFAULT_STOP_WORD_DICT,
});
```

#### 词典说明

+ dict: 主词典，带权重和词性标签，建议使用默认词典。
+ hmmDict: 隐式马尔科夫模型，建议使用默认词典。
+ userDict: 用户词典，建议自己根据需要定制。
+ idfDict: 关键词抽取所需的idf信息。
+ stopWordDict: 关键词抽取所需的停用词列表。

## API 文档

### 分词

#### 1. 默认分词

```js
var nodejieba = require("nodejieba");
var result = nodejieba.cut("南京市长江大桥");
console.log(result);
// ["南京市", "长江大桥"]
```

#### 2. 使用HMM模型分词

```js
var result = nodejieba.cutHMM("南京市长江大桥");
console.log(result);
// ["南京市", "长江大桥"]
```

#### 3. 全模式分词

```js
var result = nodejieba.cutAll("南京市长江大桥");
console.log(result);
// ["南京", "南京市", "市长", "长江", "长江大桥", "大桥"]
```

#### 4. 搜索引擎模式分词

```js
var result = nodejieba.cutForSearch("南京市长江大桥");
console.log(result);
// ["南京", "市", "长江", "大桥", "南京市", "长江大桥"]
```

#### 5. 小粒度分词

```js
var result = nodejieba.cutSmall("南京市长江大桥", 3);
console.log(result);
// ["南京市", "长江大桥"]
```

### 词性标注

```js
var nodejieba = require("nodejieba");
var result = nodejieba.tag("红掌拨清波");
console.log(result);
// [ { word: '红掌', tag: 'n' },
//   { word: '拨', tag: 'v' },
//   { word: '清波', tag: 'n' } ]
```

### 关键词提取

```js
var nodejieba = require("nodejieba");
var sentence = "我是拖拉机学院手扶拖拉机专业的。不用多久，我就会升职加薪，当上CEO，走上人生巅峰。";
var result = nodejieba.extract(sentence, 5);
console.log(result);
// [ { word: '升职', weight: 11.739204307083542 },
//   { word: '加薪', weight: 10.8561552143 },
//   { word: 'CEO', weight: 10.642581114 },
//   { word: '手扶拖拉机', weight: 10.0088573539 },
//   { word: '巅峰', weight: 9.49395840471 } ]
```

### TextRank关键词提取

```js
var nodejieba = require("nodejieba");
var sentence = "我是拖拉机学院手扶拖拉机专业的。不用多久，我就会升职加薪，当上CEO，走上人生巅峰。";
var result = nodejieba.textRankExtract(sentence, 5);
console.log(result);
// [ { word: '当上', weight: 1 },
//   { word: '不用', weight: 0.9897190043 },
//   { word: '多久', weight: 0.9897190043 },
//   { word: '加薪', weight: 0.9897190043 },
//   { word: '升职', weight: 0.9897190043 } ]
```

### 添加自定义词语

```js
var nodejieba = require("nodejieba");
console.log(nodejieba.cut("男默女泪"));
// ["男默", "女泪"]
nodejieba.insertWord("男默女泪");
console.log(nodejieba.cut("男默女泪"));
// ["男默女泪"]
```

More Detals in [demo](https://github.com/yanyiwu/nodejieba-demo)

### 关键词抽取

```js
var nodejieba = require("nodejieba");
var topN = 4;
console.log(nodejieba.extract("升职加薪，当上CEO，走上人生巅峰。", topN));
//[ { word: 'CEO', weight: 11.739204307083542 },
//  { word: '升职', weight: 10.8561552143 },
//  { word: '加薪', weight: 10.642581114 },
//  { word: '巅峰', weight: 9.49395840471 } ]

console.log(nodejieba.textRankExtract("升职加薪，当上CEO，走上人生巅峰。", topN));
//[ { word: '当上', weight: 1 },
//  { word: '不用', weight: 0.9898479330698993 },
//  { word: '多久', weight: 0.9851260595435759 },
//  { word: '加薪', weight: 0.9830464899847804 },
//  { word: '升职', weight: 0.9802777682279076 } ]
```

More Detals in [demo](https://github.com/yanyiwu/nodejieba-demo)

## Develop NodeJieba

```sh
git clone --recurse-submodules https://github.com/yanyiwu/nodejieba.git
cd nodejieba
npm install
npm test
```

## 应用

+ 支持中文搜索的 gitbook 插件: [gitbook-plugin-search-pro]
+ 汉字拼音转换工具: [pinyin]

## 性能评测

应该是目前性能最好的 Node.js 中文分词库
详见: [Jieba中文分词系列性能评测]


[由NodeJieba谈谈Node.js异步实现]:https://github.com/yanyiwu/blog/blob/posts2023archive/_posts/2015-03-21-nodejs-asynchronous-insight.md
[Node.js的C++扩展初体验之NodeJieba]:https://github.com/yanyiwu/blog/blob/posts2023archive/_posts/2014-02-22-nodejs-cpp-addon-nodejieba.md
[CppJieba]:https://github.com/yanyiwu/cppjieba.git
[cnpm]:http://cnpmjs.org
[Jieba中文分词]:https://github.com/fxsjy/jieba

[Jieba中文分词系列性能评测]:https://github.com/yanyiwu/blog/blob/posts2023archive/_posts/2015-06-14-jieba-series-performance-test.md
[contributors]:https://github.com/yanyiwu/nodejieba/graphs/contributors
[YanyiWu]:http://github.com/yanyiwu
[gitbook-plugin-search-pro]:https://plugins.gitbook.com/plugin/search-pro
[pinyin]:https://github.com/hotoo/pinyin

## Contributors

### Code Contributors

This project exists thanks to all the people who contribute. [[Contribute](CONTRIBUTING.md)].
<a href="https://github.com/yanyiwu/nodejieba/graphs/contributors"><img src="https://opencollective.com/nodejieba/contributors.svg?width=890&button=false" /></a>

### Financial Contributors

Become a financial contributor and help us sustain our community. [[Contribute](https://opencollective.com/nodejieba/contribute)]

#### Individuals

<a href="https://opencollective.com/nodejieba"><img src="https://opencollective.com/nodejieba/individuals.svg?width=890"></a>

#### Organizations

Support this project with your organization. Your logo will show up here with a link to your website. [[Contribute](https://opencollective.com/nodejieba/contribute)]

<a href="https://opencollective.com/nodejieba/organization/0/website"><img src="https://opencollective.com/nodejieba/organization/0/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/1/website"><img src="https://opencollective.com/nodejieba/organization/1/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/2/website"><img src="https://opencollective.com/nodejieba/organization/2/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/3/website"><img src="https://opencollective.com/nodejieba/organization/3/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/4/website"><img src="https://opencollective.com/nodejieba/organization/4/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/5/website"><img src="https://opencollective.com/nodejieba/organization/5/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/6/website"><img src="https://opencollective.com/nodejieba/organization/6/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/7/website"><img src="https://opencollective.com/nodejieba/organization/7/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/8/website"><img src="https://opencollective.com/nodejieba/organization/8/avatar.svg"></a>
<a href="https://opencollective.com/nodejieba/organization/9/website"><img src="https://opencollective.com/nodejieba/organization/9/avatar.svg"></a>
