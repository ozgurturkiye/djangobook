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
