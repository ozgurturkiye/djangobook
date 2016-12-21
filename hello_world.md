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
 
 

