
# 图片验证码
### 安装
建议使用openresty
gd.so依赖
```
yum -y install libjpeg-devel libpng-devel freetype-devel fontconfig-devel libXpm-devel libpng12 stix-fonts
```
依赖md5, string lua模块
将lualib里面文件放入/usr/local/openresty/lualib
checkcode.lua里面文件放入/usr/local/openresty/nginx/conf
http {}中加入:
```
lua_shared_dict checkcode 1m;
limit_req_zone $binary_remote_addr zone=req_check:1m rate=5r/s;
```
server {}中加入:
```
location /codeimg/ {
    default_type 'image/png';
    alias /dev/shm/checkcode/;
}
location ~ /restapi/v1/captchas(.*)$ {
    default_type 'text/html';
    limit_req zone=req_check burst=5;
    set $hashkey $1;
    content_by_lua_file /usr/local/openresty/nginx/conf/checkcode.lua;
}
```
### 使用方法如下

1. post方式访问http://IP或者域名/restapi/v1/captchas，post可以无内容，可以传当前时间戳
默认都应该正常返回信息，例如： {"errno":0,"errmsg": "success","data":"75889829502f06cfc2fc006a37e57c95"}
PS:
```
curl -d ' ' http://IP或者域名/restapi/v1/captchas
```

2. get方式拼接url访问，例如浏览器访问http://IP或者域名/restapi/v1/captchas/75889829502f06cfc2fc006a37e57c95
正常情况下或得到一个图片验证码

3. post方式访问http://IP或者域名/restapi/v1/captchas/check ， post数据格式如：'key=1873656651249e378ba4c2f55373702f&code=teiq'，其中hash为第一次请求时候的data，code为验证码图片内容（数字+字母共四位）
验证通过返回数据为：{"errno": 0,"errmsg": "success" }
验证失败返回信息为：{"errno": -3,"errmsg": "fail" }
ps:
```
curl http://IP/restapi/v1/captchas/check -d 'key=75889829502f06cfc2fc006a37e57c95&code=p4n9'
```

注意：
* 执行第一步后获取的hash会缓存5分钟，在五分钟内执行第二步可申请验证码，验证码自申请开始有效期5分钟，重复申请会重新获得验证码
* 已做限速控制，每秒每ip最多请求5次
