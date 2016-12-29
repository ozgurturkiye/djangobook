# Templates in Views
# Views in içinde Şablonlar

Şablon sistemini kullanmanın temellerini öğrendiniz; Şimdi bu bilgileri bir görünüm oluşturmak için kullanalım.

Önceki bölümde başlattığımız mysite.views'deki current_datetime görünümünü hatırlayın. Şuna benziyor:

```python
from django.http import HttpResponse
import datetime

def current_datetime(request):

    now = datetime.datetime.now()
    html = "<html><body>It is now %s</body></html>" % now
    return HttpResponse(html)
```

