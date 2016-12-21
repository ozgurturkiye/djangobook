# Django Views and URLconfs 
## Hello world sayfasını yayınlayalım
### Detaylı ve mantık silsilesi içinde anlatım için http://djangobook.com/views-urlconfs/ adresini ziyaret ediniz :)

### İlk view oluşturulur
1. mysite dizini içerisinde `views.py` dosyasını oluştur.
Bu python modülü bu bölüm için oluşturacağımız view'leri içerecek
2. `views.py` dosyasının içeriğini aşağıdaki gibi oluştur.
```python
from django.http import HttpResponse  

def hello(request):
    return HttpResponse("Hello world")
```
 Şimdi neler yaptık satır satır inceleyelim :)
 * İlk olarak HttpResponse `sınıfını` django.http `modülü` içerisinden import ettik
 * Sonra, hello fonksiyonunu tanımladık - view fonksiyonu
 * Her view fonksiyonu en az bir parametre alır ve adı `request` olur gelenek olarak ona göre python geleneklere saygılıdır.
 Bu nesne aktif web request(bunu ingilizcesi ile bırakıyorum) içerir ve view i tetikler. Ayrıca django.http.HttpRequest sınıfının örneği olur kendisi.
 * Bu örnekte `request` ile ilgili herhangi birşey yapmadık fakat ilk parametre olmak zorundadır.
 * Son satırda yanlızca HttpResponse nesnesini döndürür ki "Hello world" tür kendisi :)
 * Burada ki ana ders: view yalnızca bir Python fonksiyonudur ve parametre olarak `HttpRequest` alır ve `HttpRespons` örneğini döndürür.
 
### İlk URLCONF
1. Django'da belirli bir URL'den view fonksiyonunu yakalamak için URLconf' kullanırız. URLconf'u içerik tablosu gibidir.
2. urls.py nin orjinal hali şu şekilde ve biz onu bir altındaki şekle getireceğiz :)
```python
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
]
```
Sonra şu eklemeleri yapıyoruz.(Detaylı anlatım için ingilizcesine bakınız uzun geldi)
```python
from django.conf.urls import include, url
from django.contrib import admin
from mysite.views import hello  
urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^hello/$', hello),
]
```
3. Şimdi http://127.0.0.1:8000/hello/ yazdın mı iş tamamdır. Tabi localde çalışıyorsan geçerli diğer türlü kendi web sitemizi yazıcağız. 
4. Çalıştıysa Hobaaaa diyoruz geçiyoruz sıradaki çalışmaya :) Bir çay kahve de gider tabii bu arada ;)

### 404 Hatası hakkında küçük bir not
Bu bölümde olmayan bir url girdiğinizde örnek: http://127.0.0.1:8000/rtfm/ anladın sen. Page not found hatası alırsın. Tabi sebebi sebebi henüz geliştirme aşamasındayız ve bizden başka gören yok. Eğer productiona(yani hizmette olan sitede) bunu yaparsan hassas bilgilerin zıplar dışarı üzülürsün demiş adam ben onu iletiyim bilgin olsun :) İleride kapatacağız hata modunu.

### Site ana sayfası için küçük bir not
Aşağıdaki şekilde yaparsan site kök dizinini istediğin view e yönlendirirsin. Olay ` URLpattern “^$” ` kullanımında.
```python
from mysite.views import hello, my_homepage_view

urlpatterns = [
    url(r'^$', my_homepage_view),
    # ...
```




