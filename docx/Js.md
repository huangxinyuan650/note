Promise
标准使用示例
promise状态一旦从pending变为resolve或者reject就无法再被改变
promise支持多级使用即一个then中可以返回一个promise对象继续完成下一个promise中的then方法
catch方法捕获promise构造方法中执行的异常，可以作为reject方法来使用
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();
    function handler() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });
  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});


Let 和const都是块级变量，const是不可变变量指的是变量对应的内存，即变量可以添加属性

var，function定义的是全局变量作为window的属性，let，const，class定义的全局变量不作为window的属性

map，filter
map为对数组进行处理映射，filter为对数组进行过滤，返回满足条件的元素形成一个新的数组
S = [1,2,3,4]
S.map(function(item){
    Return item*2;
})
S.filter(function(item){
    If (item >2){
        Return item;
    }
})

获取页面某个标签$(‘#tag_id’)/$(‘.tag_class’).     .attr()增加属性

数组追加，push

Json文件转成字符串：
使用js进行转化，在浏览器的控制台中将json赋给一个变量a，然后将该变量通过b=JSON.stringify(a)，然后打印变量b即可得到字符串

JS字符串替换
Str.replace(“oldStr”,”newStr”)


ES6语法
…为将一个数组转换为一个用逗号分隔的参数序列
快速生成数组方式
方式一：[…Array(100).keys()]
方式二：Array(100).fill(‘1’).map(function(_v,_i){return _i})
方式三：Array.from(new Array(100),function(_v,_i){return _i})