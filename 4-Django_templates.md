# Django Templates

Bir önceki bölümde view içine HTML kodlarını doğrudan Python kodları içine yazdık. Aşağıdaki örnekte görebilirsin:

```python
def current_datetime(request):
    now = datetime.datetime.now()
    html = "It is now %s." % now
    return HttpResponse(html)
```
Bu başlangıçta view'ın nasıl çalıştığını anlamak için amaca uygundu fakat HTML kodlarını doğrudan view içine yazmak pek iyi bir fikir değildir. Sebeplerini aşağıda sıralayalım:

* Tasarımda herhangi bir değişiklik Python kodlarını da değiştirmemizi gerektirir. Tasarım Python kodları ile oynadamdan değiştirilmeye müsait olmalı.
* Bu yaptıklarımız çok basit örneklerdi. Genelde web sayfalarının templateleri yüzlerce satır HTML ve script kodu içerir. Bu kodlar arasında hata bulmak bir kabusa dönüşür.(Bu kısım PHP de karşılaşılan zorluklara işaret eder ki ben PHP yi de severim :))
* Python kodu yazma ve HTML tasarımı iki farklı disiplindir ve genellikle sorumluluklar iki farklı birim tarafından  paylaşılır. (Haa bizde öyle mi diye sorarsan her zaman değil tabii) Tasarımcılar ve HTML/CSS kodu yazanlar Python kodunu düzenlemeye ihtiyaç duymamalılar.
* Bir kişinin herşey ile uğraşmasındansa; Programcıların Python kodu ile tasarımcıların template ile ilgilenmeleri daha verimli olur. 

Bu sebeplerden ötürü Python kodunun kendisini sayfa tasarımından ayırmak, daha temiz bir kod sağlar ve sürdürülebilirdir.

## Template System Basics
## Template Sistemi Temelleri

Django template bir metin dizisidir ve dokümanın verisini(data) ve sunumunu ayırmaya yöneliktir. Djanogo templates HTML sunumu yanında, herhangi bir metin tabanlı biçimlendirme için de yeteneklidir.

Alışveriş sonrası teşekkür eden basit bir örnek ile başlayalım (Yazar böyle demiz ama bana çok basit gelmedi baştan söyliyeyim ;))

```python
<html>
<head>
    <title>Ordering notice</title>
</head>
<body>

    <h1>Ordering notice</h1>

    <p>Dear {{ person_name }},</p>

    <p>Thanks for placing an order from {{ company }}. It's scheduled to ship on {{ s\
hip_date|date:"F j, Y" }}.</p>
    <p>Here are the items you've ordered:</p>
    <ul>
        {% for item in item_list %}
        <li>{{ item }}</li>{% endfor %}
    </ul>

    {% if ordered_warranty %}
    <p>Your warranty information will be included in the packaging.</p>
    {% else %}
    <p>
        You didn't order a warranty, so you're on your own when
        the products inevitably stop working.
    </p>
    {% endif %}

    <p>Sincerely,<br />{{ company }}</p>

</body>
</html>
```
Yukarıdaki templates basit HTML kodları ile birkaç değişken ve template tags içerir. Hadi inceleyelim...

* Süslü parantezlerle kapalı alan değişkendir. {{ variable }} Bunun anlamı variable isimli değişkeni göster demektir.
* Süslü parantez ve yüzde işareti ile kapalı alan bir template tag'tır {% if ordered_warranty %} Bu oldukça geniş bir anlama gelir ve en basit anlamıyla "do something" yani bişeyler yap demektir.
* Yukarıdaki örnek template, for ve if tagları içeriyor. for ve if taglarının kullanımı Pythondaki for ve if ifadelerine benzer.
* Son olarak ikinci paragraf bir filtre örneği içeriyor. Bu değişkenleri biçimlendirmenin en uygun yoludur. Buradaki örnekte `{{ ship_date|date:"F j, Y" }}` `ship_date` değişkenini `date` filtresinden geçiriyoruz. `date` filtresi "F j, Y" argumanlarını alıyor. Filtreler, Unix sistemlerdeki gibi pipe (|), karakterini kullanarak bağlanır. 
* Her Django template'i dahili tag ve filtrelerle birlikte gelir. Bunları daha sonra inceleyeceğiz. Hatta kendi filtre ve tag larımızı yaratabiliriz.

## Using the Template System
## Template Sistemini Kullanma

Django dahili template sistemi ile gelir ve adı da Django Template Language (DTL)'dir.
Djanog `contrib` uygulaması şuna benzer şekilde (`django.contrib.admin`) templateleri içerir. Biz buradaki uygulamada DTL'i kullanacağız :)

Aşağıda Python kodu içinde Django Template System'i kullanmanın basit yolu anlatılacaktır.

1. `Template` objesini oluştur. Bu karakter dizisi olarak ham template kodu oluşturur.
2. `Template` objesinin `render()` metodunu çağır ki bu (context) içindeki değişken listesini verir. Bu işlem düzenlenmiş template nesnesini karakter dizisi olarak verir. Tabi context e belirlenenlere bağlı olarak. Buralar biraz karışık oluyor ama okudukça ve uyguladıkça anlaşılıyor :)

```python
 >>> from django import template
 >>> t = template.Template('My name is {{ name }}.')
 >>> c = template.Context({'name': 'Nige'})
 >>> print (t.render(c))
 My name is Nige.
 >>> c = template.Context({'name': 'Barry'})
 >>> print (t.render(c))
 My name is Barry.
```
Aşağıda her adımı daha detaylı olarak anlatacağız.

## Creating Template Objects
## Template Objesi Oluşturma

Öncelikle proje klasörümüzün içine girerek şu komutu çalıştıralım. manage.py'nin olduğu klasör `python manage.py shell`
Bu arada `Template` objesini oluşturmanın en kolay yolu onu direk olarak örneklemektir(instantiate). `Template` sınıfı `django.template` modülü içerisinde yer alır ve inşacısı(constructor) bir arguman alır ki buda ham template kodudur(raw template code).

Hadi basitçe inceleyelim:
```python
>>> from django.template import Template
>>> t = Template('My name is {{ name }}.')
>>> print (t)
```
Eğer bunu interaktif shell de çalıştırdıysanız şuna benzer bir çıktı alacaksınız 
`<django.template.base.Template object at 0x030396B0>`
Şu adres `0x030396B0` `Template` objesinin Pythondaki kimliğidir(identity)
Template objesi yarattığımızda, template sistemi düzenlemeye(render() edilmeye)ham karakter dizisi oluşturulur.

## Rendering a Template
## Templeti render etme (Hazır duruma getirme de diyebiliriz :))

Öncelikle bir Template objemiz olmalı ve bunun içerisine `context` veri göndermeliyiz. Bir `context` basitçe, Template değişken isimlerini ve onlara atanan değerleri tutan bir listedir. Hatta sözlük demek daha doğru olur.
Django içinde context, `Context` sınıfında bulunur ve bu sınıfta `django.template` modülünde bulunur. `Context` inşacıları(constructor) değişken sayıda arguman alır.
Basit olarak yapılacak adımlar aşağıdadır.

`Template` objesinin `render()` metodunu `Context` ile çağır ki olsun.

```python
>>> from django.template import Context, Template
>>> t = Template('My name is {{ name }}.')
>>> c = Context({'name': 'Stephane'})
>>> t.render(c)
'My name is Stephane.'
```
Küçük bir not: Daha önce Python kullandıysanız, interaktif shell i neden direk çalıştırmak yerine `python manage.py shell` diye çalıştırdığımızı merak etmiş olabilirsiniz. Bunu yapmamızın amacı Django'nun hangi settings dosyası ile çalışacağını bildirmek.

## Dictionaries and Contexts
## Sözlükler ve Contexts(Şartlar, bağlamlar)

Pythonda sözlükler, bilinen anahtarlar ile değişken değerleri arasında haritalama yapar.(Karışık geldiyse listenin bir benzeri diye düşün :)) `Context` sözlüklere benzer fakat ek özellikler de sunar ki bunlara ileride chapter 8 de bakacağız. 
Değişken isimlerinin case sensetive(büyük küçük harf duyarlı) olduğunu unutmayalım.

Aşağıdaki örneği inceleyelim.

```python
>>> from django.template import Template, Context
>>> raw_template = """<p>Dear {{ person_name }},</p>
...
... <p>
    Thanks for placing an order from {{ company }}. It's scheduled to
    ... ship on {{
ship_date|date:"F j, Y"
    }}.
</p>
...
... {% if ordered_warranty %}
... <p>Your warranty information will be included in the packaging.</p>
... {% else %}
... <p>
    You didn't order a warranty, so you're on your own when
    ... the products inevitably stop working.
</p>
... {% endif %}
...
... <p>Sincerely,<br />{{ company }}</p>"""
>>> t = Template(raw_template)
>>> import datetime
>>> c = Context({'person_name': 'John Smith',
...     'company': 'Outdoor Equipment',
...     'ship_date': datetime.date(2015, 7, 2),
...     'ordered_warranty': False})
>>> t.render(c)
"<p>Dear John Smith,</p>\n\n<p>Thanks for placing an order from Outdoor Equipment. It\
's scheduled to\nship on July 2,2015.</p>\n\n\n<p>You didn't order a warranty, so you\
're on your own when\nthe products inevitably stop working.</p>\n\n\n<p>Sincerely,<br\
 />Outdoor Equipment</p>"
```
* İlk olarak `django.template` modülünden `Template` ve `Context` sınıflarını import ettik.
* `raw_template` değişkenine ham metin verimizi atadık. Tabii burada """ üç nokta ataması kullanmayı unutmuyoruz çünkü neden, tek tırnaklar var yeni satırlar var bunlar bize karışıklık yaratmasın diye.
* `t` adında `Template` objesi oluşturuyoruz. İnşacılasına(constructor) raw_template değişkenini veriyoruz.
* `datetime` modülünü import ediyoruz.
* `c` adında `Context` objemizi oluşturuyoruz. Context inşacısı Python sözlükleri alır, bu değişken ismini değerine haritalar.
* Son olarak `render()` metodunu Template objemizin üzerinde içine context'i göndererek çağırıyoruz. Bu işlem bize düzenlenmiş karakter dizisi olarak dönüyor. Burada “You didn’t order a warranty” paragrafı gözükecek çünkü `ordered_warranty değişkeni` False olarak atanmış. Ayrıca tarih karakter dizisini “F j, Y” şeklinde biçimlendirdiğimiz için `July 2, 2015` biçiminde gözükecek.
* Python'da yeniyseniz \n karakterlerinin neden gözüktüğünü merak edebilirsiniz karakter dizileri ve Python interaktif kabuk konuların detaylı ve mükemmel anlatımı için http://www.istihza.com/ adresini ziyaret ediniz.
* `print(t.render(c))` yazarsanız \n karakterleri gözükmeyecektir.

Burada anlatılanlar Django template system'in temelleridir.
- template string yaz.
- Template objesi oluştur.
- Context oluştur.
- render() metodunu çağır.

## Multiple Contexts, Same Template
## Aynı Template içinde çoklu Context kullanımı






