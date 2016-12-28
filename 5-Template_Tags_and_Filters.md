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
