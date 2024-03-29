# chatGPT 接入搜索引擎实战教学（二）eventStream 的接收

# chatGPT tools plus++

<a href="https://greasyfork.org/zh-CN/scripts/456131-chatgpt-tools-plus-cookie%E7%89%88"><img src="https://img.shields.io/badge/GreasyFork-v1.3.2-black.svg" alt="example"></a> <a href="https://github.com/LiWeny16/chatGPT-tool-plus/blob/main/LICENSE"><img src="https://img.shields.io/badge/LICENSE-MIT-pink.svg" alt="example"></a> <a href="https://github.com/LiWeny16/chatGPT-tool-plus"><img src="https://img.shields.io/badge/Link-Github-2.svg" alt="example"></a>
<a href="https://scriptcat.org/script-show-page/756"><img src="https://img.shields.io/badge/ScriptCat-v1.3.2-blue.svg" alt="example"></a>
<a href="https://bigonion.cn"><img src="https://img.shields.io/badge/NameSpace-bigonion.cn-white.svg" alt="example"></a>


## 难度级别

- **草履虫头目**
- ~~大妈~~
- ~~小学生~~
- ~~博士生~~

## 关于我和 zhengbangbo 的故事

<br>

在他注意到我借鉴了他接收 SSE 的方式代码后，他给我发了私信：
![Alt text](../source/zhengbanbo.png)
受到他的启发，我开始继续追寻如何接收 SSE 类型，找来找去也没有关于油猴的资料，就当我`陷入迷茫与绝望的时候`  
我想到了福尔摩斯的一句话

```
排除了所有不可能，剩下的就是真相   --福尔摩斯
```

## 转机

这也就是说，油猴一般只会用 GM_xmlhttpRequest 函数来跨域，而这里必须跨域请求，那么，再去仔细看看 GM_xmlhttpRequest 的参数说不定会有效果，于是我又看了几遍 details 参数，欸，等等，这是什么？

- onloadstart callback to be executed on load start, provides access to the stream object if responseType is set to "stream"

这不就是 stream 类型吗，我试试他返回了什么数据类型吧，于是在

- responseType one of arraybuffer, blob, json or stream

这不就有 stream 类型吗！<br>
responseType 里面设置为 stream ，onloadstart 里面打印返回的结果
<br>![Alt text](../source/response.png)  
是 Readablestream 类型，等等，我记得之前在看到 fetch 接收的时候也见过这个类型
[https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream/ReadableStream](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream/ReadableStream)<br>
在我仔细的翻找之下，终于找到了 stream 的构造以及接收方法，以下是接收方法

```js
        onloadstart:(stream)=>{
            const reader = stream.response.getReader();
            let charsReceived = 0;
            reader.read().then(function processText({ done, value }) {
                if (done) {
                    return;
                }
                charsReceived += value.length;
                console.log(value)
                return reader.read().then(processText);
            });
        },
        responseType:"stream",
```

以下是具体解释：<br>

- Readablestream 类型带有一个 getReader 方法，返回一个
- ReadableStreamDefaultReader 类型  
  ![Alt text](../source/reada.png)<br>
  ![Alt text](../source/readableSTream.png)<br>
  <br>

- 同时这个类型带有一个重要的 read 方法<br>
  <br>
  <br>

![Alt text](../source/readablemethod.png)<br>
<br>

- read()方法返回一个 promise<br>
  promise.then 方法中就带着 done 和 value 参数  
  done 表示传输是否结束，结束为 1，没结束为 0  
  value 则表示每一次流传输的数据  
  charsReceived 代表已经接收的数据长度  
  接着 processText 最后 return 自己，即 reader.read().then(processText)  
  形成循环递归调用  
  最后我们输出以下 value 来看看结果吧！
  <br>
  <br>

我超，真的拿到了数据!但是怎么是一堆**不停打印的**Uint8Array 数组啊
![Alt text](../source/values.png)

难道是？

## 关于

作者：Onion  
邮箱：bigonion@bigonion.cn  
声明：未经本人同意，禁止转载、搬运、抄袭！

NameSpace: https://bigonion.cn  
Origin:https://bigonion.cn/blog

## (二)完结
