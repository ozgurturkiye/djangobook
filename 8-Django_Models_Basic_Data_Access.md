# Django Models: Basic Data Access
# Django Modelleri: Temel Veri Erişimi

Bir model oluşturduktan sonra, Django otomatik olarak bu modellerle çalışmak için üst düzey bir Python API'si sağlar. `python manage.py shell` çalıştırıp aşağıdakileri yazarak deneyin:

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

1. İlk olarak, `Publisher` model sınıfımızı içe aktarıyoruz. Bu, yayıncıları içeren veritabanı tablosuyla etkileşim kurmamızı sağlar.
2. Her bir alanın (ad, adres vb.) Değerleriyle örneklendirerek bir `Publisher` nesnesi oluşturduk.
3. Nesneyi veritabanına kaydetmek için save() yöntemini çağırın. Perde arkasında, Django burada bir SQL INSERT deyimini yürütür.
4. Veritabanından yayıncıları almak için, tüm yayıncıların bir kümesi olarak düşünebileceğiniz Publisher.objects özniteliğini kullanın. Publisher.objects.all() ifadesi ile veritabanındaki tüm Publisher nesnelerinin bir listesini getirin. Sahne arkasında, Django burada bir SQL SELECT deyimini yürütür.

Bu örneğinden açık olması için bir şey söylemeye değer. Django modeli API'sini kullanarak nesneler oluştururken, save () yöntemini çağırana dek Django nesneleri veritabanına kaydetmez:

```python
p1 = Publisher(...)
# At this point, p1 is not saved to the database yet!
p1.save()
# Now it is.
```

Bir nesne oluşturmak ve veritabanına tek bir adımda kaydetmek istiyorsanız, objects.create () yöntemini kullanın. Bu örnek yukarıdaki örneğe eşdeğerdir:

```python
>>> p1 = Publisher.objects.create(name='Apress',
...     address='2855 Telegraph Avenue',
...     city='Berkeley', state_province='CA', country='U.S.A.',
...     website='http://www.apress.com/')
>>> p2 = Publisher.objects.create(name="O'Reilly",
...     address='10 Fawcett St.', city='Cambridge',
...     state_province='MA', country='U.S.A.',
...     website='http://www.oreilly.com/')
>>> publisher_list = Publisher.objects.all()
>>> publisher_list 
[<Publisher: Publisher object>, <Publisher: Publisher object>]
```

Doğal olarak, Django veritabanı API'si ile oldukça fazla şey yapabilirsiniz - ancak önce küçük bir sıkıntıya dikkat edelim.

## Adding Model String Representations
## Model Karakter Dizisi Gösterimlerini Ekleme

Yayıncı listesini yazdırdığımızda elimizdeki tek şey, Yayıncı nesnelerini birbirinden ayırmak zorlaşan bu yararsız ekrandı:

```python
[<Publisher: Publisher object>, <Publisher: Publisher object>]
```

Bunu, __str __() adlı bir yöntemi `Publisher` sınıfımıza ekleyerek kolayca düzeltebiliriz. Bir __str __() yöntemi Python'a bir nesnenin insan tarafından okunabilir bir gösterimini nasıl göstereceğini söyler. Bunu, üç modele bir __str __() yöntemi ekleyerek görebilirsiniz:

```python
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    def __str__(self):
        return self.name

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

    def __str__(self):
        return u'%s %s' % (self.first_name, self.last_name)

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()

    def __str__(self):
        return self.title
```

Gördüğünüz gibi bir __str __() yöntemi, bir nesnenin gösterimini döndürmek için yapması gereken her şeyi yapabilir. Burada, Publisher ve Book için __str __() yöntemleri sırasıyla yalnızca nesnenin adını ve başlığını döndürür ancak Author için __str __() biraz daha karmaşıktır - bir boşlukla ayrılmış `first_name` ve `last_name` alanlarını birlikte parçalar. __str __() için tek şart, bir dize nesnesi döndürmesidir. Eğer __str __() bir dize nesnesi döndürmezse - bir tam sayı döndürürse - Python gibi bir mesajla bir `TypeError` üretecektir:

```python
TypeError: __str__ returned non-string (type int).
```

`__str__()` değişiklikleri etkili olması için Python kabuğundan çıkıp python manage.py kabuğu ile tekrar girin. (Bu, kod değişikliklerinin etkili olmasını sağlamanın en basit yoludur.) Şimdi Publisher nesnelerinin listesi anlamak daha kolaydır:

```python
>>> from books.models import Publisher
>>> publisher_list = Publisher.objects.all()
>>> publisher_list
[<Publisher: Apress>, <Publisher: O'Reilly>]
```

Tanımladığınız herhangi bir modelin __str__() yöntemi olduğundan emin olun - yalnızca etkileşimli tercümanı kullanırken kendi kolaylığınız için değil, aynı zamanda Django nesneleri görüntülemesi gerektiğinde birkaç yerde __str__() çıktısını kullandığından emin olun. Son olarak, __str__() işlevinin davranış eklemenin iyi bir örneği olduğunu unutmayın.
Modeller. Bir Django modeli, bir nesne için veritabanı tablosu düzeninden daha fazlasını açıklar; Ayrıca, nesnenin nasıl yapılacağını bildiği herhangi bir işlevselliği de açıklar. __str__() bu tür bir işlevselliğe bir örnektir - bir model kendisini nasıl göstereceğini biliyor.

## Inserting and Updating Data
## Verileri Ekleme ve Güncelleme

Bunu zaten gördünüz: Veritabanınıza bir satır eklemek için önce modelinizin bir örneğini anahtar kelime bağımsız değişkenlerini kullanarak oluşturun:

```python
>>> p = Publisher(name='Apress',
...         address='2855 Telegraph Ave.',
...         city='Berkeley',
...         state_province='CA',
...         country='U.S.A.',
...         website='http://www.apress.com/')
```

Yukarıda belirttiğimiz gibi, bir model sınıfını örneklendiren bu hareket veritabanıyla temas kurmaz. Kayıt, save () işlevini çağırana dek, veritabanına kaydedilmez:

```python
>>> p.save()
```

SQL'de bu kabaca şu şekilde çevrilebilir:

```SQL
INSERT INTO books_publisher
    (name, address, city, state_province, country, website)
VALUES
    ('Apress', '2855 Telegraph Ave.', 'Berkeley', 'CA',
     'U.S.A.', 'http://www.apress.com/');
```

Yayıncı modeli, birincil kimliği artıran bir otomatik kimlik kullandığından, save () işlevinin ilk çağrısı,
Bir şey daha yapar: kayıt için birincil anahtar değerini hesaplar ve onu örnekteki id özniteliğine ayarlar:

```python
>>> p.id
52    # this will differ based on your own data Subsequent calls to `save()` will sav\
e the record in place, without creating a new record (i.e.,
performing an SQL `UPDATE` statement instead of an `INSERT`):

>>> p.name = 'Apress Publishing'
>>> p.save()
```

Yukarıdaki save() ifadesi kabaca aşağıdaki SQL ile sonuçlanır:

```SQL
UPDATE books_publisher SET
    name = 'Apress Publishing',
    address = '2855 Telegraph Ave.',
    city = 'Berkeley',
    state_province = 'CA',
    country = 'U.S.A.',
    website = 'http://www.apress.com'
WHERE id = 52;
```

Evet, yalnızca tek bir alanı değil, tüm alanların güncelleneceğini unutmayın. Uygulamanıza bağlı olarak, yarış koşullarına neden olabilir. Bu (biraz farklı) sorguyu nasıl yürütüleceğini öğrenmek için aşağıdaki "Bir Bildirgedeki Birden Çok Nesneyi Güncelleştirme" konusuna bakın:

```sql
UPDATE books_publisher SET
    name = 'Apress Publishing'
WHERE id=52;
```

## Selecting Objects
## Nesneleri Seçme
