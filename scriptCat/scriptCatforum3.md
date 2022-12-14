# chatGPT 接入搜索引擎实战教学（三）eventStream的数据处理
# chatGPT tools plus++
<a href="https://greasyfork.org/zh-CN/scripts/456131-chatgpt-tools-plus-cookie%E7%89%88"><img src="https://img.shields.io/badge/GreasyFork-v1.3.2-black.svg" alt="example"></a>  <a href="https://github.com/LiWeny16/chatGPT-tool-plus/blob/main/LICENSE"><img src="https://img.shields.io/badge/LICENSE-MIT-pink.svg" alt="example"></a>  <a href="https://github.com/LiWeny16/chatGPT-tool-plus"><img src="https://img.shields.io/badge/Link-Github-2.svg" alt="example"></a>
  <a href="https://scriptcat.org/script-show-page/756"><img src="https://img.shields.io/badge/ScriptCat-v1.3.2-blue.svg" alt="example"></a>


## 难度级别
+ **草履虫**
+ ~~大妈~~
+ ~~小学生~~
+ ~~博士生~~

## 那些Uint8Array数组是什么？
上回说到，看到一堆不停打印的Uint8Array数组，em？这都是啥？  
![Alt text](../source/values.png)  <br>
难道是？  
看到这些数字基本没有超过120的，我心里有了一个答案，也许是ASCII码？  
此时对其中一个数组点击展开   
然后，只需要点击这个小图标来到内存查看器  
![Alt text](../source/Inkedarrayexpended_LI.jpg)<br>  
![Alt text](../source/resultInunicode.png)<br>  
真的看到解码后的数据了耶！  
让我们来看看他到底经历了什么变换吧！  
首先我们的原始数据为  
```
[100, 97, 116, 97, 58, 32, 123, 34......]
```
对其每一项十进制转十六进制可以得到  
```
[64,61,74,61,3A,20,7B,22]
``` 
|dec|hex|character|
|---|---|---|
|100|64 | d|
|97|61  | a|
|116|74 | t|
|97|61  | a|
|......|......|......|
|||

这不就轻轻松松拿到了真正的数据了吗，只需要把所有的数组里面的数字都转成ASCII字符，我们只需要打表，循环转换，然后...那当然是不可能滴，js中有现成的转换ASCII函数:[String.fromCharCode()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode)  
只需要丢给他十进制数字，他就能帮你转成ASCII字符，例如:  
```js
String.fromCharCode(103,103,110,98)
```
返回什么大家可以自己试试看  
什么？你不知道ASCII码？[点我了解ASCII码](https://www.zhihu.com/question/419185359)  
就当我以为胜券在握的时候，没想到直接往里面丢数组居然报错了
```
String.fromCharCode([1,2,3,4,5])
```
这样居然不可以！
我也想过丢字符串去掉"[""]"进去，但是发现这个函数只能接收1,2,3,4,5这种形式，最后还是Google到了解决方法:
String.fromCharCode.apply(null,Array)  
这个我在MDN也没找到解释，但是真的能用。。。
```js
String.fromCharCode.apply(null,[100,97,100,97]) //大家自个试试
```
接下来我们看看转成字符后的输出吧！<br>
![Alt text](../source/dataFormed.png)<br>

那么再接下来的事情就很清晰明朗了，对所有返回的8bitarray转成ASCII字符后，接着想办法拿到真正的回复内容，回复内容是parts里面的东西，一看就知道用的是Unicode编码，这个js中也有现成的函数。  
什么,还不知道什么是Unicode编码？[点我，了解Unicode和utf-8的区别与关系](https://www.cnblogs.com/xiaoran991/p/12497238.html)
```js
unescape("\u8f6c\u6362") //输出：转换
```

好的我们已经拿到parts里面的数据了
我比较喜欢用正则匹配，大家也可以各显神通，什么Slice，什么Split啊包括JSON.parse转成JSON对象再调用啊都可以，下面就只用正则匹配的方式来阐述。    
什么？你还不知道什么是正则表达式？[点我，十分钟了解正则！](https://www.bilibili.com/video/BV1da4y1p7iZ/?spm_id_from=333.337.search-card.all.click&vd_source=347109678632e4593a175ba64105c5ff)    
下面是示例代码
```js
const chunk = value;
result += chunk;
normalArray = Array.from(chunk) //没有这一步也可以
nowResult = String.fromCharCode.apply(null, normalArray).match(/(?<=\[).*(?=\])/g)[0] //取出真正的回答内容

if (nowResult !== "DONE") {
finalResult = nowResult
}

document.getElementById('gptAnswer').innerHTML = mdConverter(decodeUnicode(finalResult.replace(/\\n+/g, "\n")))

function decodeUnicode(str) {
    str = str.replace(/\\/g, "%");
    str = unescape(str);
    str = str.replace(/%/g, "\\");
    str = str.replace(/\\/g, "");
    return str;
}
```
finalResult取出来之后，观察发现带有很多形如\nn的换行符，所以需要稍微修剪一下把他们统一换成\n,最后用Unicode解码，然后用第三方markdown转HTML的解析js库来转换，最后改改样式，呈现到浏览器页面上！  
![Alt text](../source/sample.png)<br>

## 杂谈
GM_xmlhttprequest提供的fetch方式难道不可以吗？答案是，可以，但是返回的也是一个ReadableStream对象，和XMLHttpRequest在这里的处理方式基本没有区别，也是要在onloadstart对这个对象进行读取处理，并且fetch有一个问题，即不能对其进行abort也不支持超时控制，所以在这里，XHR是更为优质的选择  

## 后记
找到解决方法后，和zbbgg分享了如何解决的方法,也是荣幸出现在致谢表里了😁<br>![Alt text](../source/%E8%87%B4%E8%B0%A2.png)

<br>
<br>
<br>

## (三)完结
