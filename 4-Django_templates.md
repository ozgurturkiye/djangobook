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

Öncelikle Template objesi oluştur ve farklı context'ler ile render edelim.

```python
>>> from django.template import Template, Context
>>> t = Template('Hello, {{ name }}')
>>> print (t.render(Context({'name': 'John'})))
Hello, John
>>> print (t.render(Context({'name': 'Julie'})))
Hello, Julie
>>> print (t.render(Context({'name': 'Pat'})))
Hello, Pat
```
Aynı template objesini farklı context'ler ile render edeceğimiz zaman öncelikle Template objesini oluşturmak ve sonra render() metodunu çalıştırmak en verimlisidir.(Burada çok fazla ingilizce kavram kullanıyoruz fakat Template-render-context adında kalması anlaşılırlık açısından daha mantıklı)

Aşaıda iyi ve kötü örnekleri inceleyiniz ;)

```python
# Bad
for name in ('John', 'Julie', 'Pat'):
t = Template('Hello, {{ name }}')
print (t.render(Context({'name': name})))

# Good
t = Template('Hello, {{ name }}')
for name in ('John', 'Julie', 'Pat'):
print (t.render(Context({'name': name})))
```

## Context Variable Lookup
## Context Değişkenlerine Bakış

Şuana kadar context içinden basit değerler aktardık ki genelde karakter dizisi. Buna rağmen Template sistemi liste, sözlük, genel obje gibi değerleri de aktarabilir. Django Template içinde karmaşık veri yapılarının kullanımı için kullanılan anathar "." nokta karakteridir.
Noktayı, sözlük anahtarlarına, niteliklere(attributes), methodlara ve obje indexlerine ulaşmak için kullanırız.
Aşağıda Python sözlüklerinin template içinde kullanımına bir örneğe bakalım. Sözlük anathar değerlerine, sözlük anahtarları ile ulşamak için nokta kullanımı.

```python
>>> from django.template import Template, Context
>>> person = {'name': 'Sally', 'age': '43'}
>>> t = Template('{{ person.name }} is {{
person.age
}} years old.')
>>> c = Context({'person': person})
>>> t.render(c)
'Sally is 43 years old.'
```
Benzer bir şekilde noktayı obje niteliklerine(attribute) de erişmek için kullanabiliriz. Örnek olarak datetime.date objesi `year`, `month` ve `day` niteliklerine sahiptir ve bunlara nokta kullanarak ulaşabiliriz.

```python
>>> from django.template import Template, Context
>>> import datetime
>>> d = datetime.date(1993, 5, 2)
>>> d.year
1993
>>> d.month
5
>>> d.day
2
>>> t = Template('The month is {{ date.month }}
and the year is {{ date.year }}.')
>>> c = Context({'date': d})
>>> t.render(c)
'The month is 5 and the year is 1993.'
```
Aşağıdaki örnek genel bir sınıf kullanıyor ve sınıf niteliklerine nokta kullanarak nasıl ulaşacağımızı gösteriyor.

```python
>>> from django.template import Template, Context
>>> class Person(object):
...     def __init__(self, first_name, last_name):
...         self.first_name, self.last_name =
first_name, last_name
>>> t = Template('Hello, {{ person.first_name }} {{ person.last_name }}.')
>>> c = Context({'person': Person('John', 'Smith')})
>>> t.render(c)
'Hello, John Smith.'
```
Nokta ayrıca objelerin(nesnelerin) methodlarına da referans verebilir. Örnek olarak her Python string'i upper() ve isdigit() methodlarına sahiptir. Bu methodları Djanog templateslerde kullanabilir.

```python
>>> from django.template import Template, Context
>>> t = Template('{{ var }} -- {{ var.upper }} -- {{ var.isdigit }}')
>>> t.render(Context({'var': 'hello'}))
'hello -- HELLO -- False'
>>> t.render(Context({'var': '123'}))
'123 -- 123 -- True'
```
Not: Method çağrılarında parantezleri kullanamazsın. Ayrıca methodlara argumanda gönderemezsin. Yalnızca arguman istemeyen methodları kullanbilirsin.(Daha sonra neden olacağını açıklayacağım yazılmış :)). Son olarak nokta liste indexlerine erişmek için kullanılır.

```python
>>> from django.template import Template, Context
>>> t = Template('Item 2 is {{ items.2 }}.')
>>> c = Context({'items': ['apples', 'bananas',
'carrots']})
>>> t.render(c)
'Item 2 is carrots.'
```
Not: Negatif liste indexlerine izin verilmez. Örnek olarak şu `{{ items.-1 }}` template değişkeni şu `TemplateSyntaxError.` hatayı verecektir.

Nokta kullanımı inceleme şu şekilde özetlenebilir: Template sistemi değişkende noktaya rastladığı zaman, aşağıdaki sırayla işlem yapmaya çalışır:

- Dictionary lookup (e.g., foo["bar"]) (Sözlüğe bakar)
- Attribute lookup (e.g., foo.bar) (niteliğe bakar)
- Method call (e.g., foo.bar()) (methoda bakar)
- List-index lookup (e.g., foo[2]) (liste index'e bakar)

Nokta kullanımı çoklu seviyede olabilir. Örnek olarak {{ person.name.upper }} öncelikle sözlük olarak işler (person['name']) sonra method çağrısı yapar  (upper()):

```python
>>> from django.template import Template, Context
>>> person = {'name': 'Sally', 'age': '43'}
>>> t = Template('{{ person.name.upper }} is {{
person.age
}} years old.')
>>> c = Context({'person': person})
>>> t.render(c)
'SALLY is 43 years old.'
```
## Method Call Behavior
## Method Çağırma Davranışı

Method çağrıları, diğer çağrı tiplerinden biraz daha karmaşıktır. Aşağıda aklında tutman gereken şeyleri listeledik.

* Eğer method çağrısı sırasında, method hata fırlatırsa :) ve bu hatanın `silent_variable_failure` niteliği değeri `True` değil ise, hata yayılır diğer türlü hata vermez. (Karışık oldu örnek açıklayacaktır)

```python
>>>
  t = Template("My name is {{ person.first_name }}.")
  >>> class PersonClass3:
  ...     def first_name(self):
  ...         raise AssertionError("foo")
  >>> p = PersonClass3()
  >>> t.render(Context({"person": p}))
  Traceback (most recent call last):
  ...
  AssertionError: foo

  >>> class SilentAssertionError(Exception):
  ...     silent_variable_failure = True
  >>> class PersonClass4:
  ...     def first_name(self):
  ...         raise SilentAssertionError
  >>> p = PersonClass4()
  >>> t.render(Context({"person": p}))
  'My name is .'
  ```
  * Method çağrısı yalnızca eğer method zorunlu argumana sahip değilse çalışacaktır. Diğer türlü sistem sonraki lookup tipini(bakmayı) çalıştıracaktır. (list-index lookup).
  * Tasarım olarak Django, mantıksal işlemlerin Template içinde yapılmasını engellemiştir. Bu sebepten Template içinden methodlara arguman gönderme imkanı yoktur. Veri views içinde hesaplanmalı ve Template'e sadece gösterim için gönderilmelidir.
  * Template sisteme bu erişimlerin izinlerini verilseydi güvenlik açığı oluşturdu ve akıllıca olmazdı.
  * Örnek olarak `BankAccount` nesnesi ve sahip olduğu delete() methodu olduğunu düşünelim. Eğer template şunun gibi birşey içerse {{ account.delete }} templete render edildiğinde nesne silinirdi. Bundan kaçınmak için fonksiyon niteliği olan `alters_data` method içinde düzenlenmeli. (Bu kısmı tam anlayamadım bana da karışık geldi ?)
 
 Not: Django model objesinde otomatik oluşturulan delete() ve save() methodları otomatik olarak alters_data=true olur.
 Bu kısmın orjinal ingilizce dokümana bakmak gerek tam anlaşılamadı ki tam anlatılsın ???...
 
 ## How Invalid Variables Are Handled
 ## Uygun olmayan değişkenler nasıl ele alınır
 
 Genellikle, template sistem eğer değişken yoksa `string_if_invalid` ayarlarına göre işlem yapar. Bu standartta boş stringtir. Yani boş karakter basar.
 
 ```python
 >>> from django.template import Template, Context
>>> t = Template('Your name is {{ name }}.')
>>> t.render(Context())
'Your name is .'
>>> t.render(Context({'var': 'hello'}))
'Your name is .'
>>> t.render(Context({'NAME': 'hello'}))
'Your name is .'
>>> t.render(Context({'Name': 'hello'}))
'Your name is .'
```

Bu davranış hata vermekten daha iyidir çünkü bu planlanmış insanların yapabileceği hatadır.
Yukarıdaki örnekte tüm lookup'lar(aramalar) başarısız oldu çünkü değişken isimleri hatalı.
Gerçek dünyada, küçük template yazım hatalarından dolayı Web sitelerinin erişilemez olaması kabul edilemez.
 

