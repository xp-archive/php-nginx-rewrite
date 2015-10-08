# php-nginx-rewrite

## description

这段 nginx 的 rewrite 能够实现如下自动寻找入口的功能：

第 n 个请求失败时尝试第 n+1 种方案。

1. 请求 /a/b/c
2. 请求 /a/b/index.php，/c 作为 PATH_INFO
3. 请求 /a/index.php，/b/c 作为 PATH_INFO
4. 请求 /index.php，/a/b/c 作为 PATH_INFO
5. 返回 404


## configuration

```nginx
server {
    # php first
    index index.php index.html index.htm;

    # init
    set $path $request_uri;
    set $path_info "";
        
    location / {
        try_files $uri $uri/ @404;
    }

    location @404 {
        if ($path ~ ^(.*)(/.+)$) {
            set $path $1/index.php;
            set $path_info $2;
            rewrite .* $path last;
        }
        return 404;
    }

    location ~ .+\.php($|/) {
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        
        if ($path_info !~ .+) {
            set $path_info $fastcgi_path_info;
        }

        try_files $fastcgi_script_name @404php;
        
        fastcgi_param PATH_INFO $path_info;

        fastcgi_index index.php;
        include fastcgi.conf;
    }

    location @404php {
        if ($path = /index.php) {
            return 404;
        }
        if ($path ~ ^(.*)(/.+)/index\.php$) {
            set $path_info $2$path_info;
            set $path $1/index.php;
            rewrite .* $path last;
        }
        return 404;
    }

}

```
