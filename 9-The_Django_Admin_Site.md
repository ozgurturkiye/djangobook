# The Django Admin Site

Çoğu modern Web sitesinde, bir yönetici arabirimi altyapının önemli bir parçasıdır. Bu, site içeriğinin eklenmesi, düzenlenmesi ve silinmesini sağlayan güvenilir site yöneticileri ile sınırlı, Web tabanlı bir arabirimdir. Bazı yaygın örnekler: blogunuza yüklemek için kullandığınız arayüz, arka plan site yöneticileri, kullanıcı tarafından oluşturulan yorumları denetlemek için kullanır; bu, müşterilerinizin sizin için oluşturduğunuz Web sitesinde bulunan basın bültenlerini güncellemek için kullandığı araçtır.

Ancak, yönetici arayüzleri ile ilgili bir sorun var: bunları oluşturmak sıkıcıdır. Web üzerinden geliştirme, herkese açık işlevleri geliştirirken çok eğlencelidir, ancak yönetici arabirimleri oluşturmak daima aynıdır. Kullanıcıların kimliğini doğrulamak, formları görüntülemek ve işlemek, girişi doğrulamak vb. Gerekiyor. Çok sıkıcı ve tekrarlayıcı.

Peki bu sıkıcı, tekrarlayan görevlere Django'nun yaklaşımı nedir? Senin için hepsini yapar.

Django ile yönetici arayüzü oluşturmak çözülmüş bir sorundur. Bu bölümde Django'nun otomatik yönetici arayüzünü inceleyeceğiz: Modellerimize nasıl uygun bir arayüz oluşturduğunu kontrol ederek,
Ve onunla yapabileceğimiz diğer yararlı şeylerden bazıları.

## Using the Django Admin Site
## Django Yönetici Site'sini Kullanma

Bölüm 1'de `django-admin startproject mysite`'yi çalıştırdığınızda, Django varsayılan admin sitesini sizin için oluşturup yapılandırdı. Yapmanız gereken şey bir yönetici kullanıcısı (süper kullanıcı) oluşturmak ve ardından yönetici sitesinde oturum açabilmek.

Bir yönetici kullanıcısı oluşturmak için aşağıdaki komutu çalıştırın:

```python
python manage.py createsuperuser 
```

İstediğiniz kullanıcı adını girin ve enter tuşuna basın.

`Username: admin`

Ardından, istediğiniz e-posta adresini girmeniz istenir:

`Email address: admin@example.com`

Son adım şifrenizi girmektir.
İki kez şifrenizi girmeniz istenecek, ikinci sefer ise ilk şifrenizi onaylamanız istenecektir.

```python
Password: **********
Password (again): *********
Superuser created successfully.
```

## Start the Development Server
## Geliştirme Sunucusunu Başlatın

Django 1.8'de, varsayılan olarak django admin sitesi etkinleştirilmiştir. Geliştirme sunucusunu başlatalım ve keşfedelim. Geliştirme sunucusunu böyle başlattığınızı önceki bölümlerden hatırlayın:

`python manage.py runserver`

Şimdi, bir Web tarayıcısı açın ve yerel etki alanınızdaki / admin / adresine gidin - ör. `http://127.0.0.1:8000/admin/`. Yönetici'nin giriş ekranını görmelisiniz.

Çeviri varsayılan olarak açık olduğundan, tarayıcı ayarlarına ve Django'nun bu dil için bir çeviri olup olmadığına bağlı olarak giriş ekranı kendi dilinizde görüntülenebilir.

## Enter the Admin Site
## Yönetici Sitesine Girin

Şimdi, önceki adımda oluşturduğunuz süper kullanıcı hesabı ile giriş yapmayı deneyin. Django admin dizin sayfasını görmelisiniz (Şekil 5-2). Kaynakça: http://djangobook.com/django-admin-site/

Düzenlenebilir içerikte iki tür görmelisiniz: gruplar ve kullanıcılar. Bunlar, Django tarafından gönderilen kimlik doğrulama çerçevesi olan `django.contrib.auth` tarafından sağlanmaktadır. Yönetici sitesi, teknik olmayan kullanıcılar tarafından kullanılmak üzere tasarlanmıştır ve bu nedenle, kendini açıklayıcı nitelikte olmalıdır. Yine de, size temel özellikleri hızlı bir şekilde gözden geçirelim.

Şekil 5-2: Django admin ana sayfası

Django yönetim sitesinde bulunan her veri türü bir değişiklik listesi ve bir düzenleme formuna sahiptir. Değişiklik listeleri, veritabanındaki mevcut tüm nesneleri gösterir ve formları düzenleme, veritabanınızdaki belirli kayıtları ekleme, değiştirme veya silmenize izin verir. Kullanıcılar için değişim listesi sayfasını yüklemek için "Kullanıcılar" satırındaki "Değiştir" bağlantısını tıklayın (Şekil 5-3).

Şekil 5-3: Kullanıcı değiştirme listesi sayfası

Bu sayfa, veritabanındaki tüm kullanıcıları görüntüler; Bunu bir `SELECT * FROM auth_user`'in hazırlanmış bir Web sürümü olarak düşünebilirsiniz; SQL sorgusu. Devam eden örneğimizi takip ediyorsanız yalnızca bir tane eklediğinizi varsayarak yalnızca bir kullanıcı görürsünüz, ancak bir kere daha kullanıcılarınız varsa, muhtemelen filtreleme, sıralama ve arama seçeneklerini kullanışlı bulacaksınız.

Filtreleme seçenekleri doğru, sıralama bir sütun başlığını tıklatarak elde edilebilir ve en üstteki arama kutusu kullanıcı adıyla arama yapmanızı sağlar. Oluşturduğunuz kullanıcının kullanıcı adını tıklayın, o kullanıcı için düzenleme formunu görürsünüz (Şekil 5-4).

Bu sayfa, kullanıcının niteliklerini (ad, soyad ve çeşitli izinler gibi) değiştirmenize izin verir. Bir kullanıcının parolasını değiştirmek için, karışık kodu düzenlemek yerine şifre alanının altındaki "şifre formunu değiştir" seçeneğini tıklamanız gerektiğini unutmayın.

Burada dikkat edilmesi gereken bir diğer husus, farklı türdeki alanların farklı widget'lere sahip olmalarıdır - örneğin, tarih /saat alanlarının takvim denetimleri, Boolean alanları onay kutuları vardır, karakter alanlarının basit metin girişi alanları vardır.

Şekil 5-4: Kullanıcı düzenleme formu

Bir kaydı, düzenleme formunun sol altındaki sil düğmesini tıklayarak silebilirsiniz. Bu, sizi bazı durumlarda bağımlı nesneleri de görüntüleyecek bir onay sayfasına götürür. (Örneğin, bir yayıncıyı silerseniz, o yayıncıyla olan tüm kitaplar da silinir!)

Yönetici giriş sayfasının ilgili sütunundaki "Ekle" yi tıklayarak bir kayıt ekleyebilirsiniz. Bu, düzenleme sayfasının doldurulması için hazır, boş bir sürümünü size verecektir.

Ayrıca, yönetici arayüzünün sizin için giriş doğrulamasını da yönettiğini fark etmiş olursunuz. Gerekli bir alanı boş bırakmayı veya bir tarih alanına geçersiz bir tarih koymayı deneyin ve bu hataları, Şekil 5-5'de gösterildiği gibi kaydetmeye çalıştığınızda göreceksiniz.

Mevcut bir nesneyi düzenlediğinizde, pencerenin sağ üst köşesinde Geçmiş bağlantısı görürsünüz. Yönetici arayüzü üzerinden yapılan her değişiklik günlüğe kaydedilir ve Geçmiş bağlantısını tıklayarak bu günlüğü inceleyebilirsiniz (bkz. Şekil 5-6).

Şekil 5-5: Hataları gösteren bir düzenleme formu

Şekil 5-6: Bir nesne geçmiş sayfası

### Yönetici Sitesi Nasıl Çalışır 

Perde arkasında, yönetici sitesi nasıl çalışır? Oldukça basittir. Django sunucu başlatıldığında yüklendiğinde, `admin.autodiscover()` işlevi çalıştırılır. Django'nun önceki sürümlerinde, bu işlevi `urls.py`'den çağırıyordunuz, ancak şimdi Django otomatik olarak çalıştırıyor. Bu işlev, `INSTALLED_APPS` ayarınızı yineler ve yüklü olan her uygulamada `admin.py` adlı bir dosyayı arar. Belirli bir uygulamada bir `admin.py` varsa, o dosyadaki kodu uygular. `admin.py` bizim kitaplar uygulamasında, `admin.site.register()` için yapılan her çağrı, verilen modeli admin ile kaydeder. Yönetici sitesi yalnızca açıkça kayıtlı olan modeller için bir düzenleme / değiştirme arayüzü görüntüler. `django.contrib.auth` uygulaması kendi `admin.py` dosyasını içermektedir. Bu nedenle, Kullanıcılar ve Gruplar otomatik olarak admin'de görünür. `Django.contrib.redirects` gibi diğer `django.contrib` uygulamaları da Web tarayıcısından indirebileceğiniz birçok üçüncü taraf Django uygulaması gibi kendilerini yönetici olarak eklerler. Django yönetici sitesi yalnızca bir Django uygulaması olan Kendi modelleri, şablonları, görünümleri ve URL kalıpları. Kendi görünümlerinizde olduğu gibi, uygulamanıza URLconf'a bağlayarak onu uygulamanıza ekleyin. Django kod tablonuzun kopyasında django / contrib / admin'de dolaşarak şablonlarını, görünümlerini ve URL kalıplarını inceleyebilirsiniz - ancak doğrudan orada hiçbir şeyi değiştirmeye cazip gelmeyin, çünkü bunu özelleştirecek çok kanca var. Django admin uygulaması etrafında dolaşmaya karar verirseniz, modellerle ilgili meta verileri okurken oldukça karmaşık şeyleri aklınızda bulundurun, dolayısıyla kodu okumak ve anlamak için büyük bir zaman alacağını göz önünde bulundurun.
