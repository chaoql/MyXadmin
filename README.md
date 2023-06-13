# 使用xadmin-django2.2构建后台管理系统

> 由于原本的xadmin版本报错太多，难以与python3.x版本以及django2.2版本兼容，因此解决所有报错后创建此项目便于后续项目开发。

[toc]

## 配置xadmin

1. 下载源码到项目本地

2. 在setting的INSTALLED_APPS中添加crispy_forms、xadmin、reversion、crispy_bootstrap3和django.conf，并配置语言和时区。

   ```python
   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       'apps.users.apps.UsersConfig',
       'apps.courses.apps.CoursesConfig',
       'apps.operations.apps.OperationsConfig',
       'apps.organizations.apps.OrganizationsConfig',
       'crispy_forms',
       'xadmin.apps.XAdminConfig',
       'reversion',
       'crispy_bootstrap3',
       'django.conf',
   ]
   CRISPY_TEMPLATE_PACK = 'bootstrap3'
   
   LANGUAGE_CODE = 'zh-hans'  # 配置显示为中文
   
   TIME_ZONE = 'Asia/Shanghai'  # 配置时区
   USE_TZ = False
   ```

3. 安装xadmin的依赖

   ```python
   pip install -i https://pypi.doubanio.com/simple/ -r requirements.txt
   ```

4. 生成数据表

   ```python
   makemigrations
   
   migrate
   ```

5. 配置urls.py

   ```python
   from django.contrib import admin
   from xadmin.plugins import xversion
   from django.urls import path
   import xadmin
   xversion.register_models()
   xadmin.autodiscover()
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('xadmin/', xadmin.site.urls),
   ]
   ```

## 注入app的数据表

1. 在app目录下新建adminx.py文件，并编辑如下：

   ```python
   import xadmin
   from apps.courses.models import City
   
   
   class CourseAdmin(object):
       """
       为每个需要注入的数据表创建Admin函数
       """
       list_display = ['name', 'desc', 'detail', 'degree', 'learn_times', 'students']  # 定义列表页显示的字段
       search_fields = ['name', 'desc', 'detail', 'degree', 'students']  # 定义搜索的字段
       list_filter = ['name', 'teacher__name', 'desc', 'detail', 'degree', 'learn_times', 'students']  # 定义过滤器字段('外键__外键属性'：这种格式可以直接定位到外键属性)
       list_editable = ["degree", "desc"]  # 定义允许在列表中直接编辑的字段
   
   xadmin.site.register(Course, CourseAdmin)  # 注册数据表
   ```

2. 修改在xadmin网页中显示的该app名称（编辑app目录下的apps.py）

   ```python
   from django.apps import AppConfig
   
   
   class CoursesConfig(AppConfig):
       name = 'apps.courses'
       verbose_name = "课程管理"  # 别称
   ```

##  配置后台管理系统样式

```python
class GlobalSettings(object):
    site_title = "CW后台管理系统"  # 定义后台系统主题名称
    site_footer = "CW网站页脚"  # 定义后台系统网站页脚
    menu_style = "accordion"  # 左侧导航栏收起

class BaseSettings(object):
    enable_themes = True  # 允许更换主题皮肤配置
    use_bootswatch = True
    
xadmin.site.register(xadmin.views.CommAdminView, GlobalSettings)
xadmin.site.register(xadmin.views.BaseAdminView, BaseSettings)
```

