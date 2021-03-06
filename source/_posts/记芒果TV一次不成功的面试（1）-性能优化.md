---
title: 记芒果TV一次不成功的面试（1）-----性能优化
date: 2019-04-15 21:20:51
tags: [面试经历,性能优化]
categories: 面试
description: 因为朋友内推，运气好有芒果TV的一面机会。但由于一年多没参加面试，自己平时的积累总结又不够，所以回答并不是很完善完整，一面感觉....
---
因为朋友内推，运气好有芒果TV的一面机会。但由于一年多没参加面试，自己平时的积累总结又不够，所以回答并不是很完善完整，一面感觉....

不管怎样，下面针对面试内容，来回顾和总结下相关知识，为接下来的求职面试，做准备吧。

在这次面试中，面试官问到了前端性能优化、跨域、H5、组件编写能力等等，我会分篇记录。

###  性能优化的目的

1. 从用户角度而言，优化能让页面加载更快、用户操作响应更及时、用户使用体验更好。
2. 从服务器角度而言，优化能减少页面请求数、减少请求所占带宽、能够节省资源。

合理的优化能够改善和提升用户体验，节省资源成本。

前端优化的途径有很多，按粒度大致可以分为两类，第一类是页面级别的优化，例如 HTTP请求数、脚本的无阻塞加载、内联脚本的位置优化等 ;第二类则是代码级别的优化，例如 Javascript中的DOM 操作优化、CSS选择符优化、图片优化以及 HTML结构优化等等。另外，本着提高投入产出比的目的，后文提到的各种优化策略大致按照投入产出比从大到小的顺序排列。

#### 一、页面级优化

1. 减少HTTP请求数
   这条策略基本上所有前端人都知道，而且也是最重要最有效的。都说要减少 HTTP请求，那请求多了到底会怎么样呢 ?首先，每个请求都是有成本的，既包含时间成本也包含资源成本。一个完整的请求都需要经过 DNS寻址、与服务器建立连接、发送数据、等待服务器响应、接收数据这样一个 “漫长” 而复杂的过程。时间成本就是用户需要看到或者 “感受” 到这个资源是必须要等待这个过程结束的，资源上由于每个请求都需要携带数据，因此每个请求都需要占用带宽。另外，由于浏览器进行并发请求的请求数是有上限的 (具体参见此处 )，因此请求数多了以后，浏览器需要分批进行请求，因此会增加用户的等待时间，会给用户造成站点速度慢这样一个印象，即使可能用户能看到的第一屏的资源都已经请求完了，但是浏览器的进度条会一直存在。
   减少HTTP请求数的主要途径包括：
   （1）设计层面对界面进行简化
   （2）合理设置HTTP缓存
   （3）资源压缩与合并
   （4）CSS Sprites
2. 将加载的内容在页面信息加载后再加载
3. 异步执行页面脚本
4. Lazy Load Javascript
5. CSS in HEAD
6. 异步请求callback
7. 减少不必要的HTTP跳转
8. 避免重复请求

#### 二、代码级优化

1. javascript
   （1）DOM
   DOM操作应该是脚本中最耗性能的一类操作，例如增加、修改、删除 DOM元素或者对 DOM集合进行操作。如果脚本中包含了大量的 DOM操作则需要注意以下几点：
   a. HTML Collection（HTML收集器，返回的是一个数组内容信息）。在脚本中 document.images、document.forms 、getElementsByTagName()返回的都是 HTMLCollection类型的集合，在平时使用的时候大多将它作为数组来使用，因为它有 length属性，也可以使用索引访问每一个元素。不过在访问性能上则比数组要差很多，原因是这个集合并不是一个静态的结果，它表示的仅仅是一个特定的查询，每次访问该集合时都会重新执行这个查询从而更新查询结果。所谓的 “访问集合” 包括读取集合的 length属性、访问集合中的元素。因此，当你需要遍历 HTML Collection的时候，尽量将它转为数组后再访问，以提高性能。即使不转换为数组，也请尽可能少的访问它，例如在遍历的时候可以将 length属性、成员保存到局部变量后再使用局部变量。
   b. Reflow & Repaint。除了上面一点之外， DOM操作还需要考虑浏览器的 Reflow和Repaint ，因为这些都是需要消耗资源的。
   （2）慎用with。 使用 with相当于增加了作用域链长度。而每次查找作用域链都是要消耗时间的，过长的作用域链会导致查找性能下降。 除非你能肯定在 with代码中只访问 obj中的属性，否则慎用 with，替代的可以使用局部变量缓存需要访问的属性。
   （3）避免使用eval和Function。每次 eval 或 Function 构造函数作用于字符串表示的源代码时，脚本引擎都需要将源代码转换成可执行代码。这是很消耗资源的操作 —— 通常比简单的函数调用慢 100倍以上。
   （4）减少作用域链查找。 如果在循环中需要访问非本作用域下的变量时请在遍历之前用局部变量缓存该变量，并在遍历结束后再重写那个变量，这一点对全局变量尤其重要，因为全局变量处于作用域链的最顶端，访问时的查找次数是最多的。 要减少作用域链查找还应该减少闭包的使用。
   （5）数据访问。 Javascript中的数据访问包括直接量 (字符串、正则表达式 )、变量、对象属性以及数组，其中对直接量和局部变量的访问是最快的，对对象属性以及数组的访问需要更大的开销。当出现以下情况时，建议将数据放入局部变量：
   a. 对任何对象属性的访问超过1次。
   b. 对任何数组成员的访问次数超过1次。
   另外要尽可能减少对象和数组的深度查找。
   （2） CSS选择符。
   （3）图片压缩。
   （4）代码压缩、去注释和多余代码。

#### 三、其他优化

1. CDN
2. Gzip压缩
3. 多域名
