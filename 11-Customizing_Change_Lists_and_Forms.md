# Customizing Change Lists and Forms
# Değişiklik Listeleri ve Formları Özelleştirme

Yazar modelimizin değişim listesinde görüntülenen alanları belirterek yönetici özelleştirmesine dalalım. Varsayılan olarak, değişim listesi, her nesne için `__str__()` sonucunu görüntüler. Bölüm 4'te, `__str__()` yöntemini, Yazar nesneleri için ilk adı ve soyadını birlikte gösterecek şekilde tanımladık:

```python
class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField(blank=True, verbose_name ='e-mail')

    def __str__(self):
        return u'%s %s' % (self.first_name, self.last_name)
```

Sonuç olarak, `Yazar` nesnelerinin değişim listesi, Şekil 5-7'de görebileceğiniz gibi, herbirinin ilk adını ve soyadını birlikte görüntüler.

Şekil 5-7: Yazar değişim listesi sayfası

Değişiklik listesi ekranına birkaç tane daha alan ekleyerek bu varsayılan davranışı geliştirebiliriz. Örneğin, her yazarın e-posta adresini listede görmek çok kullanışlı olurdu ve ilk ve son adına göre sıralamayı yapabilmek güzel olurdu. Bunu yapmak için, `Author` modeli için bir `ModelAdmin` sınıfı tanımlayacağız. Bu sınıf, yöneticiyi özelleştirmenin anahtarıdır ve yapmanıza izin veren en temel şeylerden biri, değişiklik listesi sayfalarında görüntülenecek alanların listesini belirtmektir. Bu değişiklikleri yapmak için `admin.py` dosyasını düzenleyin: Not: Bazen `Author` bazen `Yazar` kullanıyorum google çeviri kullandığım için siz anlarsınız ikisini de yinede bilginize sunuyorum :)

```python
from django.contrib import admin
from mysite.books.models import Publisher, Author, Book

class AuthorAdmin(admin.ModelAdmin):
    list_display = ('first_name', 'last_name', 'email')

admin.site.register(Publisher)
admin.site.register(Author, AuthorAdmin)
admin.site.register(Book)
```

İşte yaptıklarımız:

* `AuthorAdmin` sınıfını yarattık. `Django.contrib.admin.ModelAdmin` alt sınıflarının bulunduğu bu sınıf, belirli bir yönetim modeli için özel yapılandırmaya sahiptir. Yalnızca bir özelleştirme belirttik - `list_display`, bu liste, değişim listesi sayfasında görüntülenecek bir alan adı adları grubuna ayarlandı. Bu alan adları elbette modelde olmalıdır.

* `Author`dan sonra `AuthorAdmin` eklemek için `admin.site.register()` çağrısını değiştirdik. Bunu "Yazar modelini `AuthorAdmin` seçenekleri ile kaydettirin." Şeklinde okuyabilirsiniz. `Admin.site.register()` işlevi, isteğe bağlı bir ikinci bağımsız değişken olarak `ModelAdmin` alt sınıfını alır. İkinci bir bağımsız değişken belirtmezseniz (`Publisher` ve `Books` için olduğu gibi), Django bu model için varsayılan yönetici seçeneklerini kullanacaktır.

Yaptığınız çimdikle(Böyle çevirdi komik oldu bende ellemedim), yazar değişim listesi sayfasını yeniden yükleyin ve artık üç sütun - ad, soyad ve e-posta adresi - görüntülediğini göreceksiniz. Buna ek olarak, bu sütunların her biri sütun başlığına tıklayarak sıralanabilir. (Bkz. Şekil 5-8.)

Şekil 5-8: list_display ekledikten sonra yazar değişim listesi sayfası

Sonra, basit bir arama çubuğ ekleyelim. `Author_Admin`'e `search_fields` ekleyin:

Sayfayı tarayıcınıza yeniden yükleyin ve en üstte bir arama çubuğu görmelisiniz. (Bkz: Şekil 5-9) Yönetici değişiklik listesi sayfasında, `first_name` ve `last_name` alanlarına karşı arama yapan bir arama çubuğu eklemiştik. Bir kullanıcının bekleyebileceği gibi, bu duruma duyarsızdır ve her iki alanı da arar; bu nedenle, "bar" karakter dizesini aramak, hem Barney adlı bir yazar hem de Hobarson soyadı olan bir yazar bulur.

Şekil 5-9: search_fields eklendikten sonra yazar değişim listesi sayfası

Sonra, `Kitap` modelimizin değişim listesi sayfasına bazı tarih filtreleri ekleyelim:

