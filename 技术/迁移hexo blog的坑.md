

![](file-20241201213418419.png)

1、执行部署的时候报错
```shell
INFO  Validating config

WARN  Deprecated config detected: "use_date_for_updated" is deprecated, please use "updated_option" instead. See https://hexo.io/docs/configuration for more details.

INFO  Deploying: git

INFO  Clearing .deploy_git folder...

FATAL {

  err: Error: EACCES: permission denied, unlink '/Users/castile/Documents/hexo-blog/blog-backup/.deploy_git/404.html'

      at async Object.unlink (node:internal/fs/promises:1060:10) {

    errno: -13,

    code: 'EACCES',

    syscall: 'unlink',

    path: '/Users/castile/Documents/hexo-blog/blog-backup/.deploy_git/404.html'

  }
```


![[Pasted image 20241107222926.png]]

> 解决：
> 切换root用户
> sudo -s
> 重新执行 hexo d 即可


![[Pasted image 20241201212941.png]]
