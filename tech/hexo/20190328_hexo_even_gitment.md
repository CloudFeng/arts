# Hexo Even主题集成gitment评论系统

## 准备条件

本文介绍hexo Even主题集成giment评论系统的过程。所谓gitment就是把评论放到github的issues系统里。前提是你需要使用Hexo和Even搭建博客。然后在[github applications](https://github.com/settings/applications/new)注册自己网站，拿到client_id和client_secret。


## 一步一步的集成

### 修改`themes/even/_config.yml`

添加如下代码，将xxx和yyy修改成自己的即可。

```Code
gitment:
  enable: true # 是否开启gitment评论系统
  mint: true #
  count: true # 是否显示评论数
  lazy: false # 懒加载，设置为ture时需手动展开评论
  cleanly: true # 是否隐藏'Powered by ...'
  language: en # 语言，置空则随主题的语言
  github_user: xxx # Github用户名
  github_repo: xxx.github.io # 在Github新建一个仓库用于存放评论，这是仓库名
  client_id: yyyy # 注册OAuth Application时生成
  client_secret: xxxxxxxxx # 注册OAuth Application时生成
  proxy_gateway: # Address of api proxy, See: https://github.com/aimingoo/intersect
  redirect_protocol: # Protocol of redirect_uri with force_redirect_protocol when mint enabled
```

### 修改`themes/even/languages/en.yml(zh-cn.yml)`

找到`posts`,然后添加如下代码：

- `themes/languages/zh-cn.yml`： `gitmentbutton: 显示 Gitment 评论`；
- `themes/languages/en.yml`：`gitmentbutton: Show comments from Gitment`

### 修改`themes/even/layout/_partials/comments.swig`

- 把第一行代码改为：`{% if page.comments and is_post() %}`

- 在下面的代码后面：

```Code
<div id="lv-container" data-id="city" data-uid="{{ theme.livere_datauid }}">
    <noscript>为正常使用来必力评论功能请激活JavaScript</noscript>
</div>
```

添加如下代码：

```Code
{% elseif theme.gitment.enable %}
  {% if theme.gitment.lazy %}  
    <div onclick="ShowGitment()" id="gitment-display-button">{{  __('posts.gitmentbutton') }}</div>
    <div id="gitment-container" style="display:none"></div>
  {% else %}
    <div id="gitment-container"></div>
```

### 添加 `gitment.swig`文件

在`themes/even/layout/_script/_comments/`目录下新建`gitment.swig`文件，里面的内容如下：

```Code
{% if theme.gitment.enable %}
   {% set owner = theme.gitment.github_user %}
   {% set repo = theme.gitment.github_repo %}
   {% set cid = theme.gitment.client_id %}
   {% set cs = theme.gitment.client_secret %}
   <link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
   <script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
   {% if not theme.gitment.lazy %}
       <script type="text/javascript">
           var gitment = new Gitment({
               id: window.location.pathname, 
               owner: '{{owner}}',
               repo: '{{repo}}',
               oauth: {
                   client_id: '{{cid}}',
                   client_secret: '{{cs}}',
               }});
           gitment.render('gitment-container');
       </script>
   {% else %}
       <script type="text/javascript">
           function ShowGitment(){
               document.getElementById("gitment-display-button").style.display = "none";
               document.getElementById("gitment-container").style.display = "block";
               var gitment = new Gitment({
                   id: document.location.href, 
                   owner: '{{owner}}',
                   repo: '{{repo}}',
                   oauth: {
                       client_id: '{{cid}}',
                       client_secret: '{{cs}}',
                   }});
               gitment.render('gitment-container');
           }
       </script>
   {% endif %}
{% endif %}
```

### 修改`themes/even/layout/_script/comments.swig`

- 修改第一行：`{% if is_post() %}`
- 引入`gitment.swig`: `{% include '_comments/gitment.swig' %}`


### 新增 `_gitment.scss`

在`themes/even/css/_partial/_post`目录下新增`_gitment.scss`文件，内容如下：

```Code
#gitment-display-button{
  display: inline-block;
  padding: 0 15px;
  color: #0a9caf;
  cursor: pointer;
  font-size: 14px;
  border: 1px solid #0a9caf;
  border-radius: 4px;
}
#gitment-display-button:hover{
  color: #fff;
  background: #0a9caf;
}
```

整个流程就结束了。

## 参考资料

- [hexo next主题集成gitment评论系统](https://blog.csdn.net/yanzi1225627/article/details/77890414)
- [hexo 辅助函数](https://hexo.io/zh-cn/docs/helpers.html)