## About Async functions in JavaScriprt
今天必须记录一下......为了这个异步有关的事情真的花了太多时间了。我觉得问题是JavaScript我一直没有系统学过，和C++什么的上手感觉完全不一样。看来有必要对着书深入地学习一下。这个后面再谈，今天先记录一下学会的方法。
## How to load JSON file in JavaScript
任爹上课和我们说，JSON是可以用来模拟后端数据的。上次给任爹看了[HWnan](https://github.com/llIllIllIlllIll/SE100-HWnan) 的测试什么的，任爹建议我加一个这样子的东西。所以今天第一件事是在研究怎么样才可以导入一个JSON文件。参考了Stack Overflow上一个关于导入local JSON文件的帖子，惊喜地发现了一个网站可以用来寄存JSON文件——寄存了之后就可以非常方便地通过https request来导入。这个网站非常非常非常nice，[Myjson.com](http://myjson.com/)。简直造福人类，以后我闲得没事干了我也要做这种好事儿。
##
当你把自己的JSON文件上传之后，你就可以用类似这样的方法进行导入：
```javascript
let response = await fetch('https://api.myjson.com/bins/17yr1m');
data =await response.json();

//或者这样也可以
$.getJSON("https://api.myjson.com/bins/17yr1m", function(json) {
    for(var i=0;i<json.existed_accounts.length;i++)
        {
            data[i]=json.existed_accounts[i];
        }});
```
要论方便明显还是上面那个方便，就一股脑儿什么都不管往你的一个var data里面塞。但是要用上面那个东西，还得了解一些异步相关的东西，这些东西简直就是我今晚时间的黑洞。
## How to write Async Functions and how to use await and how to test in Jasmine
这两个问题，简单地单纯从语法角度来说就是await一般是用来wait一个async function的结果，然后await必须要用在一个async function里面。

[我学习这个看的网站](https://javascript.info/async-await)

上面这个网站可以参考。我发觉这个问题是因为，在测试的时候，我发现console里面我的data是没有问题的（成功从服务器加载了），但是测试的结果里面显示我的data并没有加载成功。这个就是因为，JavaScript的运行并不是严格线性的，如果在我的data fetch成功之前进行了测试，那么毫无疑问测试是会失败的。Jasmine的报错信息就会告诉我， property undefined。
当然我发觉这点之后我就把loadData相关的函数改成了异步函数，但还是不行。

那么这个时候我就觉得会是Jasmine的问题，可能是Jasmine对于Async会有冲突这样。然后发现Jasmine有自己的异步处理的方法beforeAll，但是仍然需要和async这些结合起来用。现在我找到的一个合理的解决方案是这样的：（以下代码片段需要加在响应describe函数的开头）
```javascript
beforeAll(async function () { 
            let response = await fetch
            ('https://api.myjson.com/bins/17yr1m');
            data =await response.json();
            }
            
        );
```