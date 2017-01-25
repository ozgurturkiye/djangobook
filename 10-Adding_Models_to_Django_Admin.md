## Adding Models to Django Admin
## Django Admin'e Modeller Ekleme

Henüz yapmadığımız önemli bir bölüm var. Kendi modellerimizi yönetim sitesine ekleyelim, böylece bu güzel arayüzü kullanarak özel veritabanı tablolarımıza nesneler ekleyebilir, değiştirebilir ve silebilirsiniz. `books` örneğini, Bölüm 3'te tanımladığımız Bölüm 4'ten devam edeceğiz: Yayıncı, Yazar ve Kitap. `books` dizininde (mysite/books) `startapp`, `admin.py` adlı bir dosya oluşturmuş olmalıydı, değilse, kendiniz yaratın ve aşağıdaki kod satırlarını yazın:

```python
from django.contrib import admin
from .models import Publisher, Author, Book

admin.site.register(Publisher)
admin.site.register(Author)
admin.site.register(Book)
```

Bu kod, Django yönetim sitesine bu modellerin her biri için bir arabirim sunmasını söyler. Bunu yaptıktan sonra Web tarayıcınızdaki (http://127.0.0.1:8000/admin/) yönetici ana sayfanıza gidin; Yazarlar, Kitaplar ve Yayıncılar için bağlantılar içeren bir "Kitaplar" bölümü görmelisiniz. (Değişikliklerin yürürlüğe girmesi için geliştirme sunucusunu durdurup başlatmanız gerekebilir.) Artık bu üç modelin her biri için tamamen işlevsel bir yönetici arayüzüne sahipsiniz. Kolaydı!

Veritabanınızı bazı verilerle doldurmak için kayıt eklemek veya değiştirmek için biraz zaman ayırın. Bölüm 4'ün `Publisher` nesneleri oluşturma örneklerini izlediyseniz (ve bunları silmediyseniz), zaten bu kayıtları yayıncı değişim listesi sayfasında göreceksiniz.

Buraya değmeye değer bir özellik, her ikisi de `Book` modelinde görünen yabancı anahtarların ve çok-çoklu ilişkilerin yönetim sitesinde ele alınmasıdır. Bir hatırlatma olarak, `Book` modelinin görünümü şöyledir:

```python
class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()

    def __str__(self):
        return self.title 
```

Django admin sitesinin "Add book" sayfası (http://127.0.0.1:8000/admin/books/book/add/) üzerinde, yayıncı (ForeignKey) bir seçim kutusu ile temsil edilir ve yazarlar alanı (a ManyToManyField), çoktan seçmeli bir kutuyla gösterilir. Her iki alan da, o türün ilgili kayıtlarını eklemenizi sağlayan yeşil bir artı işaret simgesinin yanında bulunur.

Örneğin, "Yayıncı" alanının yanındaki yeşil artı işaretini tıklarsanız, bir yayıncı eklemenizi sağlayan bir açılır pencere görürsünüz. Açılır pencerede yayıncıyı başarıyla oluşturduktan sonra, "Kitabı ekle" formu yeni oluşturulan yayıncı ile güncellenecektir. Becerikli :)

## Making Fields Optional
## İsteğe Bağlı Alan Yapma

Bir süre admin sitesinde gezdikten sonra bir sınırlama fark edersiniz - düzenleme formları doldurulacak her alanı gerektirir, oysa çoğu durumda bazı alanların isteğe bağlı olmasını istersiniz. Örneğin, Yazar modelimizin e-posta alanının isteğe bağlı olmasını istediğimizi, yani boş bir dizenin kullanılmasına izin verilmesini diyelim. Gerçek dünyada, her yazar için bir e-posta adresiniz olmayabilir.

`email` alanının isteğe bağlı olduğunu belirtmek için, `Author` modelini düzenleyin (bu, Bölüm 4'ten hatırlatacağınız üzere, `mysite/books/models.py`'de yaşar). `email` alanına şu şekilde `blank=True` ekleyin:

```python
class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField(blank=True)
```

Bu, Django'ya yazarların e-posta adresleri için gerçekten boş bir değere izin verildiğini söyler. Varsayılan olarak, tüm alanların `blank=False`, boş değerlere izin verilmediği anlamına gelir.

Burada ilginç bir şeyler var. Şimdiye kadar, `__str__()` yöntemi dışında, modellerimiz veritabanı tablolarımızın - SQL `CREATE TABLE` ifadelerinin pythonic ifadelerinin - tanımlamaları olarak hizmet etmiştir. `blank=True`'ı eklerken, modelimizi, veritabanı tablosunun neye benzediğinin basit bir tanımı ötesine genişletmeye başladık.

Şimdi, model sınıfımız, Yazar nesnelerinin ne oldukları ve neler yapabileceği konusunda daha zengin bir bilgi topluluğu haline geliyor. E-posta alanı veritabanında bir VARCHAR sütunuyla temsil edilmekle kalmaz; Django yönetim sitesi gibi bağlamlarda isteğe bağlı bir alan.

Bu `blank=True` olarak ekledikten sonra, "Yazar ekle" düzenleme formunu yeniden yükleyin (http://127.0.0.1:8000/admin/books/author/add/) ve alanın etiketini göreceksiniz - "E-posta "- artık kalın yazılı değil. Bu, zorunlu bir alan olmadığını gösterir. Artık yazarları e-posta adresleri göndermek zorunda kalmadan ekleyebilirsiniz; Alan boş bırakılırsa, kıpkırmızı "Bu alan gerekli" mesajını almazsınız.

## Making Date and Numeric Fields Optional
## Tarih ve Sayısal Alanların İsteğe Bağlı Olarak Oluşturulması

`blank=True` ile ilgili ortak bir kaza, tarih ve sayısal alanlarla ilgisi olmakla birlikte, adil bir miktar arka plan açıklaması gerektirir. SQL boş değerleri belirten kendi yoluna sahiptir - özel bir değer `NULL` denir. `NULL`, "bilinmeyen" veya "geçersiz" veya başka bir uygulamaya özgü anlam anlamına gelebilir. SQL'de boş bir dizgeden farklı bir `NULL` değeri vardır, tıpkı özel Python nesnesi `None`, boş bir Python dizesinden farklıdır (""). Sonuç olarak NULL özel bir tanımlama.

Bu, belirli bir karakter alanının (ör. Bir `VARCHAR` sütununun) hem `NULL` değerlerini hem de boş dize değerlerini içermesi olasılığı anlamına gelir. Bu, istenmeyen belirsizlik ve karışıklığa neden olabilir: "Neden bu kayıt NULL var, ancak bu diğerinde boş bir dize var mı? Bir fark var mı yoksa veri tutarsız şekilde mi girildi? "Ve:" Boş bir değere sahip tüm kayıtları nasıl edinebilirim - hem boş kayıtları hem de boş dizeleri aramalı mıyım yoksa yalnızca boş olanları mı seçmeliyim? Dizeleri? "

Bu belirsizliği önlemek için, Django'nun otomatik oluşturduğu `CREATE TABLE` deyimleri (Bölüm 4'te ele alınmıştır) her sütun tanımına açık bir `NOT NULL` ekler. Örneğin, Yazar Modelimiz için Bölüm 4'ten üretilen bildirim:

```sql
CREATE TABLE "books_author" (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(40) NOT NULL,
    "email" varchar(75) NOT NULL
);
```

Çoğu durumda, bu varsayılan davranış, uygulamanız için en ideal yöntemdir ve sizi veri uyumsuzluğundan kurtarır. Ve bir karakter alanını boş bıraktığınızda, boş bir dize (NULL değeri değil) ekleyen Django yönetim sitesi gibi Django'nun geri kalanıyla güzel çalışıyor.

Ancak boş dizeleri geçerli değerler olarak kabul etmeyen veritabanı sütun türleriyle - örneğin tarihler, saatler ve sayılar gibi bir istisna vardır. Bir tarih veya tam sayı sütununa boş bir dize eklemeyi denerseniz, kullandığınız veritabanına bağlı olarak büyük olasılıkla bir veritabanı hatası alırsınız. (Sıkı PostgreSQL burada bir istisna oluşturacaktır; MySQL, kullandığınız sürüme, günün saatine ve aya ait faza bağlı olarak onu kabul edebilir veya kullanmayabilir.)

Bu durumda, `NULL` boş bir değeri belirtmenin tek yoludur. Django modellerinde, bir alana `null=True` ekleyerek NULL'ya izin verileceğini belirtmiş olursunuz. Bunu söylemenin uzun bir yoludur: Bir tarih alanında boş bir değer (örn., DateField, TimeField, DateTimeField) veya sayısal alan (örneğin, IntegerField, DecimalField, FloatField) izin vermek isterseniz, hem `null=True` ve `blank=True`değerlerini girmelisiniz.

Örnek vermek adına, `Book` modelimizi boş bir `publication_date`'ye izin vermek için değiştirelim. İşte revize edilmiş kod:

```python
class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField(blank=True, null=True)
```

`null = True` eklemek, `blank = True` eklemekten daha karmaşıktır; çünkü `null = True`, veritabanının semantiklerini değiştirir; yani, `publication_date` alanından `NOT NULL` kaldırmak için `CREATE TABLE` deyimini değiştirir. Bu değişikliği tamamlamak için veritabanını güncellememiz gerekecek. Django, çeşitli nedenlerle, veritabanı şemalarındaki değişiklikleri otomatikleştirmeye çalışmaz; bu nedenle, bir modelde böyle bir değişiklik yaptığınızda `python manage.py migrate` geçiş komutunu yürütmek sizin sorumluluğunuzdadır. Bunu yönetici sitesine geri getirirken, şimdi "Kitap ekle" düzenleme formu boş yayın tarihi değerlerine izin vermelidir.

## Customizing Field Labels
## Alan Etiketlerini Özelleştirme

Yönetici sitenin düzenleme formlarında, her alanın etiketi model alan adından oluşturulur. Algoritma basit: Django sadece boşluklarla alt çizginin yerini alıyor ve ilk karakteri büyük harfle yazıyor, örneğin, `Book` modelinin `publication_date` alanı "Publication date" etiketine sahip.

Bununla birlikte, alan adları her zaman güzel yönetici alan etiketlerine borç vermez, bu nedenle bazı durumlarda bir etiketi özelleştirmek isteyebilirsiniz. Bunu, uygun model alanındaki `verbose_name` belirtilerek yapabilirsiniz. Örneğin, `Author.email` alanının etiketini tire ile "e-mail" nasıl değiştirebileceğimiz aşağıda açıklanmıştır:

```python
class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField(blank=True, verbose_name ='e-mail')
```

Bu değişikliği yapın ve sunucuyu yeniden yükleyin ve alanın yazar düzenleme formundaki yeni etiketini görmelisiniz. Her zaman büyük harfle yazılmış olmamalı (ör. "ABD eyaleti") bir `verbose_name`'in ilk harfini büyük harf kullanmamalısınız. Django, gerektiğinde onu otomatik olarak büyük harfle kullanır ve büyük harf kullanımını gerektirmeyen diğer yerlerde tam `verbose_name` değerini kullanır.

## Custom ModelAdmin classes
## Özel ModelAdmin sınıfları

Şimdiye kadar yaptığımız değişiklikler - `blank = True`, `null = True` ve `verbose_name` - model düzeyindeki değişikliklerdir, yönetim düzeyindeki değişiklikler değil. Yani, bu değişiklikler modelin temel bir parçasıdır ve sadece yönetici sitesi tarafından kullanılmaktadır; Burada admin'e özgü hiçbir şey yok.

Bunların ötesinde, Django yönetim sitesi, yönetim sitesinin belirli bir model için nasıl çalıştığını özelleştirebilmenize olanak tanıyan zengin seçenekler sunar. Bu tür seçenekler, belirli bir yönetim sitesi örneğinde belirli bir model için yapılandırma içeren sınıflar olan `ModelAdmin sınıfları`nda bulunur.
