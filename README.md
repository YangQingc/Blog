# 博客系统实验报告
## 一、实验目的
1.对CSS,HTML,node.js及数据库的使用进行基本了解
2.设计博客系统
## 二、实验步骤
### 1. log.js
#### （1）首页
以下是首页js代码
```
app.get('/',(req,res)=>{
        res.sendFile(__dirname+'/html/start.html');
    });
```
以下是首页html(即start.html)
```
<h1> Welcome to online blog.</h1>
<form action="/toLogin" method="POST">
    <input type="submit" value="Login" /> <br>
</form>
<form action="/toRegist" method="POST">
    <input type="submit" value="Regist" /> <br>
</form>
```
功能是让用户选择登陆或注册。若选择登录则像后端以post方法发送'/toLogin',注册则利用post发送'/toRegist'。
#### (2) 注册

```
app.post('/toRegist',async (req,res)=>{
        newID=await ams.getNewID();
        var registPage=fs.readFileSync(__dirname+'/html/regist.html').toString();
        res.send('ID: '+newID+'<br>'+registPage);
    });
```
(以上是注册按钮响应)
其中ams是账户管理系统模块，它是一个自定义模块，并非预置模块。ams.getNewID是从后端获取一个未被注册的用户ID。此处不做赘述，将在后文详细解释。分配好ID后，用户设置昵称和密码。
```
<form action="/regist" method="POST">
    name: <input type="text" name="name" id="name"/> <br>
    
    password: <input type="text" name="password" id="password"/> <br>
    
    <input type="submit" value="Regist" /> <br>

</form>
```
（以上是注册界面,即regist.html）
```
app.post('/regist',async (req,res)=>{
        await ams.add(req.body.name,newID,req.body.password);
        newID=0;
        res.sendFile(__dirname+'/html/home.html');
        return newID;
    });
```
(以上是响应注册的中间件)
对新用户的信息进行保存,并跳转到home界面。后面会详细介绍home界面。ams.add的参数是新用户的ID,新用户的名字和密码，功能是在数据库中添加一个新用户，详细内容会在后面介绍。
#### （3）登录
```
app.post('/toLogin',(req,res)=>{
        res.sendFile(__dirname+'/html/login.html');
    });
```
（以上是登录按钮响应）
```
<form action="/login" method="POST">
    id: <input type="text" name="id" id="id"/> <br>
    
    password: <input type="text" name="password" id="password"/> <br>
    
    <input type="submit" value="Login" /> <br>
    
</form>
```
（以上是登陆界面，即login.html）
输入ID,密码即可登录
```
app.post('/login',async (req,res)=>{
        var id=req.body.id;
        var data=await ams.find(id);
        var loginPage=fs.readFileSync(__dirname+'/html/login.html').toString();
        if(data!=null)
        {
            if(data.password==req.body.password)
            {
                res.sendFile(__dirname+'/html/home.html');
                return newID;
            }
            else
            {
                res.send(loginPage+'<p>Wrong password.</p>');
            }
        }
        else
        {
            res.send(loginPage+'<p>The account does not exist.</p>');
        }
    });
```
以上是登录响应，获取用户输入的ID和密码后，先判断ID是否存在于数据库中，若不存在，则输出:The account does not exist.若存在，则判断密码是否正确。若不正确则输出:Wrong password，若正确则在后端记录当前登录用的ID，并且跳转至home.html。ams.find的唯一参数是需要查询的id，若存在该用户，则返回全部用户信息，否则返回null.
#### (4) home.html
```
<html>

<h1>Welcome back!!!!</h1>

<form action="/shift" method="GET">
    <input type="submit" value="Back!" /> <br>
</form>

</html>
```
以上是home.html的全部内容。
功能是欢迎用户登录，点击“back”后，正式进入博客系统
#### (5) log.js的导出函数
log.js仅导出一个函数start(port,app)，第一个参数是博客系统的端口号，第二个参数是博客系统的app。其返回值当前登录用户的ID。功能是完成所有的登录注册及页面跳转，直至正式进入博客系统
### 2. ams.js
```
const database=require('./database');
const account=database.account;

async function add(name,id,password)
{
    var data=new account({
        name:name,
        id:id,
        password:password
    });
    await data.save();
}
async function remove(id)
{
    await account.deleteMany({id:id});
}
async function update(id,password)
{
    var data=await account.findOne({id:id});
    data.password=password;
    await data.save();
}
async function find(id)
{
    var data=await account.findOne({id:id});
    return data;
}
async function getNewID()
{
    for(var i=1;i<=9999999;i++)
    {
        var s=i.toString();
        var tem=7-s.length;
        for(var j=1;j<=tem;j++)
        {
            s='0'+s;
        }
        if(await find(s)==null)
        {
            return s;
            break;
        }
    }
    return '0000000';
}

module.exports.add=add;
module.exports.remove=remove;
module.exports.update=update;
module.exports.find=find;
module.exports.getNewID=getNewID;
```
以上是账户管理系统模块的全部代码。
它导出了五个函数，分别是add,remove,update,find,getNewID。其中add,find,getNewID已介绍过函数原型，代码已做展示，不做赘述，仅介绍getNewID的算法：从0000001开始，逐次加一，判断是否已被注册，返回第一个未被注册的ID。
remove的参数是用户的ID，功能是永久注销该ID所属的账户。
update的参数是ID和新密码，功能是将该ID所属的账户的密码更换为新密码。
可以注意到ams.js引用了另一个自定义模块database.js。database.js只有基本的约束声明和数据库连接，model获取等，故仅作代码展示。database.js代码如下:
```
const mongoose=require('mongoose');
mongoose.connect('mongodb://my-mongo/wsy');

const accountSchema=new mongoose.Schema({
    id:String,
    name:String,
    password:String
});

const account=mongoose.model('account',accountSchema);

async function clear()
{
    await account.deleteMany({});
};

module.exports.account=account;
module.exports.clear=clear;
```
### 3. server.js
```
const express = require('express');
const app = express();

function wsy() {

const marked = require('marked')


const mongoose = require('mongoose');
mongoose.connect('mongodb://my-mongo/wsy');
const articleSchema = new mongoose.Schema({
    title: String,
    description: String,
    createdAt: { type: Date, default: Date.now },
    markdown: String,
    html: String
});

articleSchema.pre('validate', function(next) {

  if (this.markdown) {
    this.html = marked(this.markdown)
  }

  next()
})

const article = mongoose.model('article', articleSchema);

app.use(express.static('public'));

app.set('view engine', 'ejs');

app.get('/shift', async (req, res) => {
  const all = await article.find().sort({ createdAt: 'desc' });
  res.render('all', { articles: all })
})

app.get('/new', (req, res) => {
  res.render('new');
})

app.get('/display/:id', async (req, res) => {
  const one = await article.findOne({ _id: req.params.id });
  res.render('display', { article: one });
})

app.get('/edit/:id', async (req, res) => {
    const one = await article.findOne({ _id: req.params.id });
    res.render('edit', { article: one })
})

app.use(express.urlencoded({ extended: false }))

app.post('/new', async (req,res) => {
    one = new article({ title: req.body.title, description: req.body.description, markdown: req.body.markdown });
    await one.save();
    res.render('display', { article: one })
})

const methodOverride = require('method-override')
app.use(methodOverride('_method'))

app.delete('/:id', async (req, res) => {
  await article.deleteMany({ _id: req.params.id });
  res.redirect('/shift?')
})

app.put('/:id', async (req, res) => {
    let data = {}
    data.title = req.body.title
    data.description = req.body.description
    data.markdown = req.body.markdown

    var one = await article.findOne({ _id: req.params.id });
    if (one != null) {
        one.title = data.title;
        one.description = data.description;
        one.markdown = data.markdown;
        await one.save();       
    }  
    res.redirect(`/display/${req.params.id}`);
})

app.listen(12331)
}
module.exports.wsy= wsy
module.exports.app=app;
```
server.js模块完成了文章的编辑，删除，查看和创建。同时按照时间顺序展示全部的已发布文章。
该模块金导出了博客系统的app，并且将所有功能汇集到wsy函数中，wsy无参数无返回值。
### 4. main.js
```
const server=require('./server.js');
server.wsy();
const log=require('./log/log.js');
log.start(12331,server.app);
```
该模块对所有功能进行汇总，是主模块，整个程序从这个模块开始进行。
## 三、实验总结与反思
做出了博客系统的整体框架，但美术细节仍需优化，同时可以利用账户管理系统开发新的社交功能，如添加好友等。

| 基础功能 | 新增功能1 | 新增功能2 | 新增功能3 | 新增功能4 | 新增功能5 | 新增功能6 | 新增功能7 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 博客系统的增删改查 | 记录文本创建时间并排序 | 添加markdown输入框 | 加入read more按钮 | 实现markdown | 在display中渲染markdown | 美化博客系统 | 登陆注册以及账户管理系统 |
| 7 | 8 | 1 | 2 | 1 | 2 | 2 | 18 |
