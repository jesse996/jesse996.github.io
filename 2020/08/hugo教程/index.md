# Hugo教程


# 常用命令

## hugo new

添加网站内容. 例如: `hugo new about.md`, 他会在 **content** 目录下生成一个 `about.md` 的文件, 根据这个文件可以生成对应的静态页面. 可以在 `about.md` 前面添加对应的路径, 但文件会以 `content` 为根目录, 也就是说所有添加新文件都会存放在 `content` 目录下面.

## hugo new theme

为网站添加 UI, 也就是模板文件/主题文件. 例如: `hugo new theme mytheme`. 这会在 themes 目录下创建一个 mytheme 目录, mytheme 目录中会默认添加一些基本的文件结构. 所有的模板/主题文件都会保存在 themes 目录中.

## hugo

生成静态网站, 默认在生成的静态文件保存在 public 目录中. 也可以指定路.

## hugo server

hugo 自带一个 web 服务器, 运行 hugo server 后可以通过 `http://localhost:1313` 来访问静态网站.

下面是 hugo server 常用的参数, 注意大小写:

-   -p 端口: 修改默认端口
-   -D: 在使用 server 预览网站时, draft 属性为 ture 的草稿文件是不会生成预览的. 添加-D 后可以预览草稿文件.

# 文章和页面的对应关系

```
└── content
    ├── _index.md          // [home]            <- https://example.com/ **
    ├── about.md           // [page]            <- https://example.com/about/
    ├── posts
    |   ├── _index.md      // [section]         <- https://example.com/posts/ **
    |   ├── firstpost.md   // [page]            <- https://example.com/posts/firstpost/
    |   ├── happy
    |   |   ├── _index.md  // [section]         <- https://example.com/posts/happy/ **
    |   |   └── ness.md    // [page]            <- https://example.com/posts/happy/ness/
    |   └── secondpost.md  // [page]            <- https://example.com/posts/secondpost/
    └── quote
        ├── _index.md      // [section]         <- https://example.com/quote/ **
        ├── first.md       // [page]            <- https://example.com/quote/first/
        └── second.md      // [page]            <- https://example.com/quote/second/

// hugo默认生成的页面, 没有对应的markdown文章
分类列表页面               // [taxonomyTerm]    <- https://example.com/categories/  **
某个分类下的所有文章的列表  // [taxonomy]        <- https://example.com/categories/one-category  **
标签列表页面               // [taxonomyTerm]    <- https://example.com/tags/  **
某个标签下的所有文章的列表  // [taxonomy]        <- https://example.com/tags/one-tag  **
```

中括号`[]`中标注的是页面的 kind 属性, 他们整体上分为两类: `single(单页面 - page)` 和 `list(列表页 - home, section, taxonomyTerm, taxonomy)`.

content 目录下的所有 `_index.md` 可以用来生成对应的列表页面, 如果没有这些 markdown 文件, hugo 也会默认生成对应的页面. 有这些 markdown 文件的话, hugo 会根据文件里面的 FrontMatter 的设置生成更个性的页面.

# 页面和模板的对应关系

页面和模板的应对关系是根据页面的一系列的属性决定的, 这些属性有: Kind, Output Format, Language, Layout, Type, Section. 他们不是同时起作用, 其中 kind, layout, type, section 用的比较多.

-   kind: 用于确定页面的类型, 单页面使用 single.html 为默认模板页, 列表页使用 list.html 为默认模板页, 值不能被修改
-   section: 用于确定 section tree 下面的文章的模板. section tree 的结构是由 content 目录结构生成的, 不能被修改, content 目录下的一级目录自动成为 root section, 二级及以下的目录, 需要在目录下添加 `_index.md` 文件才能成为 section tree 的一部分. 如果页面不在 section tree 下 section 的值为空
-   type: 可以在 Front Matter 中设置, 用户指定模板的类型. 如果没设定 type 的值, type 的值等于 section 的值 或 等于 page(section 为空的时候)
-   layout: 可以在 Front Matter 中设置, 用户指定具体的模板名称.

从层次上 hugo 中的模板分为三个级别的, hugo 依据从上到下的顺序一次查找模板,直到找到为止.

-   特定页面的模板
-   应对某一类页面的模板
-   应对全站的模板: 存放在\_default 目录下面的 list.html 和 single.html 页面

# HomepageTemplate 查找顺序

## 首页模板的查找顺序

```
layouts/page/index.html  //page为type的默认值
layouts/page/home.html
layouts/page/list.html

layouts/index.html
layouts/home.html
layouts/list.html

layouts/_default/index.html
layouts/_default/home.html
layouts/_default/list.html
```

## 设置了 type 属性的首页模板的查找顺序

```
layouts/type的值/index.html
layouts/type的值/home.html
layouts/type的值/list.html

layouts/index.html
layouts/home.html
layouts/list.html

layouts/_default/index.html
layouts/_default/home.html
layouts/_default/list.html
```

_如果要设置首页的 type 属性, 需要在 content 目录下面添加\_index.md 文件, 并设置 FromtMatter 中的 type 属性, 同时在 layouts 目录下面创建和 type 属性的值相同的目录名._

# SingePageTemplate 查找顺序

## section 下单页面模板的查找顺序

```
layouts/section的值/layout的值.html
layouts/section的值/single.html

layouts/_default/single.html
```

## 非 section 下单页面模板的查找顺序

```
layouts/page/layout的值.html
layouts/page/single.html  //type的默认值

layouts/_default/single.html
```

## 设置了 type 属性的单页面模板的查找顺序

```
layouts/type的值/layout的值.html
layouts/type的值/single.html

layouts/_default/single.html
```

# SectionTemplate 的查找顺序

section template 为列表类型的模板, 用来展示 section tree 中某个的节点文章列表, Kind 可以轻松地与模板中的 where 函数结合使用，以创建种类特定的内容列表.

## section 模板的查找顺序

```
layouts/section的值/section的值.html
layouts/section的值/section.html
layouts/section的值/list.html

layouts/section/section的值.html
layouts/section/section.html
layouts/section/list.html

layouts/_default/section的值.html
layouts/_default/section.html
layouts/_default/list.html
```

## 设置 type 的 section 模板的查找顺序

```
layouts/type的值/section的值.html
layouts/type的值/section.html
layouts/type的值/list.html

layouts/section的值/section的值.html
layouts/section的值/section.html
layouts/section的值/list.html

layouts/section/section的值.html
layouts/section/section.html
layouts/section/list.html

layouts/_default/section的值.html
layouts/_default/section.html
layouts/_default/list.html
```

# TaxonomyTemplate 的查找顺序

tags 和 categories 是 hugo 默认会创建的两种分类, 如果要手工创建分类可以在 config 文件中配置

```
[taxonomies]
  category = "categories"
  tag = "tags"
```

如果不希望 hugo 创建任何分类, 配置 config 中的 disableKinds 属性

```
disableKinds = ["taxonomy", "taxonomyTerm"]
```

taxonomy list(某一分类的文章列表)), taxonomy terms list(所有的分类)

## Taxonomy Terms List Pages

```
layouts/categories/category.terms.html
layouts/categories/terms.html
layouts/categories/list.html

layouts/taxonomy/category.terms.html
layouts/taxonomy/terms.html
layouts/taxonomy/list.html

layouts/category/category.terms.html
layouts/category/terms.html
layouts/category/list.html

layouts/_default/category.terms.html
layouts/_default/terms.html
layouts/_default/list.html
```

## Taxonomy List Pages

```
layouts/categories/category.html
layouts/categories/taxonomy.html
layouts/categories/list.html

layouts/taxonomy/category.html
layouts/taxonomy/taxonomy.html
layouts/taxonomy/list.html

layouts/category/category.html
layouts/category/taxonomy.html
layouts/category/list.html

layouts/_default/category.html
layouts/_default/taxonomy.html
layouts/_default/list.html
```

# 基础模板--Baseof.html

基础模板页的文件名字为 `baseof.html` 或 `<TYPE>-baseof.html`
在基础模板页中使用 `block` 定义了一个占位符, 当模板页使用了一个基础模板页时, 模板页的解析后的内容会嵌入到基础模板页面中 `block` 的位置

示例:
baseof.html 的内容如下

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>baseof基础模板页</title>
    </head>
    <body>
        <div id="content">
            {{- block "main" . }}{{- end }}
            <!-- . 点表示从基础模板页面传递到模板页面的变量 -->
        </div>
    </body>
</html>
```

网站首页引用基础模板页

```html
<!-- block定义的占位符是 'main', 所以这里需要定义名为 'main'的模板 -->
{{- define "main" -}}
<section id="posts" class="posts">
    被define 和 end 包裹的内容会插入到baseof.html文件的{{- block "main" . }}{{-
    end }}位置.
</section>
{{- end -}}
```

除了要在基础模板页使用 block 外, 基础模板页的 命名 和 存放位置 也要征询一定的规制

### 基础模板页的存放位置及命名

```
/layouts/section/<TYPE>-baseof.html
/themes/<THEME>/layouts/section/<TYPE>-baseof.html

/layouts/<TYPE>/baseof.html
/themes/<THEME>/layouts/<TYPE>/baseof.html

/layouts/section/baseof.html
/themes/<THEME>/layouts/section/baseof.html

/layouts/_default/<TYPE>-baseof.html
/themes/<THEME>/layouts/_default/<TYPE>-baseof.html

/layouts/_default/baseof.html
/themes/<THEME>/layouts/_default/baseof.html
```

# 自定义数据 Hugo 中的数据库

## data 目录

在 data 目录下创建 `json | yaml | toml` 格式的文件. 通过 .Site.Data 变量以 MPA 的方式访问这些数据. 也就是 MAP 的结构组织目录和文件名的. 具体的可以输出 `.Site.Data` 来查看

例如在 data 目录中存放了两个用户的信息:

```
data/users/jim.toml
data/users/tom.toml
```

文件的内容为:

```toml
name = "Jim"
age = 22
address = [
    "地址一",
    "地址二"
]
```

在模板中使用这些数据:

```HTML
{{$users := .Site.Data.users}}
{{$user_jim := .Site.Data.users.jim}}
<div>jim.tom文件中的姓名: {{$user_jim.name}}</div>
<div>输出users目录下所有文件的内容: {{$users}}</div>
```

## 外源数据

hugo 还可以通过 getJSON 和 getCSV 两个函数加载外部数据, 这两个函数是模板函数. 在外部数据加载完成以前, hugo 会暂停渲染模板文件.

语法:

```HTML
{{ $dataJ := getJSON "url" }}
{{ $dataC := getCSV "separator" "url" }}
```

带可变参数语法:

```HTML
{{ $dataJ := getJSON "url prefix" "arg1" "arg2" "arg n" }}
{{ $dataC := getCSV  "separator" "url prefix" "arg1" "arg2" "arg n" }}
```

getCSV 的第一个参数为分隔符.

示例:

```HTML
  <table>
    <thead>
      <tr>
      <th>Name</th>
      <th>Position</th>
      <th>Salary</th>
      </tr>
    </thead>
    <tbody>
    {{ $url := "https://example.com/demo.csv" }}
    {{ $sep := "," }}
    {{ range $i, $r := getCSV $sep $url }}
      <tr>
        <td>{{ index $r 0 }}</td>
        <td>{{ index $r 1 }}</td>
        <td>{{ index $r 2 }}</td>
      </tr>
    {{ end }}
    </tbody>
  </table>
```

