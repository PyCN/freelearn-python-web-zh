# 第十章：扩展 Django

到目前为止，我们已经走了很长的路，涉及了大量与 Django 功能相关的代码和基本概念。在本章中，我们将更多地讨论 Django，但我们将简要讨论不同的参数，例如自定义标签、过滤器、子框架、消息系统等。以下是本章将涉及的主题：

+   自定义模板标签和过滤器

+   基于类的通用视图

+   贡献的子框架

+   消息系统

+   订阅系统

+   用户分数

# 自定义模板标签和过滤器

Django 模板系统配备了许多模板标签和过滤器，使编写模板变得简单灵活。但是，有时您可能希望使用自己的标签和过滤器扩展模板系统。当您发现自己多次重复相同的标签结构时，希望将结构包装在单个标签中，甚至希望添加到模板系统中的过滤器时，通常会发生这种情况。

猜猜？Django 已经允许您这样做，而且这也很容易！您基本上只需向应用程序添加一个名为**templatetags**的新包，并将包含标签和过滤器的模块放入其中。让我们通过添加一个将字符串大写的过滤器来学习这一点。在`mytweets`父文件夹中添加一个`templatetags`文件夹，并在其中放置一个名为`__init__.py`的空文件，以便 Python 将该文件夹视为包。现在，在其中创建一个名为`mytweet_filters`的模块。我们将在此模块中编写我们的过滤器。以下是目录结构的示例：

```py
templatetags/
  |-- __init__.py
  -- mytweet_filters.py
```

现在，将以下代码添加到`mytweet_filters.py`文件中：

```py
  from django import template
  register = template.Library()

  @register.filter
  def capitalize(value):
    return value.capitalize()
```

`register`变量是一个对象，可用于向模板系统引入新的标签和过滤器。在这里，我们使用`register.filter`装饰器将 capitalize 函数添加为过滤器。

要在模板中使用新的过滤器，请在模板文件的开头放入以下行：

```py
{% load mytweet_filters %}
```

然后，您可以像使用 Django 提供的任何其他过滤器一样使用新的过滤器：

```py
Hi {{ name|capitalize }}!
```

添加自定义模板标签的工作方式与过滤器类似。基本上，您定义方法来处理标签，然后注册标签以使其可用于模板。这个过程稍微复杂一些，因为标签可能比过滤器更复杂。有关自定义模板标签的更多信息，请参阅 Django 在线文档。

在编写自定义过滤器时，您必须注意 Django 的自动转义行为。可以传递给过滤器的字符串有三种类型：

+   **原始字符串**：此字符串是通过`str`命令准备的，或者是用 unicode 形成的。如果启用了自动转义，它们将自动转义。

+   **安全字符串**：这些字符串是标记为免受进一步转义的字符串。它们不需要进一步转义。要将输出标记为安全字符串，请使用`django.utils.safestring.mark_safe()`模块。

+   **标记为“需要转义”的字符串**：顾名思义，它们始终需要转义。

# 基于类的通用视图

在使用 Django 时，您会注意到无论您正在处理哪个项目，总是需要某些类型的视图。因此，Django 配备了一组可在任何项目中使用的视图。这些视图称为**通用视图**。

Django 为以下目的提供了通用视图：

+   为任务创建简单的视图，例如重定向到另一个 URL 或呈现模板

+   列出和形成详细视图以显示数据模型中的对象-这些视图类似于管理页面显示数据模型的列表和详细页面

+   生成基于日期的存档页面；这对博客特别有用

+   创建，编辑和删除数据模型中的对象

Django 的基于类的视图可以通过定义子类或直接在 URL 配置中传递参数来配置。

子类充满了消除重写常见情况模板的约定。当您使用子类时，实际上可以通过提供新值来覆盖主类的属性或方法：

```py
# app_name/views.py
from django.views.generic import TemplateView

class ContactView(TemplateView):
  template_name = "contact.html"
```

我们还将在`urls.py`文件中添加其条目以进行重定向：

```py
# project/urls.py
from django.conf.urls.defaults import *
from some_app.views import ContactView

urlpatterns = patterns('',
  (r'^connect/', ContactView.as_view()),
)
```

有趣的是，我们可以通过文件更改来实现相同的效果，并且只需在`urls.py`文件中添加以下内容即可：

```py
from django.conf.urls.defaults import *
from django.views.generic import TemplateView

urlpatterns = patterns('',
  (r'^contact/', TemplateView.as_view(template_name="contact.html")),
)
```

# 贡献的子框架

`django.contrib`包含 Django 的标准库。我们在本书的前几章中使用了该软件包中的以下子框架：

+   `admin`: 这是 Django 管理界面

+   `auth`: 这是用户认证系统

+   `sessions`: 这是 Django 会话框架

+   `syndication`: 这是提要生成框架

这些子框架极大地简化了我们的工作，无论我们是创建注册和认证功能，构建管理页面，还是为我们的内容提供提要。`django.contrib`包是 Django 的一个非常重要的部分。了解其子包及如何使用它们将为您节省大量时间和精力。

本节将为您提供有关此软件包中其他框架的简要介绍。您不会深入了解如何使用每个框架，但您将学到足够的知识以了解何时使用框架。一旦您想在项目中使用框架，您可以阅读在线文档以了解更多信息。

# Flatpages

Web 应用程序可能包含静态页面。例如，您的网站可能包括一组很少更改的帮助页面。Django 提供了一个名为**flatpages**的应用程序来提供静态页面。该应用程序非常简单；它为您提供了一个数据模型，用于存储有关每个页面的各种信息，包括以下内容：

+   URL

+   标题

+   内容

+   模板名称

+   查看页面是否需要注册

要使用该应用程序，您只需在`settings.py`文件中的`INSTALLED_APPS`变量中启用它，并将其中间件添加到`MIDDLEWARE_CLASSES`变量中。之后，您可以使用 flatpages 应用程序提供的数据模型存储和管理静态页面。

## 人性化

**humanize**应用程序提供了一组过滤器，以为您的页面增添人性化的触感。

以下是可用过滤器的列表：

+   **apnumber**: 对于 1-9 的数字，它返回拼写的数字。否则，它返回数字。换句话说，1 变成'one'，9 变成'nine'，以此类推，而 10 保持为 10。

+   **intcomma**: 这接受一个整数并将其转换为带有逗号的字符串，例如：

```py
4500 becomes 4,500.
45000 becomes 45,000.
450000 becomes 450,000.
4500000 becomes 4,500,000.
```

+   **intword**: 这将整数转换为易于阅读的形式，例如：

1000000 变成 1.0 百万。

```py
1200000 becomes 1.2 million.
1200000000 becomes 1.2 billion.
```

+   **naturalday**: 基于日期所在的范围，如果给定日期在*(+1,0,-1)*范围内，则分别显示日期为"明天"，"今天"和"昨天"，例如，（如果今天是 2007 年 1 月 26 日）：

```py
25 Jan 2007 becomes yesterday.
26 Jan 2007 becomes today.
27 Jan 2007 becomes tomorrow.
```

+   **naturaltime**: 这返回一个表示事件日期发生多少秒、分钟或小时前的字符串，例如，（如果现在是 2007 年 1 月 26 日 16:30:00）：

```py
26 Jan 2007 16:30:00 becomes now.
26 Jan 2007 16:29:31 becomes 29 seconds ago.
26 Jan 2007 16:29:00 becomes a minute ago.
26 Jan 2007 16:25:35 becomes 4 minutes ago.
26 Jan 2007 15:30:29 becomes 59 minutes ago.
26 Jan 2007 15:30:01 becomes 59 minutes ago.
26 Jan 2007 15:30:00 becomes an hour ago.
26 Jan 2007 13:31:29 becomes 2 hours ago.
25 Jan 2007 13:31:29 becomes 1 day, 2 hours ago.
25 Jan 2007 13:30:01 becomes 1 day, 2 hours ago.
25 Jan 2007 13:30:00 becomes 1 day, 3 hours ago.
26 Jan 2007 16:30:30 becomes 30 seconds from now.
26 Jan 2007 16:30:29 becomes 29 seconds from now.
26 Jan 2007 16:31:00 becomes a minute from now.
26 Jan 2007 16:34:35 becomes 4 minutes from now.
26 Jan 2007 17:30:29 becomes an hour from now.
26 Jan 2007 18:31:29 becomes 2 hours from now.
27 Jan 2007 16:31:29 becomes 1 day from now.
```

+   **ordinal**: 这将整数转换为序数形式。例如，1 变成'1st'，以此类推，每三个数字之间。

## Sitemap

**Sitemap**是一个生成站点地图的框架，这些站点地图是帮助搜索引擎索引器在您的站点上找到动态页面的 XML 文件。它告诉索引器页面的重要性以及更改频率。这些信息使索引过程更准确和高效。

站点地图框架允许您用 Python 代码表示上述信息，然后生成代表您网站站点地图的 XML 文档。这涵盖了`django.contrib`包中最常用的子框架。该包包含一些不像前面那些重要的附加应用程序，并且会不时地更新新的应用程序。要了解`django.contrib`包中的任何应用程序，您可以随时阅读其在线文档。 

## 跨站点请求伪造保护

我们在第五章中讨论了如何防止两种类型的 Web 攻击，即 SQL 注入和跨站点脚本。Django 提供了对抗另一种称为跨站点请求伪造的攻击的保护。在这种攻击中，恶意站点试图通过欺骗在您网站上登录的用户来操纵您的应用程序，使其打开一个特制的页面。该页面通常包含 JavaScript 代码，试图向您的网站提交表单。CSRF 保护通过将一个令牌（即秘密代码）嵌入到所有表单中，并在提交表单时验证该令牌来工作。这有效地使 CSRF 攻击变得不可行。

要激活 CSRF 保护，您只需要将`'django.contrib.csrf.middleware.CsrfMiddleware'`参数添加到`MIDDLEWARE_CLASSES`变量中，这将透明地工作，以防止 CSRF 攻击。

# 消息系统

我们的应用允许用户将彼此添加为好友并监视好友的书签。虽然这两种形式的通信与我们的书签应用程序的性质有关，但有时用户希望灵活地向彼此发送私人消息。这个功能对于增强我们网站的社交方面特别有用。

消息系统可以以多种方式实现。它可以简单到为每个用户提供一个联系表单，当提交时，通过发送其内容到用户的电子邮件来工作。您已经拥有构建此功能组件所需的所有信息：

+   一个消息表单，其中包含主题的文本字段和消息正文的文本区域

+   显示用户消息表单的视图，并通过`send_mail()`函数将表单内容发送给用户

当允许用户通过您的网站发送电子邮件时，您需要小心以防止滥用该功能。在这里，您可以将联系表单限制为仅限已登录的用户或仅限好友。

实现消息系统的另一种方法是在数据库中存储和管理消息。这样，用户可以使用我们的应用程序发送和查看消息，而不是使用电子邮件。虽然这种方法更加与我们的应用程序绑定，因此可以使用户留在我们的网站上，但需要更多的工作来实现。然而，与之前的方法一样，您已经拥有实现这种方法所需的所有信息。这里需要的组件如下：

+   存储消息的数据模型。它应该包含发送者、接收者、主题和正文的字段。您还可以添加日期、阅读状态等字段。

+   创建消息的表单。需要主题和正文的字段。

+   列出可用消息的视图。

+   显示消息的视图。

上述列表只是实现消息系统的一种方式。例如，您可以将列表和消息视图合并为一个视图，或者提供一个视图来显示已发送的消息以及已接收的消息。可能性很多，取决于您希望该功能有多高级。

# 订阅系统

我们提供了几种 Web 订阅，使用户能够监视我们网站的更新。然而，一些用户可能仍然更喜欢通过电子邮件监视更新的旧方式。对于这些用户，您可能希望将电子邮件订阅系统实施到应用程序中。例如，您可以让用户在朋友发布书签时收到通知，或者在特定标签下发布书签时收到通知。

此外，您可以将这些通知分组并批量发送，以避免发送大量的电子邮件。此功能的实现细节在很大程度上取决于您希望它如何工作。它可以是一个简单的数据模型，用于存储每个用户订阅的标签。它将循环遍历所有订阅特定标签的用户，并在此标签下发布书签时向他们发送通知。然而，这种方法太基础，会产生大量的电子邮件。更复杂的方法可能涉及将通知存储在数据模型中，并在每天发送一封电子邮件。

# 用户评分

一些网站（如[Slashdot.org](http://Slashdot.org)和[reddit.com](http://reddit.com)）通过为每个用户分配一个分数来跟踪用户的活动。每当用户以某种方式为网站做出贡献时，该分数就会增加。用户的分数可以以各种方式利用。例如，您可以首先向最活跃的用户发布新功能，或者为活跃用户提供其他优势，这将激励其他用户更多地为您的网站做出贡献。

实施用户评分非常简单。您需要一个数据模型来在数据库中维护评分。之后，您可以使用 Django 模型 API 从视图中访问和操作评分。

# 总结

本章的目的是为您准备本书未涵盖的任务。它向您介绍了许多主题。当需要某种功能时，您现在知道在哪里寻找框架，以帮助您快速而干净地实施该功能。

本章还为您提供了一些想法，您可能希望将其实施到我们的书签应用程序中。致力于这些功能将为您提供更多的机会来尝试 Django 并扩展您对其框架和内部工作原理的了解。

在下一章中，我们将介绍各种数据库连接的方式，如 MySQL、NoSQL、PostgreSQL 等，这对于任何基于数据库的应用程序都是必需的。