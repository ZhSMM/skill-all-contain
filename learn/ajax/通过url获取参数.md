# 通过url获取参数

### window.location对象

- window.location.href：整个URl字符串(在浏览器中就是完整的地址栏)
- window.location.protocol：URL 的协议部分   返回值：http:
- window.location.host：URL 的主机部分(带端口号)
- window.location.port：URL 的端口部分
- window.location.pathname：URL 的路径部分(就是文件地址)
- window.location.search：查询(参数)部分。得到的是url中？部分。除了给动态语言赋值以外，我们同样可以给静态页面,并使用javascript来获得相应的参数值
- window.location.hash：锚点。得到的是url中#部分。

### 函数

```javascript
//获取URL参数
getUrlParam:function(name){
  var reg=new RegExp('(^|&)'+name+'=([^&]*)(&|$)');
  var result=window.location.search.substr(1).match(reg);
  return result ? decodeURIComponent(result[2]):null;
}
```

注释：ECMAScript v3 已从标准中删除了 unescape() 函数，并反对使用它，因此应该用 decodeURI() 和 decodeURIComponent() 取而代之。
说明：

1. reg是正则表达式子：
   - `^`：匹配输入字符串开始的位置，如果设置了RegExp对象的Multiline属性，^还会与 \n 或 \r 之后的位置匹配；
   - `$`：匹配输入字符串结尾的位置，如果设置了RegExp对象的Multiline属性，^还会与 \n 或 \r 之前后的位置匹配；
   - `\b`：匹配一个字符边界，即字与空格之间的位置；
   - `\B`：非字符边界匹配；
2. location是包含了相关的url的信息，它是windown的一部分。
3. search是一个可以查询的属性，可以查询？之后的部分。
4. match()匹配函数
5. return unescpe（r[2]） 返回的值  一个数组或者null
6. decodeURIComponent() 函数可对 encodeURIComponent() 函数编码的 URI 进行解码。

### 调用

console.log(getUrlParam('test'));

### 参考

- [获取url参数window.location.search.substr(1).match(reg)](https://blog.csdn.net/h_duolalu/article/details/78302328)
- [window.location.search.substr(1)方法](https://www.jianshu.com/p/f988e4ebd627)