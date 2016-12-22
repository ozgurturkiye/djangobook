# Django Views: Dynamic Content
# Django Dinamik(hareketli) içerikler

## İkinci view'imiz: Dinamik içerik

Bir önceki örnekte yaptığımız "Hello world" view'i statik olarak kabul ediyoruz, çünkü ne zaman /hello/ yu görüntülesek hep aynı hep aynı değişen bişey yok. 
İkinci view olarak daha hareketli bişeyler yapalım :) Web sayfamız o anki sunucu zamanını göstersin mesela. Bu güzel bir başlangıç örneği olur çünkü ne veritabanı kullanıyoruz nede kullanıcıdan bir veri girişi alıyoruz. Sadece sunucu saatini gösteriyoruz  ;)

Şimdi ihtiyacımız olan iki şey var.

1. Sayfa görüntülendiği andaki zamanı hesaplamak
2. Hesaplanan değeri HttpResponse içeriği olarak göndermek.

O zaman `views.py` dosyasını düzenleyelim.
```python
from django.http import HttpResponse
import datetime

def hello(request):
    return HttpResponse("Hello world")

def current_datetime(request):
    now = datetime.datetime.now()
    html = "It is now %s." % now
    return HttpResponse(html)
```

Hadi yapılanları adım adım inceleyelim :)

1. datetime modülünü import ettik -  `import datetime`
2. Yeni bir `current_datetime` fonksiyonu tanımladık.
3. current_datetime fonksiyonu içerisindeki `now` değişkenine o anki zamanı atadık.
4. `html` değişkenine `string`(karakter dizisi) olarak zamanı aktardık.
5. Son olarak view, HttpResponse nesnesini içerisindeki zaman bilgisiyle geri döndürdü.

Tabi işler bitti mi bitmedi ;)
views.py dosyasına gerekli eklemeleri yaptıktan sonra urls.py dosyasına URLpattern'i ekliyoruz ki Django hangi url de hangi view'i handle edeceğini(yani işleyeceğini) bilsin.
Bizim için an itibari ile şöyle birşey olmalı urls.py

```python
from django.conf.urls import include, url
from django.contrib import admin
from mysite.views import hello, current_datetime

    urlpatterns = [
        url(r'^admin/', include(admin.site.urls)),
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
    ]

 ```
 
 Bitti mi tabii ki bitmedi.
 * Önce bir sunucuyu ayağa kaldıralım `python3 manage.py runserver 0.0.0.0:8000`
 * Sonra da web tarayıcıya url'yi girelim `http://127.0.0.1:8000/time/` tabi biz yerel makinada çalışmadığımız için sunucu adımızı giriyoruz ip adresi yerine. Şimdi burada web sitesi reklamı yapmayalım :)
 
 ## URLconfs and Loose Coupling
 ## URLconfs ve Gevşekçe bağlaşım(Türkçesi bir komik oldu buranın)
 
 
 
