---
title: Django跨域请求问题
date: 2018-11-01 20:11:17
tags:
---

同源：是指相同的协议、域名、端口，三者都相同才属于同源。

同源策略：浏览器处于安全考虑，在全局层面禁止了页面加载或执行与自身来源不同的域的任何脚本，站外其他来源的脚本同页面的交互则被严格限制。

跨域由于浏览器同源策略，凡是发送请求url的协议、域名、端口三者之间任意一与当前页面地址不同即为跨域

这儿由于通过rest_framework设置了前后分离，因此为了解决跨域问题重写了相应的方法

views.py

```python
import json

from django.http import HttpResponse
from django.shortcuts import render
from rest_framework import mixins, viewsets, status
from rest_framework.response import Response

from app.models import Article
from app.serializers import ArticleSerializer


class ArticleView(viewsets.GenericViewSet,
				  mixins.ListModelMixin,
				  mixins.DestroyModelMixin,
				  mixins.CreateModelMixin,
				  mixins.RetrieveModelMixin,
				  mixins.UpdateModelMixin,
				  ):
	# 查询数据
	queryset = Article.objects.all()

	# 序列化
	serializer_class = ArticleSerializer



	def list(self, request, *args, **kwargs):
		queryset = self.filter_queryset(self.get_queryset())
		page = self.paginate_queryset(queryset)
		if page is not None:
			serializer = self.get_serializer(page, many=True)
			return self.get_paginated_response(serializer.data)

		serializer = self.get_serializer(queryset, many=True)

		#这儿重写了list方法，并把headers重新赋值。就可以解决跨域问题
		return Response(serializer.data,headers={'Access-Control-Allow-Origin':'*',
												 'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
												 'Access-Control-Max-Age': '1000',
												 'Access-Control-Allow-Headers': '*'})

	def destroy(self, request, *args, **kwargs):
		instance = self.get_object()
		self.perform_destroy (instance)
		return Response (status=status.HTTP_204_NO_CONTENT)

	def perform_destroy(self, instance):
		instance.is_delete = 1
		instance.save()


def all_article(request):
	if request.method == 'GET':
		return render(request, 'all_article.html')
```

