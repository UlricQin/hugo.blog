+++
date = "2013-12-30T18:15:35+08:00"
menu = "main"
tags = ["python", "flask", "pagination", "web"]
title = "Python中通用分页方法（基于Flask）"

+++

最近因为项目需要，需要写一些Python的代码，过程中需要有一个分页功能，就写了一个通用分页代码，分享给大家：）

# 简单介绍
项目基于Flask框架，使用Jinja2模板引擎，我把大部分分页类代码都放到了一个macro中啦，这样一来，只要在页面中引入一下，就自动添加了分页功能。

# 如何使用
首先在templates目录下创建一个blocks.html，里边内容如下：

	{% macro pager(_uri, total, limit, curr_page, left=3, right=7) -%}
	{% if '?' in _uri %}
	{% set uri = _uri + '&' %}
	{% else %}
	{% set uri = _uri + '?' %}
	{% endif %}
	{% if total > limit %}
	{% set page_num = total//limit if total%limit==0 else total//limit+1 %}
	{% set pre_page = curr_page - 1 %}
	{% set pre_page = 1 if pre_page < 1 else pre_page %}
	{% set next_page = curr_page + 1 %}
	{% set next_page = page_num if next_page > page_num else next_page %}
	{% set begin_idx = 1 if curr_page <= 3 else curr_page - left %}
	{% set end_idx = begin_idx + right %}
	{% set end_idx = page_num if end_idx > page_num else end_idx %}
	<ul class="pagination pagination-sm">
	    {%if curr_page > 1 %}
	    <li><a href="{{uri}}p=1">首页</a></li>
	    <li><a href="{{uri}}p={{pre_page}}">&lt;</a></li>
	    {%else%}
	    <li class="disabled"><a>首页</a></li>
	    <li class="disabled"><a>&lt;</a></li>
	    {%endif%}
	    {% for idx in range(begin_idx, end_idx+1) %}
	    <li class="{%if curr_page == idx %}active{%endif%}">
	    <a href="{{uri}}p={{idx}}">{{idx}}</a>
	    </li>
	    {% endfor %}
	    {%if curr_page < page_num %}
	    <li><a href="{{uri}}p={{next_page}}">&gt;</a></li>
	    <li><a href="{{uri}}p={{page_num}}">尾页</a></li>
	    {%else%}
	    <li
	    class="disabled"><a>&gt;</a></li>
	    <li
	    class="disabled"><a>尾页</a></li>
	    {%endif%}
	</ul>
	{%endif%}
	{%- endmacro %}

说一下pager的几个参数的意思：
* _uri：点击分页链接的时候请求到哪里（这个对于分页的同时还希望保持住页面查询条件，url参数等比较方便）
* total 记录总数
* limit 每页显示的记录数
* curr_page 当前页码
* left 当前页码左边显示几个页码链接
* right 当前页码右边显示几个页码链接

然后你哪里需要分页，就把这个pager的macro引入进去：

	{% import "blocks.html" as blocks %}
	{{blocks.pager('/user/list?query='+query, p.total, p.limit, p.page)}}

简单解释一下，这是个user列表页，支持查询功能，调用blocks.pager的时候需要在url中拼接一个query，这是查询条件，需要一个p，这是一个dict，里边包含了一些分页用的基础数据，query和p都是从后端接口返回的：

	@app.route('/user/list')
	def list():
	    # 分页链接会自动添加p这个参数，表示页码
	    page = request.args.get('p', '1')
	    if not page:
	        page = 1
	    limit = request.args.get('limit', '10')
	    if not limit:
	        limit = 10

	    query = request.args.get('query', '')

	    users, total = User.list(page, limit, query)
	    pager = {'total': int(total), 'limit':int(limit), 'curr_page': int(page)}
	    return render_template('user/list.html', users=users, query=query, p=pager)

O了。至于分页的样式，我直接使用的bootstrap（ https://gist.github.com/UlricQin/11029085 ），如果你是自己写css的，那就自己搞起啦，样子如下：

![](http://beego-blog.qiniudn.com/static/uploads/editor/1397805745.png)
