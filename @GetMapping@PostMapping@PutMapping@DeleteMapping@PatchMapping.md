# *@GetMapping**@PostMapping**@PutMapping**@DeleteMapping**@PatchMapping*

@GetMapping用于处理请求方法的GET类型

@PostMapping用于处理请求方法的POST类型

@RequestMapping(value = “/get/{id}”, method = RequestMethod.GET)

新方法可以简化为：

@GetMapping("/get/{id}")



# get和post的使用场景



HTTP 定义了与服务器交互的不同方法，最常用的有4种，Get、Post、Put、Delete,如果我换一下顺序就好记了，Put（增）,Delete（删），Post（改）,Get（查），即增删改查，下面简单叙述一下：

1）Get， 它用于获取信息，注意，他只是获取、查询数据，也就是说它不会修改服务器上的数据，从这点来讲，它是数据安全的，而稍后会提到的Post它是可以修改数据的，所以这也是两者差别之一了。

2） Post，它是可以向服务器发送修改请求，从而**修改服务器**的，比方说，我们要在论坛上回贴、在博客上评论，这就要用到Post了，当然它也是可以仅仅获取数据的。

3）Delete 删除数据。可以通过Get/Post来实现。用的不多，暂不多写，以后扩充。

4）Put，增加、放置数据，可以通过Get/Post来实现。用的不多，暂不多写，以后扩充。





下面简述一下Get和Post区别：

1） GET请求的数据是放在HTTP包头中的，也就是URL之后，通常是像下面这样定义格式的，（*而Post是把提交的数据放在HTTP正文中的*）。

login.action?name=hyddd&password=idontknow&verify=%E4%BD%E5%A5%BD

a，以 ？ 来分隔URL和数据； 

b，以& 来分隔参数；

c，如果数据是英文或数字，原样发送；

d，如果数据是中文或其它字符，则进行BASE64编码。  

2）GET提交的数据比较少，最多1024B，因为GET数据是附在URL之后的，而URL则会受到不同环境的限制的，比如说IE对其限制为2K+35，而POST可以传送更多的数据（理论上是没有限制的，但一般也会受不同的环境，如浏览器、[操作系统](http://lib.csdn.net/base/operatingsystem)、服务器处理能力等限制，IIS4可支持80KB，IIS5可支持100KB）。

3）Post的安全性要比Get高，因为Get时，参数数据是明文传输的，而且使用GET的话，还可能造成Cross-site request forgery攻击。而POST数据则可以加密的，但GET的速度可能会快些。

所以综上几点，总结成下表：



| 操作方式 | 数据位置 | 明文密文 | 数据安全 | 长度限制                                            | 应用场景 |
| -------- | -------- | -------- | -------- | --------------------------------------------------- | -------- |
| GET      | HTTP包头 | 明文     | 不安全   | 长度较小                                            | 查询数据 |
| POST     | HTTP正文 | 可明可密 | 安全     | 支持较[大数据](http://lib.csdn.net/base/hadoop)传输 | 修改数据 |



总的来说，get是用来查询数据，post是用来修改数据。比方说，我们要在**论坛上回贴、在博客上评论**，这就要用到Post了，当然它也是可以仅仅获取数据的。