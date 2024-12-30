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

> 注意最后生成rss的地址为“站点根/rss.xml”貌似名字还和配置有关

我先是按照他的照搬，记录在下面"layouts/index.rss.xml"
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

最后结果是并没有效果，生成内容只会抓到一个post/的链接，没有抓到之后的文章链接，然后参照官网示例，并且模仿上述博客将`.Summary`改为了`.Content`为了能够显示全部内容，最终模板为

```xml
{{- $authorEmail := "" }}
{{- with site.Params.author }}
  {{- if reflect.IsMap . }}
    {{- with .email }}
      {{- $authorEmail = . }}
    {{- end }}
  {{- end }}
{{- end }}

{{- $authorName := "" }}
{{- with site.Params.author }}
  {{- if reflect.IsMap . }}
    {{- with .name }}
      {{- $authorName = . }}
    {{- end }}
  {{- else }}
    {{- $authorName  = . }}
  {{- end }}
{{- end }}

{{- $pctx := . }}
{{- if .IsHome }}{{ $pctx = .Site }}{{ end }}
{{- $pages := slice }}
{{- if or $.IsHome $.IsSection }}
{{- $pages = $pctx.RegularPages }}
{{- else }}
{{- $pages = $pctx.Pages }}
{{- end }}
{{- $limit := .Site.Config.Services.RSS.Limit }}
{{- if ge $limit 1 }}
{{- $pages = $pages | first $limit }}
{{- end }}
{{- printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\"?>" | safeHTML }}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if eq .Title .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{ . }} on {{ end }}{{ .Site.Title }}{{ end }}</title>
    <link>{{ .Permalink }}</link>
    <description>Recent content {{ if ne .Title .Site.Title }}{{ with .Title }}in {{ . }} {{ end }}{{ end }}on {{ .Site.Title }}</description>
    <generator>Hugo</generator>
    <language>{{ site.Language.LanguageCode }}</language>{{ with $authorEmail }}
    <managingEditor>{{.}}{{ with $authorName }} ({{ . }}){{ end }}</managingEditor>{{ end }}{{ with $authorEmail }}
    <webMaster>{{ . }}{{ with $authorName }} ({{ . }}){{ end }}</webMaster>{{ end }}{{ with .Site.Copyright }}
    <copyright>{{ . }}</copyright>{{ end }}{{ if not .Date.IsZero }}
    <lastBuildDate>{{ (index $pages.ByLastmod.Reverse 0).Lastmod.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
    {{- with .OutputFormats.Get "RSS" }}
    {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{- end }}
    {{- range $pages }}
    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .PublishDate.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
      {{- with $authorEmail }}<author>{{ . }}{{ with $authorName }} ({{ . }}){{ end }}</author>{{ end }}
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Content | transform.XMLEscape | safeHTML }}</description>
    </item>
    {{- end }}
  </channel>
</rss>
```

最后不要忘记将你的rss链接我的是‘/rss.xml’添加到首页代码中，这样其他人输入你的网址就能够直接获取到rss的链接进一步解析了
