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

Veritabanı kayıtlarını nasıl oluşturacağınızı ve güncelleyeceğinizi bilmek çok önemlidir, ancak oluşturacağınız Web uygulamalarının mevcut nesnelerin yenilerini oluşturmaktan çok sorgulamasını yapmanız olasılığı vardır. Belli bir model için her kaydı almak için bir yol bulmuştık:

```python
>>> Publisher.objects.all()
[<Publisher: Apress>, <Publisher: O'Reilly>]
This roughly translates to this SQL:

SELECT id, name, address, city, state_province, country, website
FROM books_publisher;
```

* Django, verileri ararken SELECT * 'ı kullanmaz ve bunun yerine tüm alanları açıkça listelediğine dikkat edin.
Bu tasarım gereğidir: Bazı durumlarda SELECT * daha yavaş olabilir ve (daha da önemlisi) listeleme alanları, Python Zen'in bir öğretisini daha yakından takip eder: "Açıkça örtükten iyidir." Python Zen hakkında daha fazla bilgi için Python prompt'a `import this `yazmayı deneyin.

Bu Publisher.objects.all () satırının her bir parçasını yakından inceleyelim:

1. İlk olarak, tanımladığımız model olan Publisher'ımız var. Burada şaşırtıcı değilsiniz: Verilere bakmak istediğinizde, o veri için modeli kullanırsınız.
2. Daha sonra, nesneler özelliğine sahibiz. Buna bir yönetici denir. Yöneticiler Bölüm 09'da ayrıntılı olarak ele alınmaktadır. Şimdilik, bilmeniz gereken tek şey, yöneticilerin, en önemlisi veri aramayı da içeren veri üzerindeki "tablo düzeyinde" işlemlere dikkat etmeleri. Tüm modeller otomatik olarak bir nesne yöneticisi alır; Model örnekleri aramak istediğinizde kullanabilirsiniz.
3. Son olarak, all() metoduna sahibiz. Bu, veritabanındaki tüm satırları döndüren nesne yöneticisinde bir yöntemdir. Bu nesne bir listeye benziyor olsa da, aslında bir QuerySet - veritabanından belirli bir satır kümesini temsil eden bir nesne. Ek C, QuerySet'leri ayrıntılı olarak ele almaktadır. Bu bölümün geri kalan kısmı için, onlara taklit ettikleri listeler gibi davranacağız.

Herhangi bir veritabanı araması bu genel modeli takip edecektir - sorgulanmak istediğimiz modele eklenmiş yönetici üzerindeki yöntemleri çağıracağız.

## Filtering Data
## Verileri Filtreleme

Doğal olarak, bir veritabanından her şeyi bir kerede seçmek nadirdir; Çoğu durumda verilerinizin bir alt kümesiyle ilgilenmek isteyeceksiniz. Django API'sında, verilerinizi filter() yöntemini kullanarak filtreleyebilirsiniz:

```python
>>> Publisher.objects.filter(name='Apress')
[<Publisher: Apress>]
```

filter(), uygun `SQL WHERE` yan tümcelerine tercüme edilen anahtar kelime bağımsız değişkenlerini alır. Önceki örnek, böyle bir şeye tercüme edilebilir:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE name = 'Apress';
```

İşlemleri daha da daraltmak için filter() öğesine birden çok bağımsız değişken geçirebilirsiniz:

```python
>>> Publisher.objects.filter(country="U.S.A.",
state_province="CA")
[<Publisher: Apress>]
```

Bu çoklu argümanlar `SQL AND` cümlelerine tercüme edilir. Böylece kod snippet'inde yer alan örnek şu şekilde çevirir:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE country = 'U.S.A.'
AND state_province = 'CA';
```

Varsayılan olarak, aramalarda tam eşleme aramaları yapmak için SQL `=` işleci kullanıldığına dikkat edin.
Diğer arama türleri de mevcuttur:

```python
>>> Publisher.objects.filter(name__contains="press")
[<Publisher: Apress>]
```

Bu, `name` ve `contains` arasında bir çift altçizgi. Python'un kendisi gibi, Django, "sihir" olayının meydana geldiğini işaretlemek için çift altçizgi kullanır - burada, `__contains` bölümü Django tarafından bir `SQL LIKE` ifadesine dönüştürülür:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE name LIKE '%press%';
```

`İcontains` (büyük / küçük harf duyarlı olmayan `LIKE`), `startswith` ve `endswith` ve `range` (SQL `BETWEEN` sorguları) dahil birçok başka arama türü mevcuttur. Ek C, bu arama türlerinin tümünü ayrıntılı olarak açıklamaktadır.

## Retrieving Single Objects
## Tek Nesneleri Alma

`filter()` örnekleri, bir listeye benzer şekilde davranabileceğiniz bir `QuerySet` döndürdü. Bazen, bir listeye değil yalnızca tek bir nesneyi getirmek daha elverişlidir. Bunun için `get()` yöntemi şu şekildedir:

```python
>>> Publisher.objects.get(name="Apress")
<Publisher: Apress>
```

Bir liste yerine (yalnızca `QuerySet`), yalnızca tek bir nesne döndürülür. Bu nedenle, birden çok nesne ile sonuçlanan bir sorgu bir istisna yaratacaktır:

```python
>>> Publisher.objects.get(country="U.S.A.")
Traceback (most recent call last):
    ...
MultipleObjectsReturned: get() returned more than one Publisher -- it returned 2! Loo\
kup parameters were {'country': 'U.S.A.'}
```

Hiçbir nesne döndürmeyen bir sorgu da bir istisna yaratır:

```python
>>> Publisher.objects.get(name="Penguin")
Traceback (most recent call last):
    ...
DoesNotExist: Publisher matching query does not exist.
```

`DoesNotExist` istisnası, modelin sınıfının bir özniteliğidir - `Publisher.DoesNotExist`. Uygulamalarınızda şu gibi bu istisnaları yakalamak isteyeceksiniz:

```python
try:
    p = Publisher.objects.get(name='Apress')
except Publisher.DoesNotExist:
    print ("Apress isn't in the database yet.")
else:
    print ("Apress is in the database.")
```

## Ordering Data
## Veri Sıralama

Önceki örneklerle uğraşırken, nesnelerin görünüşte rastgele bir sırayla döndüğünü keşfedebilirsiniz. Her şeyi hayal etmiyorsun; Şimdiye dek, veritabanına sonuçlarını nasıl sıralanabileceğimizi söylemedik, bu nedenle veritabanının seçtiği keyfi bir sırayla verileri geri alıyoruz. Django uygulamalarınızda muhtemelen sonuçlarınızı belirli bir değere, yani alfabetik olarak siparişi vermek isteyeceksinizdir. Bunu yapmak için order_by() yöntemini kullanın:

```python
>>> Publisher.objects.order_by("name")
[<Publisher: Apress>, <Publisher: O'Reilly>]
```

Bu, daha önceki all() örneğinden çok farklı görünmüyor, ancak şimdi SQL'de belirli bir sıralama var:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
ORDER BY name;
```

İstediğiniz herhangi bir alana göre sıralama yapabilirsiniz:

```python
>>> Publisher.objects.order_by("address")
 [<Publisher: O'Reilly>, <Publisher: Apress>]

>>> Publisher.objects.order_by("state_province")
 [<Publisher: Apress>, <Publisher: O'Reilly>]
```

Birden fazla alana sıralama yapmak için (birinci alanın aynı olduğu durumlarda sıralamanın belirginleşmesi için ikinci alan kullanılır) birden çok bağımsız değişken kullanın:

```python
>>> Publisher.objects.order_by("state_province",
"address")
 [<Publisher: Apress>, <Publisher: O'Reilly>]
```

Alan adının önüne "-" (eksi karakter) ekleyerek ters sıralamayı da belirtebilirsiniz:

```python
>>> Publisher.objects.order_by("-name")
[<Publisher: O'Reilly>, <Publisher: Apress>]
```

Bu esneklik kullanışlı olmasına rağmen, `order_by()` işlevinin her zaman kullanılması oldukça tekrarlayıcı olabilir. Çoğu zaman genellikle sipariş vermek istediğiniz belirli bir alana sahip olursunuz. Bu gibi durumlarda, Django modelde varsayılan bir sipariş belirtmenizi sağlar:

```python
class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['name']
```

Burada, yeni bir kavram getirdik: Sınıf, Meta sınıfı `class Meta`, Yayıncı sınıfı tanımında gömülü olan bir sınıftır (diğer bir deyişle, Sınıf Yayıncısı içinde girintilendirilmiş). Bu `Meta` sınıfını, çeşitli model özel seçeneklerini belirtmek için herhangi bir modelde kullanabilirsiniz. `Meta` seçeneklerinin eksiksiz bir referansı Ek B'de bulunmaktadır, ancak şu an için, sipariş seçeneği ile ilgileniyoruz. Bunu belirttiyseniz, `order_by()` ile açıkça belirtilmedikçe, Django'ya, Django veritabanı API'sı ile alındığında her Yayıncı nesnesinin `name` alanı ile sıralanması gerektiğini söyler.

## Chaining Lookups
## Zincirleme Arama

Verileri nasıl filtreleyebileceğinizi ve bunu nasıl sıralayabileceğinizi öğrendiniz. Çoğu zaman, elbette, her ikisini de yapmanız gerekecek. Bu durumlarda, aramaları bir araya getirerek "zincirleme" yaparsınız:

```python
>>> Publisher.objects.filter(country="U.S.A.").order_by("-name")
[<Publisher: O'Reilly>, <Publisher: Apress>]
```

Beklediğiniz gibi, bu hem bir `WHERE` hem de `ORDER BY` ile bir `SQL` sorgusuna çevirir:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE country = 'U.S.A'
ORDER BY name DESC;
```

## Slicing Data
## Verileri Dilimleme

Bir diğer sık rastlanan ihtiyaç sadece sabit sayıda satır aramaktır. Veritabanınızda binlerce yayıncının olduğunu düşünün, ancak yalnızca birincisini görüntülemek istiyorsanız. Bunu Python'un standart dilim dilimleme sözdizimini kullanarak yapabilirsiniz:

```python
>>> Publisher.objects.order_by('name')[0]
<Publisher: Apress>
```

Bu kabaca şu şekilde çevrilir:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
ORDER BY name
LIMIT 1;
```

Benzer şekilde, belirli bir veri alt kümesini Python alan dilimleme sözdizimini kullanarak alabilirsiniz:

```python
>>> Publisher.objects.order_by('name')[0:2]
```

Bu, iki nesneyi kabaca şunlara çevirerek döndürür:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
ORDER BY name
OFFSET 0 LIMIT 2;
```

* Negatif dilimlemenin desteklenmediğini unutmayın:

```python
>>> Publisher.objects.order_by('name')[-1]
Traceback (most recent call last):
  ...
AssertionError: Negative indexing is not supported.
```

Buna rağmen, kolayca sorunun çevresinden dolaşabilirsiniz. Bunun gibi order_by() ifadesini değiştirmeniz yeterlidir:

```python
>>> Publisher.objects.order_by('-name')[0]
```

## Updating Multiple Objects in One Statement
## Bir Deyimde Birden Çok Nesneyi Güncelleme

Model save() yöntemi, bir satırdaki tüm sütunları güncelleştiren "Verileri Ekleme ve Güncelleme" bölümünde belirttik. Uygulamanıza bağlı olarak, yalnızca bir sütun alt kümesini güncellemek isteyebilirsiniz. Örneğin, adı 'Apress' olan 'Apress Publishing' olarak değiştirmek için Apress Publisher'ı güncellemek istediğimizi varsayalım. save() işlevini kullanarak şu şekilde görünür:

```
>>> p = Publisher.objects.get(name='Apress')
>>> p.name = 'Apress Publishing'
>>> p.save()
```

Bu kabaca aşağıdaki SQL'ye çevirir:

```SQL
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE name = 'Apress';

UPDATE books_publisher SET
    name = 'Apress Publishing',
    address = '2855 Telegraph Ave.',
    city = 'Berkeley',
    state_province = 'CA',
    country = 'U.S.A.',
    website = 'http://www.apress.com'
WHERE id = 52;
```

(Bu örnekte, Apress'in bir yayıncı kimliğinin 52 olduğunu varsayıyoruz.) Bu örnekte, Django'nun save() yönteminin sadece ad sütununu değil, tüm sütun değerlerini ayarladığını görebilirsiniz. Başka bir işlem nedeniyle veritabanının diğer sütunlarının değişebileceği bir ortamdaysanız, yalnızca değiştirmek istediğiniz sütunu değiştirmek daha akıllıdır. Bunu yapmak için, `QuerySet` nesneleri üzerinde `update()` yöntemini kullanın. İşte bir örnek:

```python
>>> Publisher.objects.filter(id=52).update(name='Apress Publishing')
```

Buradaki SQL çevirisi çok daha verimli ve yarış koşulları şansı yok:

```SQL
UPDATE books_publisher
SET name = 'Apress Publishing'
WHERE id = 52;
```

update() yöntemi, birden fazla kaydı toplu olarak düzenleyebileceğiniz anlamına gelen herhangi bir Query Setinde çalışır. Burada herbir kayıtta Ülkeyi 'U.S.A'den' nasıl 'USA' değiştirebilirsiniz gösteriliyor:

* Yanlız filtre(`filter`) kısmını boş brakırsan bütün `country` leri "USA" yaparsın sonra gözlerin dolu dolu olur, ağlarsın...

```python
>>> Publisher.objects.filter(country="U.S.A").update(country="USA")
2 
```

update() yönteminde bir dönüş değeri vardır - kaç kayıt değiştirildiğini gösteren bir tam sayı. Yukarıdaki örnekte 2 var.

## Deleting Objects
## Nesneleri Silme

Bir nesneyi veritabanından silmek için, nesnenin delete() yöntemini çağırmanız yeterlidir:

```python
>>> p = Publisher.objects.get(name="O'Reilly")
>>> p.delete()
>>> Publisher.objects.all()
[<Publisher: Apress Publishing>]
```

Herhangi bir `QuerySet`'in sonucunda `delete()` öğesini çağırarak nesneleri toplu olarak silebilirsiniz. Bu, son bölümde gösterdiğimiz `update()` yöntemine benzer:

* Hatırlatmakta fayda var filter boş olursa yine herşey silinir. ` Publisher.objects.filter().delete() ` -- Bunu yapma demek oluyor ;)

```python
>>> Publisher.objects.filter(country='USA').delete()
>>> Publisher.objects.all().delete()
>>> Publisher.objects.all()
[]
```

Verilerinizi silmek için dikkatli olun! Belirli bir tablodaki tüm verilerin silinmesine karşı bir önlem olarak, Django, tablonuzdaki her şeyi silmek isterseniz `all()` işlevini kullanmanızı ister. Örneğin, bu işe yaramaz:

```python
>>> Publisher.objects.delete()
Traceback (most recent call last):
  File "", line 1, in 
AttributeError: 'Manager' object has no attribute 'delete'
```

Ancak all() yöntemini eklerseniz işe yarayacaktır:

```python
>>> Publisher.objects.all().delete()
If you're just deleting a subset of your data, you don't need to include `all()`.
```

To repeat a previous example:

```python
>>> Publisher.objects.filter(country='USA').delete()
```

## What’s Next?
## Sıradaki Ne? :D

Bu bölümü okuduktan sonra, temel veritabanı uygulamalarını yazabilmek için Django modelleri hakkında yeterli bilgiye sahip olursunuz. Bölüm 9, Django'nun veritabanı katmanının daha gelişmiş kullanımı hakkında bazı bilgiler sağlayacaktır. Modellerinizi tanımladıktan sonra, bir sonraki adım veritabanınızı veri ile doldurmaktır. Eski veriler olabilir, bu durumda Bölüm 21, eski veritabanlarıyla entegrasyon konusunda size tavsiyeler sunar. Verilerinizi sunmak için site kullanıcılarına güvenebilirsiniz, bu durumda Bölüm 6 size, kullanıcı tarafından gönderilen form verilerini nasıl işleyeceğini öğretecektir. Ancak, bazı durumlarda, siz veya ekibiniz verileri elle girmeniz gerekebilir; bu durumda, veriyi girmek ve yönetmek için Web tabanlı bir arabirime sahip olmak faydalı olacaktır. Sonraki bölümde, tam da bu sebeple var olan Django'nun yönetici arayüzü ele alınmaktadır.
