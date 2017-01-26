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
from .models import Publisher, Author, Book #Bu bölümde çok sorun yaşadım .models olması gerekiyormuş

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

```python
from django.contrib import admin
from .models import Publisher, Author, Book

class AuthorAdmin(admin.ModelAdmin):
    list_display = ('first_name', 'last_name', 'email')
    search_fields = ('first_name', 'last_name')

class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'publisher', 'publication_date')
    list_filter = ('publication_date',)

admin.site.register(Publisher)
admin.site.register(Author, AuthorAdmin)
admin.site.register(Book, BookAdmin)
```

Burada, farklı seçenekler grubuyla uğraştığımız için, `BookAdmin` adlı ayrı bir `ModelAdmin` sınıfı oluşturduk. İlk olarak, bir değişim listesinin biraz daha hoş görünmesini sağlamak için bir `liste_görüntüsü` tanımladık. Ardından, change list sayfasının sağ tarafında filtreler oluşturmak için kullanılacak bir alanlar kümesine ayarlanmış olan `list_filter`i kullandık. Tarih alanlarında Django, listeyi "Bugün", "Son 7 gün", "Bu ay" ve "Bu yıl" şeklinde filtrelemek için kısayollar sağlar; bu kısayollar Django'nun geliştiricilerinin bulduğu kısayollar tarihe göre filtreleme için genel vakalara uygundur. Şekil 5-10, neye benzediğini göstermektedir.

Şekil 5-10: Liste_filterinden sonra defter değiştirme listesi sayfası

`list_filter`, yalnızca `DateField` değil, başka türdeki alanlarda da çalışır. (Örneğin, `BooleanField` ve `ForeignKey` alanları ile deneyin.) Filtreler, seçilecek en az 2 değer olduğu sürece görünür. Tarih filtreleri sunmanın başka bir yolu, aşağıdaki gibi `date_hierarchy` yönetici seçeneğini kullanmaktır:

```python
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'publisher','publication_date')
    list_filter = ('publication_date',)
    date_hierarchy = 'publication_date'
```

Bunu yaptığınızda, değişim listesi sayfası, Şekil 5-11'de gösterildiği gibi, listenin en üstünde bir tarih bulma gezinme çubuğu alır. Kullanılabilir yılların bir listesinden başlar, sonra aylar ve bireysel günlere kadar iner.

Şekil 5-11: `date_hierarchy`den sonra kitap değiştirme listesi sayfası

Hiyerarşiyi yapmak için yalnızca bir tarih alanı kullanılacağı için `date_hierarchy`'nin bir "tuple" değil bir "karakter dizisi" kullandığını unutmayın. Son olarak, değişiklik listesi sayfasındaki kitapların her zaman yayın tarihlerine göre inerek düzenlenmesini sağlamak için varsayılan siparişi değiştirelim. Varsayılan olarak, değişim listesi, Meta sınıfında (4. Bölüm'de ele almış olduğumuz) modelin siparişine göre nesneleri emir verir; ancak bu sıralama değerini belirtmediyseniz, sipariş tanımsızdır.

```python
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'publisher','publication_date')
    list_filter = ('publication_date',)
    date_hierarchy = 'publication_date'
    ordering = ('-publication_date',)
```

Bu yönetici `sıralama` seçeneği, tam olarak listede ilk alan adını kullanması hariç, modellerin sınıfı Meta'daki siparişlerle aynı şekilde çalışır. Azalan sıralama düzenini kullanmak için bir liste veya alan adlarının birçoğunu geçin ve bir alana eksi işareti ekleyin. Bunu hareket halinde görmek için kitap değiştirme listesini tekrar yükleyin. "Yayın tarihi" başlığının artık kayıtların hangi yönde sıralanmış olduğunu gösteren küçük bir ok içerdiğini unutmayın. (Bakınız Şekil 5-12.)

Şekil 5-12: Siparişten sonra kitap değiştirme listesi sayfası

Ana değişiklik listesi seçeneklerini buradan aldık. Bu seçenekleri kullanarak, yalnızca birkaç satırlık bir kodla üretime hazır çok güçlü bir veri düzenleme arabirimi oluşturabilirsiniz.

## Customizing Edit Forms
## Formları Düzenlemeyi Özelleştirme

