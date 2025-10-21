## 展示页面

* [清羽飞扬の友链朋友圈](https://blog.liushen.fun/fcircle/)

* [❖星港◎Star☆ 的友链朋友圈](https://blog.starsharbor.com/fcircle/)

* [梦爱吃鱼的友链朋友圈](https://blog.bsgun.cn/fcircle/)

* [mccsjs的友链朋友圈](https://seln.cn/fc/)


## 项目介绍

- **爬取文章**: 爬取所有友链的文章，结果放置在根目录的all.json文件中，方便读取并部署到前端。
- **邮箱推送更新(对作者推送所有友链更新)**: 作者可以通过邮箱订阅所有rss的更新（未来开发）。
- **issue邮箱订阅(对访客实时推送最新文章邮件)**: 基于`GitHub issue`的博客更新邮件订阅功能，游客可以通过简单的提交`issue`进行邮箱订阅站点更新，删除对应`issue`即可取消订阅。
- **文件分离**: 将前后端分离，前端文件放在page分支，后端文件放在主分支

## 特点介绍

* **轻量化**：对比原版友链朋友圈的功能，该友圈功能简洁，去掉了设置和fastAPI的臃肿，仅保留关键内容。
* **无数据库**：因为内容较少，我采用`json`直接存储文章信息，减少数据库操作，提升`action`运行效率。
* **部署简单**：原版友链朋友圈由于功能多，导致部署较为麻烦，本方案仅需简单的部署action即可使用，vercel仅用于部署前端静态页面和实时获取最新内容。
* **文件占用**：对比原版`4MB`的`bundle.js`文件大小，本项目仅需要`5.50KB`的`fclite.min.js`文件即可轻量的展示到前端。

## 功能概览

* 文章爬取
* 暗色适配
* 显示作者所有文章
* 获取丢失友链数据
* 随机钓鱼
* 邮箱推送
* 美观邮箱模板
* 自部署(2024-08-11添加)
* 前端单开分支(2024-09-05添加) @CCKNBC

## action部署使用方法

### 前置工作

1. **Fork 本仓库:**
   点击页面右上角的 Fork 按钮，将本仓库复制到你自己的`GitHub`账号下，不要勾选仅复刻main分支。
   ![](./static/fork.png)

2. **配置 Secrets:**
   在你 Fork 的仓库中，依次进入 `Settings` -> `Secrets` -> `New repository secret`，添加以下 Secrets：
   - `SMTP_PWD`(可选): SMTP 服务器的密码，用于发送电子邮件，如果你不需要，可以不进行配置。

   ![](./static/1.png)
   
3. **配置action权限：**
   
   在设置中，点击`action`，拉到最下面，勾选`Read and write permissions`选项并保存，确保action有读写权限。
   
4. **启用 GitHub Actions:**
   GitHub Actions 已经配置好在仓库的 `.github/workflows/*.yml` 文件中，当到一定时间时将自动执行，也可以手动运行。
   其中，每个action功能如下：
   
   - `friend_circle_lite.yml`实现核心功能，爬取并发送邮箱，需要在Action中启用；
   - `deal_subscribe_issue.yml`处理固定格式的issue，打上固定标签，评论，并关闭issue；
   
5. **设置issue格式：**
   这个我已经设置好了，你只需要检查issue部分是否有对应格式即可，可以自行修改对应参数以进行自定义。

### 配置选项

1. 如果需要修改爬虫设置或邮件模板等配置，需要修改仓库中的 `config.yaml` 文件：

   - **爬虫相关配置**
     使用 `requests` 库实现友链文章的爬取，并将结果存储到根目录下的 `all.json` 文件中。
     
     ```yaml
     spider_settings:
       enable: true
       json_url: "https://blog.qyliu.top/friend.json"
       article_count: 5
       merge_result:
         enable: true
         merge_json_url: "https://fc.liushen.fun"
     ```
     
     `enable`：开启或关闭，默认开启；
     
     `json_url`：友链朋友圈通用爬取格式第一种（下方有配置方法）;
     
     `article_count`：每个作者留存文章个数。

     `marge_result`：是否合并多个json文件，若为true则会合并指定网络地址和本地地址的json文件并去重
     
     - `enable`：是否启用合并功能，该功能提供与自部署的友链合并功能，可以解决服务器部分国外网站，服务器无法访问的问题
     
     - `marge_json_path`：请填写网络地址的json文件，用于合并，不带空格！！！
     

**如果你配置正常，那么等action运行一次（可以手动运行）应该就可以在page分支看到结果了，检查一下，如果结果无误，可以继续看下一步**

### 友圈json生成

**注意，以下可能仅适用于hexo-theme-butterfly或部分类butterfly主题，如果你是其他主题，可以自行适配，理论上只要存在友链数据文件都可以整理为该类型，甚至可以自行整理为对应json格式后放到 `/source` 目录下即可，格式可以参考：`https://seln.cn/friend.json` **

1. 将以下文件放置到博客根目录：

   ```javascript
   const YML = require('yamljs')
   const fs = require('fs')
   
   let friends = [],
       data_f = YML.parse(fs.readFileSync('source/_data/link.yml').toString().replace(/(?<=rss:)\s*\n/g, ' ""\n'));
   
   data_f.forEach((entry, index) => {
       let lastIndex = 2;
       if (index < lastIndex) {
           const filteredLinkList = entry.link_list.filter(linkItem => !blacklist.includes(linkItem.name));
           friends = friends.concat(filteredLinkList);
       }
   });
   
   // 根据规定的格式构建 JSON 数据
   const friendData = {
       friends: friends.map(item => {
           return [item.name, item.link, item.avatar];
       })
   };
   
   // 将 JSON 对象转换为字符串
   const friendJSON = JSON.stringify(friendData, null, 2);
   
   // 写入 friend.json 文件
   fs.writeFileSync('./source/friend.json', friendJSON);
   
   console.log('friend.json 文件已生成。');
   ```

2. 在根目录下运行：

   ```bash
   node link.js
   ```

   你将会在source文件中发现文件`friend.json`，即为对应格式文件，下面正常hexo三件套即可放置到网站根目录。

3. (可选)添加运行命令到脚本中方便执行，在根目录下创建：

   ```bash
   @echo off
   E:
   cd E:\Programming\HTML_Language\willow-God\blog
   node link.js && hexo g && hexo algolia && hexo d
   ```

   地址改成自己的，上传时仅需双击即可完成。

   如果是github action，可以在hexo g脚本前添加即可完整构建，注意需要安装yaml包才可解析yml文件。

## 部署静态网站

首先，将该项目部署到vercel，部署到vercel等平台的目的主要是检测仓库变动并实时更新数据，及时获取all.json文件内容。任意平台均可，但是注意，部署的分支为page分支。

1. vercel 部署完成后，检查对应页面，如果页面中没有数据，且 `/all.json` 路径无法访问可能是部署到main分支了，可以通过 `setting-git-Production Branch` ，填写为page并重新进行部署即可
   
   ![](./static/vercel.png)

2. zeabur 可以在部署时直接选择分支：
   
   ![](./static/zeabur.png)

3. CloudFlare Page 也可以在构建时即选择对应的分支，这里不再细讲。

   ![](./static/cloudflare.png)

部署完成后，你将获得一个地址，如果是通过vercel部署的，建议自行绑定域名。

检查 `https://example.com/all.json` 是否有数据，如果有，则部署成功。

## 部署到你的页面

在前端页面的md文件中写入：

```html
<div id="friend-circle-lite-root"></div>
<script>
    if (typeof UserConfig === 'undefined') {
        var UserConfig = {
            // 填写你的fc Lite地址
            private_api_url: 'https://fc.liushen.fun/',
            // 点击加载更多时，一次最多加载几篇文章，默认20
            page_turning_number: 20,
            // 头像加载失败时，默认头像地址
            error_img: 'https://i.p-i.vip/30/20240815-66bced9226a36.webp',
        }
    }
</script>
<link rel="stylesheet" href="https://fastly.jsdelivr.net/gh/willow-god/Friend-Circle-Lite/main/fclite.min.css">
<script src="https://fastly.jsdelivr.net/gh/willow-god/Friend-Circle-Lite/main/fclite.min.js"></script>
```

其中第一个地址填入你自己的地址即可，**注意**尾部带`/`，不要遗漏。

然后你就可以在前端页面看到我们的结果了。效果图如上展示网站，其中两个文件你可以自行修改，在同目录下我也提供了未压缩版本，有基础的可以很便捷的进行修改。

## Star增长曲线

[![Star History Chart](https://api.star-history.com/svg?repos=willow-god/Friend-Circle-Lite&type=Timeline)](https://star-history.com/#willow-god/Friend-Circle-Lite&Timeline)
