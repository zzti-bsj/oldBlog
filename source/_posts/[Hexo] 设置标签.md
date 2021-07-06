

title: 在博客的根目录下创建tags目录

tags: Hexo

---

## 在博客的根目录下创建tags目录

1. 创建tags目录

`hexo new page tags`

2. 设置tags/index.html

   在index.html中加上type: "tags"

   ```shell
   ---
   title: tags
   date: 2021-07-06 23:44:14
   type: "tags"
   ---
   ```

## 创建categories目录

同创建tags目录的方式一样，把关键字tags换成categories即可。