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


