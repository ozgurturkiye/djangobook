# Django Template Tags and Filters
# Django Template Etiket ve Filtreleri

Daha önce de söylediğimiz gibi, template sistemi bult-in(dahili) tags ve filtreler ile gelir.

## TAGS

### if/else

{% if %} tagı değişkenleri değerlendirir ve eğer değişken değeri "True"(Değişkenin var olması, boş olmaması ve değerinin Bolean False olmaması) ise sistem, {% if %} ve {% endif %} tagları arasındaki herşeyi gösterir.
```python
{% if today_is_weekend %}
<p>Welcome to the weekend!</p>
{% endif %}

An `{% else %}` tag is optional:

{% if today_is_weekend %}
<p>Welcome to the weekend!</p>
{% else %}
<p>Get back to work.</p>
{% endif %}
```
if tagı bir veya daha fazla {% elif %} tagı alabilir.

```python
{% if athlete_list %}
Number of athletes: {{ athlete_list|length }}
{% elif athlete_in_locker_room_list %}
<p>Athletes should be out of the locker room soon! </p>
{% elif ...
...
{% else %}
<p>No athletes. </p>
{% endif %}
```
{% if %} tag'ı and, or ve not değerlendirme operatörlerini kabul edebilir.

```python
{% if athlete_list and coach_list %}
<p>Both athletes and coaches are available. </p>
{% endif %}

{% if not athlete_list %}
<p>There are no athletes. </p>
{% endif %}

{% if athlete_list or coach_list %}
<p>There are some athletes or some coaches. </p>
{% endif %}

{% if not athlete_list or coach_list %}
<p>There are no athletes or there are some coaches. </p>
{% endif %}

{% if athlete_list and not coach_list %}
<p>There are some athletes and absolutely no coaches. </p>
{% endif %}
```

and ve or ifadeleri arasında and'in önceliği vardır.
```python
{% if athlete_list and coach_list or cheerleader_list %}
```
Yukarıdaki komut şu şekilde yorumlanır interpreter tarafından :)
```python
if (athlete_list and coach_list) or cheerleader_list
```
Ayrıca iç içe {% if %} tagları kullanabilirsin.
```python
 {% if athlete_list %}
     {% if coach_list or cheerleader_list %}
         <p>We have athletes, and either coaches or cheerleaders! </p>
     {% endif %}
 {% endif %}
```
Mantıksal operatörleri çoklu olarak ta kullanabilirsin fakat farklı operatörleri birleştiremezsin. (Demiş ama sanki çalışır gibi geliyor bana)

`{% if %}` tagını `{% endif %}` tagı ile kapatmayı unutmuyoruz. Yoksa Django bize `TemplateSyntaxError` hatasını fırlatır.(Şu fırlatma kelimesini çok seviyorum yoksa bende bilirim hata verir demeyi ;))

### for

`{% for %}` tagı dizi içindeki her eleman için döngü kurmamıza izin verir. Sözdizimi aynı Python'daki gibidir. `for X in Y` şeklinde burada Y dizi X ise her döngüde elde edilen değişkenin adıdır. Döngünün her çalışmasında `{% for %}` ve `{% endfor %}` tagları arasındaki herşey render edilir. Aşağıda `athlete_list` dizisininin içerisindekileri ekrana basıyoruz.

```python
<ul>
    {% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
    {% endfor %}
</ul>
```
Döngüyü tersten çalıştırmak için `reversed` tagını ekleyebiliriz.
```python
{% for athlete in athlete_list reversed %}
...
{% endfor %}
```
İç içe `{% for %}` kullanımına izin verilir:
```python
{% for athlete in athlete_list %}
<h1>{{ athlete.name }}</h1>
<ul>
    {% for sport in athlete.sports_played %}
    <li>{{ sport }}</li>
    {% endfor %}
</ul>
{% endfor %}
```
İç içe listeleri de kullanabiliriz. Bunun için önce her alt listenin değerini değişkene çıkarmak gerekir.
Örnek olarak `(x,y)` koordinatlarını içeriren `points` diye bir listen olsun.
```pyhon
{% for x, y in points %}
<p>There is a point at {{ x }},{{ y }}</p>
{% endfor %}
```
Ayrıca benzer bir şekilde sözlük üyelerine ulaşmak içinde benzer bir yöntem uygularız. `data` isimli sözlük için:
```python
{% for key, value in data.items %}
{{ key }}: {{ value }}
{% endfor %}
```

Listelerin boyutunu kontrol kontrol edip duruma göre çıktı vermek için genel bir örnek:
```python
{% if athlete_list %}

{% for athlete in athlete_list %}
<p>{{ athlete.name }}</p>
{% endfor %}

{% else %}
<p>There are no athletes. Only computer programmers.</p>
{% endif %}
```

Yukaradakine eş değer bir örneği `{% empty %}` tagı ile yapalım:

```python
{% for athlete in athlete_list %}
<p>{{ athlete.name }}</p>
{% empty %}
<p>There are no athletes. Only computer programmers.</p>
{% endfor %}
```

Döngü bitmeden döngüyü kırmak(Break) için bir seçenek yoktur. Bunu yapmak istiyorsan döngü kurduğun değişkeni değiştirmelisin :)
Benzer bir şekilde döngüyü atlatıp devam etmek için "continue" de bir seçenek yok. Baba bunlar niye yok diye bir soru gelebilir aklına ama adam onu da açıklamış ve demiş ki bu Django'nun felsefesi ve kısıtlamalarıdır.“Philosophies and Limitations”. Sonuç olarak adamlar bilerek buna izin vermiyor.

Her `{% for %}` döngüsünün içinde adı `forloop` olan template değişkenine ulaşabilirsin. Bu değişken döngünün durumu hakkında birkaç niteliğe sahiptir.
* `forloop.counter` integer bir değer döndürür ve döngünün kaçıncı defa çalıştığını söyler ayrıca 1'den başlar. Örnek
```python
  {% for item in todo_list %}

  <p>{{ forloop.counter }}: {{ item }}</p>
  {%
  endfor %}
  ```
  * `forloop.counter0` buda üsttekinin benzeri lakin 0'dan başlar :)
  * `forloop.revcounter` integer değer tutar ve döngünün kaç defa döneceğini söyler ve her çalışmada bir azalarak son döngüde bir(1) sayısını alır.
  * `forloop.revcounter0` bu da üsttekinin aynısı lakin son döngüde 0 olur.
  * `forloop.first` Boolean değer tutar ve döngü ilk defa çalışıyorsa "True" değerini alır. Bu özel durumlar için çok uygundur: 
  ```python
  {% for object in objects %}

  {% if forloop.first %}<li class="first">{% else %}<li>
      {%
      endif %}
      {{ object }}
  </li>
  {% endfor %}
  ```
  * `forloop.last` Boolean değer tutar ve döngü son defa çalışıyorsa "True" değerini alır. Genel kullanımı linklerin arasına   liste koymak içindir
  ```python
    {% for link in links %}
  {{ link }}{% if not forloop.last %} | {% endif %}
  {% endfor %}
  ```
  Yukarıdaki kod şunun gibi bir çıktı verecektir:
  `Link1 | Link2 | Link3 | Link4`
  Diğer bir kullanımı da kelimeler arasına virgül koymaktır:
  ```python
    <p>Favorite places:</p>
      {% for p in places %}{{ p }}{% if not forloop.last %}, {% endif %}
      {% endfor %}
  ```
  * `forloop.parentloop` bu ise iç içe kullanımda parent loop olan `forloop` a referans verir:
  ```python
  {% for country in countries %}
  <table>
      {% for city in country.city_list %}
      <tr>
          <td>Country #{{ forloop.parentloop.counter }}</td>
          <td>City #{{ forloop.counter }}</td>
          <td>{{ city }}</td>
      </tr>
      {% endfor %}
  </table>
  {% endfor %}
  ```
  
  `forloop` değişkeni yalnızca döngü içinde erişilebilir. Template `{% endfor %}` u çalıştırdıktan sonra forloop kaybolur.
  
### ifequal/ifnotequal
  
Django template sistem eksiksiz bir programlama dili değildir ve bu yüzden sana tüm Python ifadelerini çalıştırmak için izin vermez.(Tabi sebebi felsefesi ve limitlerinden gelir yoksa ayıpsın hepsini yapar)
  
Tabi yinede iki değeri karşılaştırıp eşit olup olmama durumuna göre işlem yapabilir. Django bunu `{% ifequal %}` tagı ile yapar.
`{% ifequal %}` tagı iki değeri karşılaştırır ve eşitseler `{% ifequal %}` ile `{% endifequal %}` arasındaki herşeyi gösterir.
Aşağıda `user` ve `currentuser` karşılaştırması örneği:
```python
{% ifequal user currentuser %}
<h1>Welcome!</h1>
{% endifequal %}
```
Verilen arguman sabit bir string olabilir ve çift veya tek tırnak kullanılabilir. İkiside çalışır.
```python
{% ifequal section 'sitenews' %}
<h1>Site News</h1>
{% endifequal %}

{% ifequal section "community" %}
<h1>Community</h1>
{% endifequal %}
```

Tıpkı {% if %}, gibi {% ifequal %} tag seçenek olarak {% else %} destekler:
```python
{% ifequal section 'sitenews' %}
<h1>Site News</h1>
{% else %}
<h1>No News Here</h1>
{% endifequal %}
```
Yanlızca template değişkenleri, string, integer ve decimal numbers kabul edilir: Uygun örnekler
```python
{% ifequal variable 1 %}
{% ifequal variable 1.23 %}
{% ifequal variable 'foo' %}
{% ifequal variable "foo" %}
```
Diğer türler kabul edilmez, örnek olarak Python sözlükleri, listeler ve Boolean olmaz:
```python
{% ifequal variable True %}
{% ifequal variable [1, 2, 3] %}
{% ifequal variable {'key': 'value'} %}
```

Eğer birşeyin doğru olup olmadığını kontrol etmek istiyorsan {% if %} tagını kullan {% ifequal %} a hiç bulaşma.

`ifequal` in alternatifi `if` tagı ve  “==” operatörüdür.
`ifnotequal` in alternatifi `if` tagı ve  “!=” operatörüdür.

### Comments
### Yorumlar

Html ve Python da olduğu gibi Django yorum satırlarına izin verir ve `{# #}` kullanılır:

`{# This is a comment #}`

Template render edildiğinde commentler(yorumlar) görüntülenmez. Bu sözdizimi ile çoklu satır yorumları yazamazsınız, bu kısıtlama performansı(başarımı) arttırır.

Çoklu satır yorum yazmak istiyorsan `{% comment %}` kullan, ayrıca iç içe yorum satırı kullanamazsın:
```python
{% comment %}
This is a
multi-line comment.
{% endcomment %}
```

### Filters
### Filtreler

Daha önce de açıklandığı gibi template filtreleri değişkenlerin değerlerini göstermeden önce değiştirmenin basit yoludur. Filtreler pipe karakterinin kullanırlar. "|":

`{{ name|lower }}`

Yukarıdaki örnek `{{ name }}` değişkeninin değerinin göstermeden önce `lower` filtresini uygular ve stringi küçük harfe çevirir. Filtreler uc uca bağlanabilir. (Filters can be chained böyle yazınca daha karizma oluyor lakin :)) 
Aşağıdaki örnek listenin ilk elemanını alıp onun harflerini büyük farf yapar:

`{{ my_list|first|upper }}`

Bazı filtreler arguman alabilir. Filtrelerin argumanları iki noktadan sonra gelir ve her zaman çift tırnak içinde yer alır. Örnek:

`{{ bio|truncatewords:"30" }}`

Yukarıdaki örnek bio değişkeninin ilk 30 kelimesini gösterir.

Aşağıda en çok kullanılan filtrelerin listesini inceleyelim:

* `addslashes`: Ters bölü, tek tırnak, çift tırnak karakterlerinden önce ters bölü işareti ekler. Kaçış dizileri için faydalırdır. Örnek: `{{ value|addslashes }}`
* `date`: date veya datetime nesnelerini verilen parametreye göre biçimlendirir. Örnek: `{{ pub_date|date:"F j, Y" }}` 
* `length`: Değişkenin uzunluğunu döndürür. Liste için liste eleman sayısını, karakter dizisi için karakter sayısını. Eğer değişken tanımlı değilse 0 döndürür.

## Philosophies and Limitations
## Felsefe ve Sınırlamalar

Artık Django Template Language(DTL) için bir hissiyat oluşturduk :) Sanırım DTL arkasındaki basit tasarım felsefesini açıklama zamanı geldi. İlk olarak olarak DTL sınırlamaları kasıtlı yapılmıştır. 

Bu gün için Django çekirdeğindeki felsefe:

1. Mantığın sunuştan ayrılması
2. Fazlalıktan vazgeç
3. HTML'e bağlı olma
4. XML kötüdür
5. Tasarımcıyı yeterli say
6. Whitespace'lere(boşluklara) olduğu gibi davran
7. Programlama dili icat etme
8. Emniyet ve güvenlik garantisi
9. Genişletilebilir


## 1. Mantığın sunuştan ayrılması
Bir template sistemi sunum ve sunumla ilgili mantığı kontrol eden bir araçtır - hepsi bu kadar. Template sistemi, bu temel hedefin ötesine geçen işlevleri desteklememelidir.

## 2. Fazlalıktan vazgeç
Dinamik Web sitelerinin çoğunluğu, genel bir site tasarımı - ortak bir üstbilgi, altbilgi, gezinme çubuğu vb. Kullanır. Django şablon sistemi, bu öğeleri tek bir yerde saklamayı kolaylaştırmalı ve yinelenen kod ortadan kaldırılmalıdır. Şablon devralmasının ardındaki felsefe budur.

## 3. HTML'e bağlı olma
Şablon sistemi yalnızca HTML çıktısı olarak tasarlanmamalıdır. Diğer metin tabanlı biçimleri üretmede ya da yalnızca düz metinlerde de eşit derecede iyi olmalıdır.

## 4. XML template dili için kullanılmamalıdır
Şablonları ayrıştırmak için bir XML altyapısı kullanmak, şablonları düzenleme konusunda insan hatalarının yepyeni bir dünyasını getirir ve şablon işlemede kabul edilemez düzeyde bir yük getirir.

## 5. Tasarımcıyı yeterli say
Şablon sistemi, şablonların mutlaka Dreamweaver gibi WYSIWYG düzenleyicilerinde güzel görüntülenecek şekilde tasarlanmamalıdır. Bu, bir sınırlamadan çok ağırdır ve sözdiziminin olduğu kadar güzel olmasına izin vermez.

Django, şablon yazarlarının HTML'yi doğrudan düzenlemeyi rahatça yapmalarını bekler.

## 6. Whitespace'lere(boşluklara) olduğu gibi davran
Şablon sistemi boşluklarla büyülü şeyler yapmamalıdır. Bir şablon boşluk içeriyorsa, sistem boşlukları metinde işlem görürken ele almalı - sadece onu görüntülemelidir. Şablon etiketinde olmayan herhangi bir boşluk görüntülenmelidir.

## 7. Programlama dili icad etme
Şablon sistemi kasıtlı olarak aşağıdakilere izin vermez:

    Değişkenlere atama
    İleri mantık

Hedef, bir programlama dili yaratmak değildir. Amaç, sunumla ilgili kararlar almak için gerekli olan, dallanma ve döngü gibi programlama- işlevselliğini sunmaktır.

Django şablon sistemi şablonların çoğunlukla programcılar değil tasarımcılar tarafından yazıldığını ve bu nedenle Python bilgisini üstlenmemesi gerektiğini tanır.

## 8. Emniyet ve güvenlik garantisi
Şablon sistemi, ilk geldiği şekli ile, veritabanı kayıtlarını silmek için kullanılan komutlar gibi kötü amaçlı kodların dahil edilmesini yasaklamalıdır. Şablon sisteminin keyfi Python koduna izin vermemesinin bir başka nedeni de budur.

## 9. Genişletilebilir
Şablon sistemi, gelişmiş şablon yazarlarının şablon teknolojisini genişletmek isteyebileceğini anlamalıdır. Özel şablon etiketleri ve filtrelerinin arkasındaki felsefe budur.

Üstteki 9 maddeyi google translate kullanarak çevirdim ve fark ettim ki google translate metinleri bende daha anlaşılır çeviriyor. Bu sayfadan sonra çeviriler kısmında google translate'i kullanmayı planlıyoru. :) 

