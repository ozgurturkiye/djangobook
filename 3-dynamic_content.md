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
    html = "It is now {}:".format(now)
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
 ## URLconfs ve Gevşekçe bağlaşım(Türkçesi bir komik oldu buranın lakin bu bir yazılım geliştirme yaklaşımı)
 
 `Loose Coupling` URLconfs ve Django'nun arkasındaki temel felsefedir. Eğer yanlış çevirmiyorsam! Loose Coupling: değerleri birbiri ile değiştirilebilir yapmanın önemidir. Eğer iki parça kod `Loose Coupling`(yani gevşekçe:)) olarak bağlıysa, birinde yapılan değişiklik diğerini ya çok az etkiler ya hiç etkilemez. 
 Django URLconfs bu prensibe güzel bir örnektir. Django web uygulamalarında; URL tanımlaması ve view fonksiyonları Loose Coupling'e güzel bir örnektir.
 Küçük bir örnek: istediğin URL'i istediğin view'e bağla kimse etkilenmez ve kimse de bişey diyemez :D
 ```python
 urlpatterns = [
      url(r'^admin/', include(admin.site.urls)),
      url(r'^hello/$', hello),
      url(r'^time/$', current_datetime),
      url(r'^another-time-page/$', current_datetime),
]
```
## Your Third View: Dynamic URLs
## Üçüncü View: Dinamik URLs

Bir çok dinamik web uygulamasında URL'ler parametreleri taşır ki bunlar sayfaları gösterir. Örnek olarak bir kitabevinin her bir kitabının ayrı bir sayfası olduğunu düşünelim, örnek /books/243/ ve /books/78555/ gibi. Biz şimdi üçüncü view'imizi oluşturalım ve şimdiki zamanı ve sonrasını gösterebilsin. URL'ler şu şekilde olsun /time/plus/1/  ve /time/plus/2/ ve /time/plus/24/ gibi.

`urls.py` dosyasını aşağıdaki gibi düzenleyelim.

```python
from django.conf.urls import include, url
from django.contrib import admin
from mysite.views import hello, current_datetime, hours_ahead  
urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^hello/$', hello),
    url(r'^time/$', current_datetime),
    url(r'^time/plus/(\d{1,2})/$', hours_ahead),
]
```
* Son satırdaki `(\d{1,2})` regex ifadesini biraz inceleyelim :)
* Parantezler () eşleşen metinden veri almayı sağlar. Yani parantez içindeki veriyi alıyoruz.
* Regex'te `\d` ifadesi sayı almayı `\d+` ifadesi bir veya daha fazla sayı almayı `\d{1,2}` ifadesi ise bir veya iki(0-99 arası) sayı almayı almayı sağlar.
 
Şimdi `view.py` dosyamızı düzenleyelim ve view'imizi oluşturalım.

```python
from django.http import Http404, HttpResponse
import datetime

def hours_ahead(request, offset):
    try:
        offset = int(offset)
    except ValueError:
        raise Http404()
    
    dt = datetime.datetime.now() + datetime.timedelta(hours = offset)
    html = "{} saat içinde, saat <b> {} </b> olacak".format(offset, dt)
    return HttpResponse(html)
```
Kodu daha yakından inceleyelim :)

* view fonksiyonumuz `hours_ahead` iki ayrı parametre alıyor `request` ve `offset`
* `request` zaten HttpResponse nesnesi. Hatırlatmakta fayda var herbir view her zaman HttpResponse nesnesini ilk parametre olarak alır.
* `offset` ise URLpattern'den yakalanan string(karakter dizisi)dir. Örnek olarak eğer requested URL(Türkçesini yazmıyorum) `/time/plus/3/` ise '3' gelir. Not alınız: Yakalanan değerler her zaman Unicode bir nesne döndürür. Sayı döner sanmayın!!!
* URLpattern'den regex ile sayısal yakalana fakat Unicode nesnesi olarak gönderilen değeri offset değişkenine atadık. Sen istediğin değişkeni kullanabilirsin.
* int() fonksiyonu ile `offset` değişkenini integer yaptık.
* Tabii eğer offset değişkenini integer a çeviremezsek Python bize ValueError fırlatacak :) 404 atarı yapacak :)
* Şu regex'in `(\d{1,2})` URLpattern'den yanlızca sayısal değerleri alacağını biliyoruz tabiii. O zaman madem öyle neden view tarafında tekrar int() uygulayıp ValueError tanımladık derseniz.
** (A) fonksiyona Unicode nesnesi geliyor.
** (B) Asıl önemli olan `Loose coupling` mantığı, hani urls.py değişse bile view önceden önlemini almış olsun ;)
* Sonrası bildiğin python, gelen değeri o anki saate ekliyoruz.
* Tabii son olarak `HttpResponse` içerisinde karakter dizimizi yani HTML'mizi gönderiyoruz.
* Denememizi yapalım: http://127.0.0.1:8000/time/plus/3/  ip kısmına siteyi yazacağız demeye gerek yok ama hatırlatmaktan zarar gelmez :)

Ta ta taa taaaaa olduysa, olmadıysa özgün kaynak olan: http://djangobook.com/django-views-dynamic-content/ adresini bir kontrolden zarar gelmez.

## Django’s Pretty Error Pages
## Django'nun güzel mi güzel hata sayfaları


