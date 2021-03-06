# SmartQQ协议

SmartQQ(WebQQ) API的PHP实现，灵感来自于[Java SmartQQ](https://github.com/ScienJus/smartqq)，感谢原作者对SmartQQ的详尽解释。

## 安装
```
composer require slince/smartqq *@dev
```

## 使用

### 登录，由于SmartQQ抛弃了用户名密码的登录方式，所以只能采用二维码登录
```
use Slince\SmartQQ\Client;

$smartQQ = new Client();

$smartQQ->login('/path/to/qrcode.png'); //参数为保存二维码的位置
```
如果成功的话你会在`/path/to/qrcode.png`下发现二维码，拿出手机扫一扫即可登录；注意在登录成功之前程序会阻塞直到确认成功；
成功之后你可以通过下面方式保存登录凭证，下次则可以绕过登录
```
$credential = $smartQQ->getCredential();
$credentialParameters = $credential->toArray();

//...

//还原凭证对象
$credential = Credential::fromArray($credentialParameters);
$smartQQ = new Client($credential);
```
### 查询好友、群以及讨论组

#### 好友相关

- 查询所有好友
```
$friends = $smartQQ->getFriends();

//找出昵称为张三的好友
$zhangSan = $friends->firstByAttribute('nick', '张三');

//自定义筛选，如找出所有女性并且是vip会员的好友
$girls = $friends->filter(function(Friend $friend){
     return $friend->getGender() == 'female' && $friend->isVip();
})->toArray();
```

- 查询好友详细资料

```
//上例，找出张三的资料
$profile = $smartQQ->getFriendDetail($zhangSan);
```

#### 群相关

- 查询所有群
```
$groups = $smartQQ->getGroups();

//找出名称为“少年”的群
$shaoNianGroup = $groups->firstByAttribute('name', '少年');
```
> 同样支持自定义筛选

#### 讨论组相关

- 查询所有讨论组
```
$discusses = $smartQQ->getDiscusses();

//找出名称为“少年”的讨论组
$shaoNianDiscuss = $discusses->firstByAttribute('name', '少年');
```
> 一样，也支持自定义筛选

- 查询讨论组的详细资料，比如群成员信息等

接上例，查询讨论组“少年”的详细资料
```
$shaoNianDetail = $smartQQ->getDiscussDetail($shaoNianDiscuss);

//所有群成员，支持筛选
$members = $shaoNianDetail->getMembers();
```

### 发送消息

#### 给好友发送消息

```
//1、找到好友
$friend = $friends->firstByAttribute('nick', '秋易');
//2、生成消息
$message = new FriendMessage($friend, new Content('你好'));
$result = $smartQQ->sendMessage($message);
var_dump($result);
```

#### 给群发送消息
```
//1、找到群
$group = $groups->firstByAttribute('name', 'msu');
//2、生成消息
$message = new GroupMessage($group, new Content('哈喽'));
$result = $smartQQ->sendMessage($message);
var_dump($result);
```

#### 发送讨论组消息
```
//1、找到讨论组
$discuss = $discusses->firstByAttribute('name', '他是个少年');
//2、生成消息
$message = new DiscussMessage($discuss, '讨论组消息');
$result = $smartQQ->sendMessage($message);
var_dump($result);
```

#### 给讨论组成员发消息
```
$discussMember = $smartQQ->getDiscussDetail($discuss)
    ->getMembers()
    ->firstByAttribute('nick', '张三');
$message = new FriendMessage($discussMember,  '你好');
$result = $smartQQ->sendMessage($message);
var_dump($result);
```

### 接收消息

```
$messages = $smartQQ->pollMessages();

```
关于消息的处理请参照examples


详细使用案例以及更多其它案例请参考[examples](./examples)

## 其它

- 关于103错误，多是由于webqq多点登录引起的，如果遇到错误，先到[http://w.qq.com/](http://w.qq.com/)确认能够收发消息，然后退出登录

- 关于登录凭证的时效性，smartqq是基于web接口的，对cookie有要求，登录成功之后如果长时间没有操作，cookie将会失效；此时需要重新登录
