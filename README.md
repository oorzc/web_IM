# 网页版聊天室

> 访问地址 http://im.oorzc.cn

## 项目说明

* 本项目为前后端分离项目
* 前端技术 LayUI LayIM JQuery
* 后端技术 NodeJS ThinkJs
* 数据库 Mysql  缓存处理 Redis 通信方式 WebScoket
* 前后端鉴权处理 JSON Web Token(JWT)


## 已实现功能

1. 消息发送
* 单聊、群聊、非好友临时会话
* 文字、表情、图片、文件、视频 在线/离线消息发送和接收。
* 消息撤回（2分钟内），群消息撤回（群管理员可撤回2小时内发送的消息）
* 离线消息使用redis缓存
2. 好友管理
* 添加、删除好友，修改好友备注，好友分组，移动好友至分组，分组重命名
3. 群组管理
  * 群全体禁言、解禁，指定用户时长禁言、解禁
  * 添加、移除管理员
  * 修改群名片 群主、管理员可修改群资料
  * 将用户踢出群
4. 消息通知
  * 顶号登录通知
  * 好友上线下状态更新显示（redis消息队列处理）
  * 申请添加好友、群通知，同意、拒绝添加好友、群通知
5. 个人资料、头像修改
6. 聊天记录 查看、日期搜索
7. 输入状态实时展示（隐身不向对方展示）
8. 心跳检测（防止用户长时间不操作掉线）



## 项目部分截图

* 查找好友或群
![](http://qiniu.sponges.cn/201811172002_164.png?imageView2/0/w/880/h/680)
![](http://qiniu.sponges.cn/201811172003_37.png?imageView2/0/w/880/h/680)

* 好友
![](http://qiniu.sponges.cn/201811172000_349.png?imageView2/0/w/880/h/680)

* 群组

![](http://qiniu.sponges.cn/201811171954_553.png?imageView2/0/w/880/h/680)
![](http://qiniu.sponges.cn/201811171955_252.png?imageView2/0/w/880/h/680)
![](http://qiniu.sponges.cn/201811171957_268.png?imageView2/0/w/880/h/680)
![](http://qiniu.sponges.cn/201811171958_137.png?imageView2/0/w/880/h/680)



## 数据库设计

```sql
-- ----------------------------
-- Table structure for im_chat_friend
-- ----------------------------
DROP TABLE IF EXISTS `im_chat_friend`;
CREATE TABLE `im_chat_friend` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `from_id` int(11) NOT NULL COMMENT '发送者',
  `to_id` int(11) NOT NULL COMMENT '接收者',
  `content` text NOT NULL COMMENT '聊天内容',
  `status` tinyint(1) unsigned NOT NULL DEFAULT '1' COMMENT '1未读 2已读 3撤回',
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=100 DEFAULT CHARSET=utf8 COMMENT='好友聊天日志';

-- ----------------------------
-- Table structure for im_chat_group
-- ----------------------------
DROP TABLE IF EXISTS `im_chat_group`;
CREATE TABLE `im_chat_group` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `from_id` int(11) NOT NULL,
  `to_id` int(11) NOT NULL,
  `content` text NOT NULL,
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1未读 2已读 3删除',
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=76 DEFAULT CHARSET=utf8 COMMENT='群聊天日志';

-- ----------------------------
-- Table structure for im_friend
-- ----------------------------
DROP TABLE IF EXISTS `im_friend`;
CREATE TABLE `im_friend` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) NOT NULL COMMENT '用户id',
  `friend_id` int(11) NOT NULL COMMENT '好友ID',
  `group_id` int(11) NOT NULL COMMENT '分组id',
  `remark` varchar(25) NOT NULL COMMENT '好友备注',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1正常 2特别关注 3拉黑',
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=19 DEFAULT CHARSET=utf8 COMMENT='好友表';

-- ----------------------------
-- Table structure for im_friend_group
-- ----------------------------
DROP TABLE IF EXISTS `im_friend_group`;
CREATE TABLE `im_friend_group` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) NOT NULL,
  `groupname` varchar(15) NOT NULL,
  `sort` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=30 DEFAULT CHARSET=utf8 COMMENT='好友分组';

-- ----------------------------
-- Table structure for im_group
-- ----------------------------
DROP TABLE IF EXISTS `im_group`;
CREATE TABLE `im_group` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `groupname` varchar(255) NOT NULL COMMENT '群名称',
  `avatar` varchar(255) NOT NULL  COMMENT '群头像',
  `desc` varchar(255) NOT NULL COMMENT '群介绍',
  `belong` varchar(11) NOT NULL COMMENT '群主ID',
  `manger` text NOT NULL COMMENT '管理员ID集合',
  `approval` tinyint(1) NOT NULL DEFAULT '1' COMMENT '是否需要验证 1需要 2不需要',
  `number` int(4) NOT NULL DEFAULT '200' COMMENT '群人数',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1正常 2禁言',
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10003 DEFAULT CHARSET=utf8 COMMENT='群组表';

-- ----------------------------
-- Table structure for im_group_member
-- ----------------------------
DROP TABLE IF EXISTS `im_group_member`;
CREATE TABLE `im_group_member` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) NOT NULL,
  `group_id` int(11) NOT NULL,
  `remark` varchar(25) NOT NULL,
  `type` tinyint(1) unsigned NOT NULL DEFAULT '3' COMMENT '身份 1群主 2管理员 3群员',
  `status` tinyint(1) unsigned NOT NULL DEFAULT '1' COMMENT '1正常 2禁言',
  `sort` int(11) NOT NULL,
  `disable_time` datetime NOT NULL COMMENT '禁言时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8 COMMENT='群组成员';

-- ----------------------------
-- Table structure for im_notify
-- ----------------------------
DROP TABLE IF EXISTS `im_notify`;
CREATE TABLE `im_notify` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `from_id` int(11) NOT NULL COMMENT '''消息发送者 0表示为系统消息''',
  `to_id` int(11) NOT NULL COMMENT '''消息接收者 0表示全体会员'',',
  `type` tinyint(1) unsigned NOT NULL COMMENT '1为请求添加用户 2为系统消息（添加好友）3为请求加群 4为系统消息（添加群） 5 全体会员消息',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1未读 2已读',
  `remark` varchar(255) NOT NULL COMMENT '''附加消息'',',
  `create_time` datetime NOT NULL COMMENT '''发送消息时间'',',
  `operation` tinyint(255) NOT NULL COMMENT '操作状态',
  `group_id` int(11) NOT NULL COMMENT '好友添加：好友分组id，群添加：群id',
  `handle_id` int(11) unsigned NOT NULL COMMENT '操作者id',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=23 DEFAULT CHARSET=utf8 COMMENT='消息通知';

-- ----------------------------
-- Table structure for im_user
-- ----------------------------
DROP TABLE IF EXISTS `im_user`;
CREATE TABLE `im_user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `nickname` varchar(35) NOT NULL COMMENT '昵称',
  `username` varchar(15) NOT NULL COMMENT '帐号',
  `password` varchar(35) NOT NULL COMMENT '密码',
  `avatar` varchar(255) NOT NULL  COMMENT '头像',
  `sign` varchar(25) NOT NULL COMMENT '签名',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1正常 2禁用',
  `mobile` varchar(255) NOT NULL COMMENT '手机',
  `sex` tinyint(1) unsigned NOT NULL COMMENT '性别 1男 2女 3保密',
  `job` tinyint(1) NOT NULL COMMENT '职业',
  `blood` tinyint(1) NOT NULL,
  `birthday` varchar(25) NOT NULL,
  `email` varchar(25) NOT NULL,
  `qq` varchar(25) NOT NULL,
  `wechat` varchar(25) NOT NULL,
  `online` varchar(11) NOT NULL DEFAULT 'out' COMMENT '在线状态 1online 2hide 3out',
  `reg_time` datetime NOT NULL COMMENT '注册时间',
  `last_login_time` datetime NOT NULL COMMENT '上次登录时间',
  `last_login_ip` varchar(25) NOT NULL COMMENT '上次登录IP',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10012 DEFAULT CHARSET=utf8 COMMENT='用户表';

```


