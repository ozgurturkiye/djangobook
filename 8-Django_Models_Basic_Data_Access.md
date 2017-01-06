# Django Models: Basic Data Access
# Django Modelleri: Temel Veri Erişimi

Bir model oluşturduktan sonra, Django otomatik olarak bu modellerle çalışmak için üst düzey bir Python API'si sağlar. Python manage.py kabuğunu çalıştırıp aşağıdakileri yazarak deneyin:

```python
>>> from books.models import Publisher
>>> p1 = Publisher(name='Apress', address='2855 Telegraph Avenue',
...     city='Berkeley', state_province='CA', country='U.S.A.',
...     website='http://www.apress.com/')
>>> p1.save()
>>> p2 = Publisher(name="O'Reilly", address='10 Fawcett St.',
...     city='Cambridge', state_province='MA', country='U.S.A.',
...     website='http://www.oreilly.com/')
>>> p2.save()
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
[<Publisher: Publisher object>, <Publisher: Publisher object>]
```

Bu birkaç kod satırı birkaç şey başarıyor. Önemli noktalar şunlardır:

