# Django 调试

在需要调试的程序前添加以下引用
```
import os
import django
 
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "master.settings")
django.setup()
```