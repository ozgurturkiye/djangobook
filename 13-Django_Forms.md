# Django Forms
# Django Formları

HTML formları, Google'ın tek arama kutusunun sadeliğinden, her yerde bulunan blog yorum gönderme formlarına, karmaşık özel veri girişi arabirimlerine kadar etkileşimli web sitelerinin belkemiğidir.

Bu bölümde, kullanıcı tarafından gönderilen form verisine erişmek, doğrulamak ve onunla bir şeyler yapmak için Django'yu nasıl kullanabileceğiniz ele alınmaktadır. Yol boyunca, `HttpRequest` ve `Form` nesnelerini ele alacağız.

## Getting Data from the Request Object
## İstek Objesinden Veri Almak

İlk olarak görüntüleme işlevlerini kapladığımızda `HttpRequest` nesnelerini Bölüm 2'de tanıttım, ancak o zamanlar hakkında söyleyecek çok şey yoktu. Her görünüm işlevinin bir `HttpRequest` nesnesini ilk parametresi olarak aldığını,
hello() görünümü örneğinde olduğu gibi:

```python
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello world")
```

`HttpRequest` nesneleri, örneğin "request" değişkeni, size ilginizi çekebilecek ilginç nitelikler ve yöntemler içerir, böylece mümkün olanı biliyorsunuzdur. Geçerli isteğiniz (yani, geçerli sayfayı Django ile çalışan sitenize yükleyen kullanıcı / Web tarayıcısı) hakkında bilgi almak için bu öznitelikleri, görüntüleme işlevi yürütüldüğünde kullanabilirsiniz.

## Information About the URL
## URL Hakkında Bilgi

`HttpRequest` nesneleri, şu anda istenen URL hakkında birkaç bilgi içerir (Tablo 6-1).

Tablo 6-1: `HttpRequest` yöntemleri ve öznitelikleri

Attribute/method 	Description 	Example

request.path 	The full path, not including the domain but including the leading slash. 	“/hello/”

request.get_host() 	The host (i.e., the “domain,” in common parlance). 	“127.0.0.1:8000” or “www.example.com”

request.get_full_path() 	The path, plus a query string (if available). 	“/hello/?print=true”

request.is_secure() 	True if the request was made via HTTPS. Otherwise, False. 	True or False

Görüşlerinizdeki URL'leri sabit kodlamak yerine daima bu nitelikleri / yöntemleri kullanın. Bu, başka yerlerde tekrar kullanılabilen daha esnek kodlar sağlar. Basit bir örnek:

```python
# BAD!
def current_url_view_bad(request):
    return HttpResponse("Welcome to the page at /current/")

# GOOD
def current_url_view_good(request):
    return HttpResponse("Welcome to the page at %s"
% request.path)
```

## Other Information About the Request
## İsteğe İlişkin Diğer Bilgiler

`Request.META`, kullanıcının IP adresi ve kullanıcı aracısı (genellikle Web tarayıcısının adı ve sürümü) de dahil olmak üzere, verilen istek için mevcut tüm HTTP üstbilgilerini içeren bir Python sözlüğüdür. Kullanılabilir üstbilgilerin tam listesinin, kullanıcının gönderdiği üstbilgilere ve Web sunucunuzun hangi üstbilgilere ayarladığına bağlı olduğunu unutmayın. Bu sözlükte yaygın olarak kullanılan bazı anatharlar şunlardır:

* HTTP_REFERER - Varsa yönlendiren URL. (REFERER yazım hatalı yazılmış unutmayın.)
* HTTP_USER_AGENT - Kullanıcının tarayıcısının kullanıcı aracısı dizesi, varsa. Bu şuna benziyor:
"Mozilla/5.0 (X11; U; Linux i686; fr-FR; rv:1.8.1.17) Gecko/20080829 Firefox/2.0.0.17".
* REMOTE_ADDR - Müşterinin IP adresi, ör. "12.345.67.89". (İstek, herhangi bir proxy üzerinden geçerse, bu, virgülle ayrılmış bir IP adresi listesi olabilir (ör. "12.345.67.89,23.456.78.90").)

`Request.META`'nın sadece basit bir Python sözlük olduğundan, mevcut olmayan bir anahtara erişmeye çalışırsanız bir `KeyError` istisnası alırsınız. (HTTP üstbilgileri harici verilerdir - diğer bir deyişle, kullanıcılarınızın tarayıcıları tarafından gönderilirler - güvenilmemesi gerekir ve belirli bir üstbilgi boşsa veya yoksa daima uygulamanızın düzgün bir şekilde başarısız olacak şekilde tasarlanması gerekir. ) Tanımsız tuşların durumunu işlemek için `try / except` veya `get()` yöntemlerini kullanmalısınız:

```python
# BAD!
def ua_display_bad(request):
    ua = request.META['HTTP_USER_AGENT']  # Might raise KeyError!
    return HttpResponse("Your browser is %s" % ua)

# GOOD (VERSION 1)
def ua_display_good1(request):
    try:
        ua = request.META['HTTP_USER_AGENT']
    except KeyError:
        ua = 'unknown'
    return HttpResponse("Your browser is %s" % ua)

# GOOD (VERSION 2)
def ua_display_good2(request):
    ua = request.META.get('HTTP_USER_AGENT', 'unknown')
    return HttpResponse("Your browser is %s" % ua)
```

Orada ne olduğunu öğrenmek için tüm `request.META` verilerini gösteren küçük bir görünüm yazmanızı öneririm. İşte o görünüm şöyle görünebilir:

```python
def display_meta(request):
    values = request.META.items()
    values.sort()
    html = []
    for k, v in values:
        html.append('<tr><td>%s</td><td>%s</td></tr>' % (k, v))
    return HttpResponse('<table>%s</table>' % '\n'.join(html))
```

İstek nesnesinin hangi bilgileri içerdiğini öğrenmenin bir diğer iyi yolu, sistemi çöktüğünde Django hata sayfalarına yakından bakmaktır - tüm HTTP üstbilgileri ve diğer istek nesneleri de dahil olmak üzere çok sayıda yararlı bilgi var (Örneğin `request.path`).

## Information About Submitted Data
## Gönderilen Veriler Hakkında Bilgi

İstekle ilgili temel meta verilerin ötesinde, `HttpRequest` nesnelerinin, kullanıcı tarafından gönderilen bilgileri içeren iki öznitelikleri vardır: `request.GET` ve `request.POST`. Bunların her ikisi de `GET` ve `POST` verilerine erişmenizi sağlayan sözlük benzeri nesnelerdir. `POST` verileri genellikle HTML <form> 'dan gönderilirken, `GET` verileri bir <form> veya sayfanın URL'sindeki sorgu dizesinden gelebilir.

* Sözlük benzeri nesneler `request.GET` ve `request.POST` ifadelerinin "sözlük benzeri" nesneler olduğunu söylersek, standart Python sözlükleri gibi davrandıklarını, ancak teknik olarak sözlükte olmayan sözlükleri kastediyoruz. Örneğin, request.GET ve request.POST her ikisinde de `get()`, `keys()` ve `values()` yöntemleri bulunur ve request.GET'de anahtar için yaparak anahtarların üzerinde tekrarlama yapabilirsiniz. Öyleyse ayrım neden? Çünkü hem request.GET hem de request.POST normal sözlüklerin sahip olmadığı ek yöntemlere sahiptir. Bunlara kısa bir süre içinde gireceğiz. Benzer bir terim "dosya benzeri nesneler" ile karşılaşmış olabilirsiniz - "gerçek" dosya nesneleri için stand-by görevi görmelerine izin veren read () gibi birkaç temel yöntemi olan Python nesneleri.

## A Simple Django Form-Handling Example
## Basit bir Django Form Kullanma Örneği

Devam etmekte olan kitap, yazar ve yayıncı örneklerine devam ederek, kullanıcıların kitap veritabanımızda başlığa göre arama yapmasını sağlayan basit bir görünüm oluşturalım. Genel olarak, form geliştirmenin iki parçası vardır: HTML kullanıcı arabirimi ve gönderilen verileri işleyen arka uç görünümü kodu. İlk bölüm kolay; Şimdi bir arama formu görüntüleyen bir görünüm oluşturalım:

```python
from django.shortcuts import render

def search_form(request):
    return render(request, 'search_form.html')
```

Bölüm 3'te öğrendiğiniz gibi bu görünüm, Python yolunuzun herhangi bir yerinde yaşayabilir. Argüman uğruna, `books/views.py` dosyasına koyun. Eşlik eden şablon `search_form.html` şöyle görünebilir:

```html
<html>
<head>
    <title>Search</title>
</head>
<body>
    <form action="/search/" method="get">
        <input type="text" name="q">
        <input type="submit" value="Search">
    </form>
</body>
</html>
```

Bu dosyayı Bölüm 3'te oluşturduğunuz `mysite/templates` dizinine kaydedin veya yeni bir klasör `books/templates` oluşturun. Ayarlar dosyanızda 'APP_DIRS' öğesinin True olarak ayarlandığından emin olun. `urls.py` dosyasındaki URLPattern şöyle görünebilir:

```python
from books import views

urlpatterns = [
    # ...
    url(r'^search-form/$', views.search_form),
    # ...
]
```

(Dikkat edersek, `mysite.views` `import search_form` gibi bir şey yerine, doğrudan `views` modülünü içe aktarıyoruz, çünkü eski ayrıntı daha az ayrıntılıdır. Bu içe aktarma yaklaşımını Bölüm 7'de daha ayrıntılı olarak ele alacağız.) Şimdi çalıştırırsanız Geliştirme sunucusunu ziyaret edin ve http://127.0.0.1:8000/search-form/ adresini ziyaret edin, arama arayüzünü görürsünüz. Yeterince basit Formu göndermeyi deneyin, ancak bir Django 404 hatası alırsınız. Form, URL / arama / 'yı işaret eder ve henüz uygulanmamıştır. Bunu ikinci bir görüntüleme işlevi ile düzeltelim:

Şimdilik, bu sadece kullanıcının arama terimini görüntüler, bu nedenle verilerin Django'ya düzgün şekilde gönderildiğinden emin olabilir ve böylece arama teriminin sisteme nasıl aktığını hissedebilirsiniz. Kısacası:

1. HTML <form>, q değişkenini tanımlar. Gönderildiğinde, q değeri GET (method = "get") yoluyla URL / search / adresine gönderilir.

2. URL'yi /search / (search()) işleyen Django görünümü request.GET'deki q değerine erişebilir.

Buraya dikkat çekilmesi gereken önemli bir husus, 'q' öğesinin `request.GET`'de bulunduğunu açıkça kontrol etmemizdir. Yukarıdaki request.META bölümünde belirttiğim gibi, kullanıcılar tarafından gönderilen herhangi bir şeye güvenmemelisiniz veya ilk etapta bir şey gönderdiklerini varsayalım. Bu onay eklemezsek, boş bir formun gönderilmesi KeyError'u görünüşte artıracaktır:

```python
# BAD!
def bad_search(request):
    # The following line will raise KeyError if 'q' hasn't
    # been submitted!
    message = 'You searched for: %r' % request.GET['q']
    return HttpResponse(message)
```

## Query String Parameters
## Sorgu Dizesi Parametreleri

`GET` verileri sorgu dizesinde iletildiğinden (ör. `/Search/?q=django`), sorgu dizesi değişkenlerine erişmek için `request.GET` kullanabilirsiniz. Bölüm 2'nin Django'nun URLconf sistemine girişinde, Django'nun güzel URL'lerini `/time/plus?hours=3` gibi daha geleneksel PHP / Java URL'leriyle karşılaştırmıştım ve size 6. bölümde nasıl yapılacağını göstereceğim dedi. Görüşlerinizdeki sorgu dizesi parametrelerine nasıl erişileceği (bu örnekte `saat=3` gibi) - `request.GET` kullanın.

`POST` verileri `GET` verileri ile aynı şekilde çalışır - yalnızca request.GET yerine request.POST kullanın. GET ve POST arasındaki fark nedir? Form gönderme işlemi yalnızca "veri almak" için bir istek olduğunda GET kullanın. Form gönderme işleminin herhangi bir yan etkisi olacaksa POST kullanın -
Veri değiştirme veya bir e-posta gönderme veya basit bir veri görüntüleme ötesinde bir şey. Kitap arama örneğimizde, sorgu sunucumuzdaki verileri değiştirmediğinden GET kullanıyoruz. (GET ve POST hakkında daha fazla bilgi edinmek isterseniz w3.org sitesine bakın.) Request.GET dosyasının düzgün şekilde geçtiğini doğruladıktan sonra, kullanıcının arama sorgusunu kitap veritabanımıza bağlayalım (tekrar, views.py):

```python
from django.http import HttpResponse
from django.shortcuts import render
from books.models import Book

def search(request):
    if 'q' in request.GET and request.GET['q']:
        q = request.GET['q']
        books = Book.objects.filter(title__icontains=q)
        return render(request, 'search_results.html',
                      {'books': books, 'query': q})
    else:
        return HttpResponse('Please submit a search term.')
```

Burada yaptığımız konuyla ilgili birkaç not:

* İsteğe bağlı olarak 'q' öğesinin request.GET'de olup olmadığını kontrol etmenin yanı sıra request.GET ['q'] öğesinin veritabanı sorgusuna geçirmeden önce boş olmayan bir değer olduğundan emin olun.
* Başlığı, verilen göndermeyi içeren tüm kitaplar için kitap masamızda sorgulamak için `Book.objects.filter(title__icontains=q)` kullanıyoruz. İçindekiler bir arama türüdür (Bölüm 4 ve Ek B'de açıklandığı gibi) ve açıklama kabaca "Büyük / küçük harf duyarlı olmadan başlıkları q olan kitapları al" olarak çevrilebilir.

Bu kitap araması yapmak için çok basit bir yoldur. Yavaş olabileceğinden, büyük bir üretim veritabanında basit bir icontains sorgusu kullanmamızı öneririz. (Gerçek dünyada, özel bir arama sistemi kullanmak isteyebilirsiniz. Olasılıklar hakkında fikir edinmek için Web'de açık kaynak kodlu tam metin arama yapın.)

Kitaplara, Kitap nesnelerinin bir listesini şablona göndeririz. Search_results.html dosyası buna benzer bir şey içerebilir:

```html
<html>     
    <head>         
    <title>Book Search</title>     
    </head>     
    <body>       
        <p>You searched for: <strong>{{ query }}</strong></p>
   
        {% if books %}           
            <p>Found {{ books|length }} book{{ books|pluralize }}.</p>
            <ul>               
                {% for book in books %}               
                <li>{{ book.title }}</li>               
                {% endfor %}           
            </ul>       
        {% else %}           
            <p>No books matched your search criteria.</p>       
        {% endif %}     
    </body>
</html>
```

Bulunan kitapların sayısına göre uygunsa "s" çıkartan `pluralize` şablon filtresinin kullanımına dikkat edin.

## Improving Our Simple Form-Handling Example
## Basit Form Kullanma Örneğimizin Geliştirilmesi

Daha önceki bölümlerde olduğu gibi, size muhtemelen işe yarayabilecek en basit şeyi gösterdim. Şimdi bazı sorunları işaret edip nasıl geliştireceğimi göstereceğim. İlk olarak, `arama()` görünümümüzün boş bir sorguyu işleme zayıf - yalnızca kullanıcının "tarayıcı geriye düğmesini tıklatmasını gerektiren bir"Please submit a search term."mesajı gösteriyoruz.

Bu korkunç ve profesyonel olmayan ve böyle bir şeyi gerçekte vahşi halde uygularsanız, Django ayrıcalıklarınız iptal edilecektir. Kullanıcının hemen yeniden deneyebilmesi için, formun üstünde bir hata ile yeniden görüntülenmesi daha iyi olurdu. Bunu yapmanın en kolay yolu şablonu tekrar oluşturmaktır:

```python
from django.http import HttpResponse
from django.shortcuts import render
from books.models import Book

def search_form(request):
    return render(request, 'search_form.html')

def search(request):
    if 'q' in request.GET and request.GET['q']:
        q = request.GET['q']
        books = Book.objects.filter(title__icontains=q)
        return render(request, 'search_results.html', {'books': books, 'query': q})
    else:
        return render(request, 'search_form.html', {'error': True})
```

(Tek bir yerde iki görüntüyü de görebilmeniz için `search_form()` öğesini eklediğimi unutmayın.) Sorgu boşsa, `search()` işlevini `search_form.html` şablonunu tekrar oluşturacak şekilde geliştirdik. Ve bu şablondaki bir hata iletisini göstermemiz gerektiğinden, bir şablon değişkeni iletiriz. Şimdi hata değişkenini kontrol etmek için `search_form.html` dosyasını düzenleyebiliriz:

```html
<html>
<head>
    <title>Search</title>
</head>
<body>
    {% if error %}
        <p style="color: red;">Please submit a search term.</p>
    {% endif %}
    <form action="/search/" method="get">
        <input type="text" name="q">
        <input type="submit" value="Search">
    </form>
</body>
</html>
```

Bu şablonu orijinal görünümümüz `search_form()` 'dan kullanabiliriz, çünkü `search_form()` şablona hata iletmez - bu durumda hata mesajı görünmez. Bu değişiklik yerine getirildiğinde daha iyi bir uygulama, ancak şimdi şu soruyu kabul etmiyor: özel bir `search_form()` görünümü gerçekten gerekli mi?

Durduğunda URL `/search/` (herhangi bir `GET` parametresi olmadan) bir istek boş formu görüntüler (ancak bir hata ile). Birisi ziyaret / arama / hiçbir GET parametresi olmadan hata mesajını gizlemek için search () öğesini değiştirdikçe search_form() görünümünü ve ilişkili URLpattern'ini kaldırabiliriz:

```python
def search(request):
    error = False
    if 'q' in request.GET:
        q = request.GET['q']
        if not q:
            error = True
        else:
            books = Book.objects.filter(title__icontains=q)
            return render(request, 'search_results.html',
                          {'books': books, 'query': q})
    return render(request, 'search_form.html', {'error': error})
```

Bu güncellenmiş görünümde, bir kullanıcı `GET` parametreleri olmadan `/search/` ziyaret ederse arama formunu hiçbir hata mesajı almayacaklarını görürler. Bir kullanıcı formu 'q' için boş bir değerle gönderirse, arama formunun hata mesajıyla birlikte görür. Son olarak, bir kullanıcı formu 'q' için boş olmayan bir değerle gönderirse, arama sonuçlarını görürler.

Biraz fazlalığı ortadan kaldırmak için bu uygulamaya son bir iyileştirme yapabiliriz. Şimdi, iki görüntüyü ve URL'yi hem arama biçimi görüntüleme hem de sonuç görüntüleme alanlarına tek tek ve `/search/` işleme tabi tuttuğumuza göre, `search_form.html` dosyasındaki HTML <form>, bir URL'yi sabit kodlamak zorunda değildir. Bunun yerine:

```html
<form action="/search/" method="get">
```

Bu şu şekilde değiştirilebilir:

```html
<form action="" method="get">
```

`action=""`, formun geçerli sayfa ile aynı URL'ye gönderilmesi "anlamına gelir. Bu değişiklik gerçekleştiğinde, `search()` görünümünü başka bir URL'ye bağladığınızda `action` değiştirmeyi hatırlamanız gerekmez.
