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
Yukarıda ki templates basit HTML kodları, birkaç değişken ve template tags içerir. Hadi inceleyelim...

* 

