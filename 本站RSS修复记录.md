---
title: "本站RSS修复记录"
description: ""
keywords: ""
date: 2024-12-30T13:52:30+08:00
lastmod: 2024-12-30T13:52:30+08:00
---

## 本站RSS修复记录
发现本站没有RSS订阅的功能，在互联网上搜索到了一篇解决文章

原文地址：https://xuanwo.io/2018/04/08/hugo-rss-output-all-content/

大概意思就是要参考官网给的几个位置，选一个指定位置，新增一个RSS的模板即可

对方的模板直接就能用，记录在下面"layouts/index.rss.xml",最后生成rss的地址为“站点根/rss.xml”貌似名字还和配置有关
```xml
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if eq  .Title  .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} on {{ end }}{{ .Site.Title }}{{ end }}</title>
    <link>{{ .Permalink }}</link>
    <description>Recent content {{ if ne  .Title  .Site.Title }}{{ with .Title }}in {{.}} {{ end }}{{ end }}on {{ .Site.Title }}</description>
    <generator>Hugo -- gohugo.io</generator>{{ with .Site.LanguageCode }}
    <language>{{.}}</language>{{end}}{{ with .Site.Author.email }}
    <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
    <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
    <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
    <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
    {{ with .OutputFormats.Get "RSS" }}
        {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{ end }}
    {{ range first 10 .Data.Pages }}
    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
      {{ with .Site.Author.email }}<author>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</author>{{end}}
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Content | html }}</description>
    </item>
    {{ end }}
  </channel>
</rss>
```

值得记录的是这是原博客作者自己的更改结果，目的是能够直接看到全部文章内容而不是展示一个概要，剩下的内容还需要点击链接。目前我也想展示全部内容，不过还是记录一下原版的内容，注意下面的不是完整内容，需要结合上面的完整模板自行补全
```xml
{{ range .Data.Pages }}
<item>
  <title>{{ .Title }}</title>
  <link>{{ .Permalink }}</link>
  <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
  {{ with .Site.Author.email }}<author>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</author>{{end}}
  <guid>{{ .Permalink }}</guid>
  <description>{{ .Summary | html }}</description>
</item>
{{ end }}
```
