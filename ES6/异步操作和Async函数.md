## 基本概念
### 异步
所谓“异步”， 简单说就是一个任务分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。比如，有一个任务是读取文件进行处理，任务的第一段是向操作系统发出请求，要求读取文件。然后，程序执行其它任务，等操作系统返回文件，在接着执行任务的第二段（处理文件。这种不连续的执行，就叫做异步。

相应地，连续的执行就叫做同步。由于是连续执行，不能插入其他任务，，所以操作系统从硬盘读取文件的这段时间，程序只能干等着。

### 回调函数
JavaScript语言对异步编程的实现，就是回调函数。所谓回调函数，就是把任务的第二段单独写在一函数里面，等动重新执行这个任务的时候，就直接调用这个函数。
读取文件进行处理，是这样写的
``` js
fs.readFile('etc/passwd', function (err, data) {
	if (err) throw err;
	console.log(data)
})
```
上面代码中，readFile函数的第二个参数，就是回调函数，也就是任务的第二段。等到操作系统返回了/etc/passwd这个文件以后，回调函数才会执行。
一个有趣的问题是，为什么Node.js约定，回调函数的第一个参数，必须是错误对象err（如果没有错误，该参数就是null）？原因是执行分成两段，在这两段之间抛出的错误，程序无法捕捉，只能当作参数，传入第二段。

### Promise
回调函数本身并没有问题，它的问题出现在多个回调函数嵌套。假定读取A文件之后，再读取B文件
``` js
fs.readFile(fileA, function (err, data) {
	fs.readFile(fileBl, function (err, data) {

	});
});
```
不难想象，如果依次读取多个文件，就会出现多重嵌套。代码不是纵向发展，而是横向发展，很快会乱成一团，无法管理。这种情况称为（callback hell）
Promise就是为了解决这个问题而提出的。它不是新的语法功能，而是一种新的写法，允许将回调函数的横向加载，改成纵向加载。采用Promise，连续读取多个文件，写法如下。

``` js
var readFile =  require('fs-readfile-promise');

readFile(fileA)
.then(function(data) {
	console.log(data.toString());
})
.then(function() {
	return readFile(fileB);
})
.then(function(data) {
	console.log(data.toString())
})
.catch(function(err) {
	console.log(err);
});

```
上面代码中，使用了fs-readfile-promise模块，它的作用就是返回Promise版本的readFile函数。Promise提供then方法加载回调函数，catch方法捕捉执行过程中抛出的错误。
可以看到，Promise的写法只是回调函数的改进，使用then方法，异步任务的两段看得更清楚了，除此以外，并无新意。

Promise的最大问题是代码冗余，原来的任务被Promise包装了一下，不管什么操作，一眼看去都是一堆then，原来的语义变得很不清楚

## Generator函数
### 协程
传统的编程语言，早有异步编程的解决方案（其实是多任务的解决方案）。其中有一种叫做“协程”（coroutine）。意思是多个线程相互协作，完成异步任务。
协程有点像函数，又有点像线程。它的运行流程大致如下
第一步：协程A开始执行
第二步：协程A执行到一半，进入暂停，执行权转移到协程B。
第三步：（一段时间后）协程B交还执行权
第四步：协程A恢复执行。

上面流程的协程A，就是异步任务，因为它分成两段（或多段）执行。
``` js
function *asnycJob() {
	var f = yield readFile(fileA);
}
```
上面代码的函数 asyncJob是一个协程，它的奥秘就在其中的yield命令。它表示执行到此处，将执行权交给其它协程。也就是说， yield命令是异步两个阶段的分界线。
协程遇到yield命令就暂停执行，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点，就是代码的写法非常像同步操作，如果去除yield命令，简直一模一样。

## Generator函数的概念
Generator函数是协程在ES6的实现，最大的特点就是可以交出函数的执行权（即暂停执行）。
整个函数Generator函数就是一个封装的异步任务，或者说是异步任务的容器。异步操作需要暂停的地方，都用yield语句注明。
Generator函数的执行方法如下
``` js
function* gen(x) {
	var y = yield x + 2;
	return y;
}
var g = gen(1);
g.next() // { value: 3, done: false}
g.next() // { value: undefined, done: true }
```
上面代码中，调用Generator函数，会返回一个内部指针（即遍历器）g，这是Generator函数不同于普通函数的另一个地方，即执行它不会返回结果，返回的是指针对象。调用指针g的next方法，会移动内部指针（即执行异步任务的第一段），指向第一个遇到的yield语句，上例是x + 2为止。

换言之，next方法的作用是分段执行Generator函数。没次调用next方法，会返回一个对象，表示当前阶段的信息（value属性和done属性）。value属性是yield语句后面表达式的值，表示当前阶段的值；done属性是一个布尔值，表示Generator函数是否执行完毕，即是否还有下一个阶段。

Generator函数的数据交换和错误处理
Generator函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。除此之外，它还有两个特性，使它可以作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制
next方法返回的value属性，是Generator函数向外输出数据；next方法还可以接受参数，这是向Generator函数体内输入数据。

``` js
function* gen(x) {
	var y = yield x + 2;
	return y;
}

var g = gen(1);
g.next() // {value: 3, done: false }
g.next(2) // { value: 2, done: true }
```
上面代码中，第一个next方法的value属性，返回表达式x + 2 的值（3）。第二个next方法带有参数2，这个参数可以传入Generator函数，作为上个阶段异步任务的返回结果，被函数体内的变量y接收。因此，这一步的value属性，返回的就是2（变量y的值）
Generator函数内部还可以部署错误处理代码，捕获函数体抛出的错误。
``` js
function * gen(x) {
	try {
		var y = yield x + 2;
	} catch (e) {
		console.log(e)
	}
	return y;
}
var g = gen(1);
g.next();
g.throw('出错了');

```
上面代码的最后一行，Generator函数体外，使用指针对象的throw方法抛出的错误，可以被函数体内的try...catch代码块捕获。这意味着，出错的代码与处理错误的代码，实现了时间和空间上的分离，这对于异步编程无疑是很重要的。

### 异步任务的封装
下面看看如何使用Generator函数，执行一个真实的异步任务
``` js
var fetch = require('node-fetch');

function* gen() {
	var url = 'https://api.github.com/users/github';
	var result = yield fetch(url);
	console.log(result.bio);
}
```
上面代码中，Generator函数封装了一个异步操作，该操作先读取一个远程接口，然后从JSON格式的数据解析信息。就像前面说过的，这段代码非常像同步操作，除了加上了yield命令.
执行这段代码的方法如下
``` js
var g = gen();
var result = g.next();

result.value.then(function(data) {
	return data.json();
}).then(function(data) {
	g.next(data);
});
```
上面代码中，首先执行Generator函数，获取遍历器对象，然后使用next方法，执行异步任务的第一阶段。由于Fetch模块返回的是一个Promise对象，因此要用then方法调用下一个next方法。
可以看到，虽然Generator函数将异步操作表示得很简洁，但是流程管理却很不方便（即何时执行第一阶段，何时执行第二阶段）。


## Thunk函数
### 参数的求值策略
Thunk函数早在上个世纪60年代就诞生了
那时，编程语言刚刚起步，计算机学家还在研究，编译器怎么写比较好。一个争论的焦点是“求值策略”，即函数的参数到底该何时求值。

``` js
var x = 1;

function f(m) {
	return m * 2;
}
f(x + 5)
```
上面代码先定义函数f，然后向它传入表达式 x + 5。请问，这个表达式该何时求值？
一种意见是“传值调用”（call by value），即在进入函数体之前，计算x + 5的值（等于6），再将这个值传入函数f，C语言就采用这种策略

另一种意见是“传名调用“（call by name), 即直接将表达式x + 5传入函数体，只在用到它的时候求值。Haskell语言采用这种策略。
传值调用和传名调用，哪一种比较好？回答是各有利弊。传值调用比较简单，但是对参数求值的时候，实际上还没有用到这个参数，有可能造成性能损失

### Thunk函数的含义
编译器的“传名调用”实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数

``` js
function f(m) {
	return x * 2;
}
 f(x + 5);

 //===
 var thunk = function() {
	 return x + 5;
 };
 function f(thunk) {
	 return thunk() * 2;
 }
```
 ###JavaScript语言的Thunk函数

 JavaScript语言是传值调用，它的Thunk函数含义有所不同。在JavaScript语言中，Thunk函数替换的不是表达式，而是多参数函数，将其替换成单参数的版本，且只接受回调函数作为参数。

 ``` js
 fs.readFile(fileName, callback);

 // Thunk版本的readFile（单参数版本)
```

### async函数
含义
ES7提供了async函数，使得异步操作变得更加方便。async函数是什么？一句话，async函数就是Generator函数的语法糖。
前文有一个Generator函数，依次读取两个文件
``` js
var fs = require('fs');

var readFile = function (fileName) {
	return new Promise(function (resolve, reject) {
		fs.readFile(fileName, function(error, data) {
			if (error) reject(error);
			resolve(data);
		});
	});
};

var gen = function *() {
	var f1 = yield readFile('/etc/fstab');
	var f2 = yield readFile('/etc/shells');
	console.log(f1.toString());
	console.log(f2.toString());
}
```
``` js
var asyncReadFile = async function () {
	var f1 = await readFile('/etc/fstab');
	var f2 = await readFile('/etc/shells');
	console.log(f1.toString());
	console.log(f2.toString());
};
```
一比较就会发现， async函数就是将Generator函数的星号（*）替换成async，将yield替换成await，仅此而已。
async函数对Generator函数的改进，体现在以下四点。
（1）内置执行器。Generator函数的执行必须依靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。
``` js
var result = asyncReadFile();
```
上面代码调用了asyncReadFile函数，然后它就会自动执行，输出最后结果。这完全不像Generator函数，需要调用next方法，或者用co模块，才能得到真正执行，得到最后结果。
（2）更好的语义。async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。
（3）更广的适用性。co模块约定，yield命令后面只能是Thunk函数或Promise对象，而async函数的await命令后面，可以是Promise对象和原始类型的值（数值，字符串和布尔值，但这时等于同步操作）。
（4）返回值是Promise。async函数的返回值是Promise对象，这比Generator函数的返回值是Iterator对象方便多了。你可以用then方法指定下一步的操作。
正常情况下，await命令后面是一个Promise对象，否则会被转换成Promise
``` js
async function getTitle(url) {
	let response = await fetch(url);
	let html = await response.text();
	return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

async函数的实现
async函数的实现，就是将Generator函数和自动执行器，包装在一个函数里。
``` js
async function fn(args) {

}

function fn(args) {
	return spawn(function*() {

	});
}
```
所有的async函数都可以写成上面的第二种形式，其中spawn函数就是自动执行器。
下面给出spawn函数的实现，基本就是前文自动执行器的翻版。
``` js
function spawn(genF) {
	return new Promise(function(resolve, reject) {
		var gen = genF();
		function step(nextF) {
			try {
				var next = nextF();
			} catch(e) {
				return reject(e);
			}
			if (next.done) {
				return resolve(next.value);
			}
			Promise.resolve(next.value).then(function (v) {
				setp(function () { return gen.next(v); });
			}, function(e) {
				setp(function() { return gen.throw(e); });
			});
		}
		step(function() { return gen.next(undefined); });
	})
}
```
async函数是非常新的语法功能，新到都不属于ES6，而是属于ES7.目前，它仍处于提案阶段，但是转码器Babel和regenerator都已经支持，转码都就能使用。


### async 函数的用法
同Generator函数一样，async函数返回一个Promise对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等触发的异步操作完成，再接着执行函数体内后面的语句。
``` js
async function getStockPriceByName(name) {
	var symbol = await getStockSymbol(name);
	var stockPrice = await getStockPrice(symbol);
	return stockPrice;
}
