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

`Request.META`, kullanıcının IP adresi ve kullanıcı aracısı (genellikle Web tarayıcısının adı ve sürümü) de dahil olmak üzere, verilen istek için mevcut tüm HTTP üstbilgilerini içeren bir Python sözlüğüdür. Kullanılabilir üstbilgilerin tam listesinin, kullanıcının gönderdiği üstbilgilere ve Web sunucunuzun hangi üstbilgilere ayarladığına bağlı olduğunu unutmayın. Bu sözlükte yaygın olarak kullanılan bazı tuşlar şunlardır:

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

üstteki örnekleri uygulamayı unutma :)

