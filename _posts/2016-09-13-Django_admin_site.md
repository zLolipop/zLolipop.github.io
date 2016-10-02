---
title: "Django Admin 站点"
date: 2016-09-13 14:31:20
categories: Django
---

有关Django admin 站点的一点笔记


-----------


## ModelAdmin 对象

ModelAdmin 类是模型在Admin 界面中的表示形式。通常，将它们在你的应用中的名为admin.py的文件里。让我们来看一个关于ModelAdmin类非常简单的例子:

{% highlight python %}

from django.contrib import admin
from myproject.myapp.models import Author
class AuthorAdmin(admin.ModelAdmin):
    pass
admin.site.register(Author, AuthorAdmin)

{% endhighlight %}

在上面的例子中AuthorAdmin并没有定义任何自定义的值,这时Django将使用默认的Admin界面,如果对默认的Admin界面足够满意,
那么完全没有必要自己定义*ModelAdmin*对象,可以直接注册模型类而无需提供*ModelAdmin*的描述.这样的话上述例子可以简化成:

{% highlight python%}

from django.contrib import admin
from myproject.myapp.models import Author

admin.site.register(Author)

{% endhighlight %}

# 创建超级用户

{% highlight python%}

python manage.py createsuperuser

{% endhighlight %}