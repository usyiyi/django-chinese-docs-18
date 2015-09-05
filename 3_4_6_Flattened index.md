# 基于类的通用视图 —— 索引 #

这里的索引提供基于类的视图的另外一种组织形式。对于每个视图，在类继承树中有效的属性和方法都显示在该视图的下方。按照行为进行组织的文档，参见基于类的视图。

## 简单的通用视图 ##

### View ###

`class View`

属性 （以及访问它们的方法）：

+ `http_method_names`

方法

+ `as_view()`
+ `dispatch()`
+ `head()`
+ `http_method_not_allowed()`

### TemplateView ###

`class TemplateView`

属性 （以及访问它们的方法）：

+ `content_type`
+ `http_method_names`
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `head()`
+ `http_method_not_allowed()`
+ `render_to_response()`

### RedirectView ###

`class RedirectView`

属性 （以及访问它们的方法）：

+ `http_method_names`
+ `pattern_name`
+ `permanent`
+ `query_string`
+ `url` [`get_redirect_url()`]

方法

+ `as_view()`
+ `delete()`
+ `dispatch()`
+ `get()`
+ `head()`
+ `http_method_not_allowed()`
+ `options()`
+ `post()`
+ `put()`

## 明细视图 ##

### DetailView ###

`class DetailView`

属性 （以及访问它们的方法）：

+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `http_method_names`
+ `model`
+ `pk_url_kwarg`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `slug_field` [`get_slug_field()`]
+ `slug_url_kwarg`
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_field`
+ `template_name_suffix`

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_object()`
+ `head()`
+ `http_method_not_allowed()`
+ `render_to_response()`

## 清单视图 ##

### ListView ###

`class ListView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `http_method_names`
+ `model`
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_paginator()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

## 编辑视图 ##

### FormView ###

`class FormView`

属性 （以及访问它们的方法）：

+ `content_type`
+ `form_class` [`get_form_class()`]
+ `http_method_names`
+ `initial` [`get_initial()`]
+ `prefix` [`get_prefix()`]
+ `response_class` [`render_to_response()`]
+ `success_url` [`get_success_url()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]

方法

+ `as_view()`
+ `dispatch()`
+ `form_invalid()`
+ `form_valid()`
+ `get()`
+ `get_context_data()`
+ `get_form()`
+ `get_form_kwargs()`
+ `http_method_not_allowed()`
+ `post()`
+ `put()`

### CreateView ###

`class CreateView`

属性 （以及访问它们的方法）：

+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `fields`
+ `form_class` [`get_form_class()`]
+ `http_method_names`
+ `initial` [`get_initial()`]
+ `model`
+ `pk_url_kwarg`
+ `prefix` [`get_prefix()`]
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `slug_field` [`get_slug_field()`]
+ `slug_url_kwarg`
+ `success_url` [`get_success_url()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_field`
+ `template_name_suffix`

方法

+ `as_view()`
+ `dispatch()`
+ `form_invalid()`
+ `form_valid()`
+ `get()`
+ `get_context_data()`
+ `get_form()`
+ `get_form_kwargs()`
+ `get_object()`
+ `head()`
+ `http_method_not_allowed()`
+ `post()`
+ `put()`
+ `render_to_response()`

### UpdateView ###

`class UpdateView`

属性 （以及访问它们的方法）：

+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `fields`
+ `form_class` [`get_form_class()`]
+ `http_method_names`
+ `initial` [`get_initial()`]
+ `model`
+ `pk_url_kwarg`
+ `prefix` [`get_prefix()`]
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `slug_field` [`get_slug_field()`]
+ `slug_url_kwarg`
+ `success_url` [`get_success_url()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_field`
+ `template_name_suffix`

方法

+ `as_view()`
+ `dispatch()`
+ `form_invalid()`
+ `form_valid()`
+ `get()`
+ `get_context_data()`
+ `get_form()`
+ `get_form_kwargs()`
+ `get_object()`
+ `head()`
+ `http_method_not_allowed()`
+ `post()`
+ `put()`
+ `render_to_response()`

### DeleteView ###

`class DeleteView`

属性 （以及访问它们的方法）：

+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `http_method_names`
+ `model`
+ `pk_url_kwarg`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `slug_field` [`get_slug_field()`]
+ `slug_url_kwarg`
+ `success_url` [`get_success_url()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_field`
+ `template_name_suffix`

方法

+ `as_view()`
+ `delete()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_object()`
+ `head()`
+ `http_method_not_allowed()`
+ `post()`
+ `render_to_response()`

## 基于日期的视图 ##

### ArchiveIndexView ###

`class ArchiveIndexView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `http_method_names`
+ `model`
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_date_list()`
+ `get_dated_items()`
+ `get_dated_queryset()`
+ `get_paginator()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

### YearArchiveView ###

`class YearArchiveView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `http_method_names`
+ `make_object_list` [`get_make_object_list()`]
+ `model`
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`
+ `year` [`get_year()`]
+ `year_format` [`get_year_format()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_date_list()`
+ `get_dated_items()`
+ `get_dated_queryset()`
+ `get_paginator()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

### MonthArchiveView ###

`class MonthArchiveView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `http_method_names`
+ `model`
+ `month` [`get_month()`]
+ `month_format` [`get_month_format()`]
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`
+ `year` [`get_year()`]
+ `year_format` [`get_year_format()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_date_list()`
+ `get_dated_items()`
+ `get_dated_queryset()`
+ `get_next_month()`
+ `get_paginator()`
+ `get_previous_month()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

### WeekArchiveView ###

`class WeekArchiveView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `http_method_names`
+ `model`
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`
+ `week` [`get_week()`]
+ `week_format` [`get_week_format()`]
+ `year` [`get_year()`]
+ `year_format` [`get_year_format()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_date_list()`
+ `get_dated_items()`
+ `get_dated_queryset()`
+ `get_paginator()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

### DayArchiveView ###

`class DayArchiveView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `day` [`get_day()`]
+ `day_format` [`get_day_format()`]
+ `http_method_names`
+ `model`
+ `month` [`get_month()`]
+ `month_format` [`get_month_format()`]
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`
+ `year` [`get_year()`]
+ `year_format` [`get_year_format()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_date_list()`
+ `get_dated_items()`
+ `get_dated_queryset()`
+ `get_next_day()`
+ `get_next_month()`
+ `get_paginator()`
+ `get_previous_day()`
+ `get_previous_month()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

### TodayArchiveView ###

`class TodayArchiveView`

属性 （以及访问它们的方法）：

+ `allow_empty` [`get_allow_empty()`]
+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `day` [`get_day()`]
+ `day_format` [`get_day_format()`]
+ `http_method_names`
+ `model`
+ `month` [`get_month()`]
+ `month_format` [`get_month_format()`]
+ `ordering` [`get_ordering()`]
+ `paginate_by` [`get_paginate_by()`]
+ `paginate_orphans` [`get_paginate_orphans()`]
+ `paginator_class`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_suffix`
+ `year` [`get_year()`]
+ `year_format` [`get_year_format()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_date_list()`
+ `get_dated_items()`
+ `get_dated_queryset()`
+ `get_next_day()`
+ `get_next_month()`
+ `get_paginator()`
+ `get_previous_day()`
+ `get_previous_month()`
+ `head()`
+ `http_method_not_allowed()`
+ `paginate_queryset()`
+ `render_to_response()`

### DateDetailView ###

`class DateDetailView`

属性 （以及访问它们的方法）：

+ `allow_future` [`get_allow_future()`]
+ `content_type`
+ `context_object_name` [`get_context_object_name()`]
+ `date_field` [`get_date_field()`]
+ `day` [`get_day()`]
+ `day_format` [`get_day_format()`]
+ `http_method_names`
+ `model`
+ `month` [`get_month()`]
+ `month_format` [`get_month_format()`]
+ `pk_url_kwarg`
+ `queryset` [`get_queryset()`]
+ `response_class` [`render_to_response()`]
+ `slug_field` [`get_slug_field()`]
+ `slug_url_kwarg`
+ `template_engine`
+ `template_name` [`get_template_names()`]
+ `template_name_field`
+ `template_name_suffix`
+ `year` [`get_year()`]
+ `year_format` [`get_year_format()`]

方法

+ `as_view()`
+ `dispatch()`
+ `get()`
+ `get_context_data()`
+ `get_next_day()`
+ `get_next_month()`
+ `get_object()`
+ `get_previous_day()`
+ `get_previous_month()`
+ `head()`
+ `http_method_not_allowed()`
+ `render_to_response()`

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Flattened index](https://docs.djangoproject.com/en/1.8/ref/class-based-views/flattened-index/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
