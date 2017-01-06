# Django Models
# Django Modelleri

Bölüm 2'de, dinamik Web sitelerini Django ile kurmanın temellerini ele aldık:Bunlar görünümler ve URLconf oluşturmaydı. Açıkladığımız gibi, bazı rasgele mantık(arbitrary logic) yapmaktan ve ardından bir yanıt döndürmekten bir görünüm sorumludur. Örneklerden birinde, keyfi mantığım mevcut tarih ve saati hesaplamaktı.

Modern Web uygulamalarında, keyfi mantık genellikle bir veritabanı ile etkileşimi gerektirir. Sahnenin arkasında, veritabanı odaklı bir Web sitesi bir veritabanı sunucusuna bağlanır, bazı verileri dışarı alır ve bir Web sayfasında görüntüler. Site, site ziyaretçilerinin veritabanını kendi başlarına doldurmalarının yollarını da sağlayabilir.

Birçok karmaşık Web sitesi bu ikisinin bir kombinasyonunu sağlar. Örneğin, Amazon.com, bir veritabanı odaklı siteye mükemmel bir örnektir. Her bir ürün sayfası temel olarak HTML biçiminde biçimlendirilmiş Amazon ürün veritabanına bir sorgudur ve bir müşteri incelemesi yayınladığınızda incelemelerin veritabanına eklenir.

Django, veritabanı odaklı Web siteleri yapmak için çok uygundur, çünkü Python kullanarak veritabanı sorgularını gerçekleştirmek için kolay ama güçlü araçlar ile birlikte gelir. Bu bölüm bu işlevi açıklıyor: Django'nun veritabanı katmanı.

* Django'nun veritabanı katmanını kullanabilmek için temel ilişkisel veritabanı teorisini ve SQL'i bilmek kesinlikle gerekli değildir, ancak şiddetle tavsiye edilir. Bu kavramlara bir giriş, bu kitabın kapsamı dışındadır, ancak bir veritabanı yeni üyeniz olsanız bile okumaya devam edin. Muhtemelen birbirine uyan içeriği ve temel kavramları kavrayacaksınız.

## The “Dumb” Way to Do Database Queries in Views
## Views'daki Veritabanı Sorgularını Yapacak "Saçma" Yolu (Dumb için döküm çevirisi de mevcut fakat saçma güzel oldu)

Kısım 2'de, bir görünümü (doğrudan metni görünümü içinde sabit kodlayarak) üretmek için "aptal" bir yol ayrıntılı olarak, bir görünümde bir veritabanından veri almak için "aptal" bir yolu var. Çok basit: bir SQL sorgusu yürütmek ve sonuçlarla ilgili bir şeyler yapmak için mevcut Python kitaplığını kullanın. Bu örnek görünümde, bir MySQL veritabanına bağlanmak, bazı kayıtları almak ve bunları bir Web sayfası olarak görüntülemek üzere bir şablona beslemek için `MySQLdb` kitaplığını kullanıyoruz:

```python
from django.shortcuts import render
import MySQLdb

def book_list(request):
    db = MySQLdb.connect(user='me', db='mydb',  passwd='secret', host='localhost')
    cursor = db.cursor()
    cursor.execute('SELECT name FROM books ORDER BY name')
    names = [row[0] for row in cursor.fetchall()]
    db.close()
    return render(request, 'book_list.html', {'names': names})
```

Bu yaklaşım çalışır, ancak bazı sorunlar hemen size anlatılmalıdır:

1. Veritabanı bağlantı parametrelerini kodlamakta zorlanıyoruz. İdeal olarak, bu parametreler Django yapılandırmasında saklanır.
2. Eşyalar kodunun adil bir kısmını yazmak zorundayız: bir bağlantı oluşturmak, bir imleç oluşturmak, bir deyimi yürütmek ve bağlantıyı kapatmak. İdeal olarak, tek yapmamız gereken sonuçlarımızı belirtmektir.
3. Bizi MySQL ile bağdaştırıyoruz. Yolda, MySQL'den PostgreSQL'e geçersek, büyük olasılıkla kodumuzun büyük bir bölümünü yeniden yazmak zorunda kalacağız. İdeal olarak, kullandığımız veritabanı sunucusu soyutlanır, böylece bir veritabanı sunucusu değişikliği tek bir yerde yapılabilir. (Bu özellik, mümkün olduğunca çok insan tarafından kullanılmak istenen bir açık kaynak Django uygulaması oluşturuyorsanız özellikle önemlidir.)

Tahmin edebileceğiniz gibi, Django'nun veritabanı katmanı bu sorunları çözmektedir.

## Configuring the Database
## Veritabanını Yapılandırma

Bütün bu felsefeyi göz önünde bulundurarak, Django'nun veritabanı katmanını keşfetmeye başlayalım. İlk olarak, uygulamayı oluştururken settings.py'ye eklenen başlangıç yapılandırmasını inceleyelim:

```python
# Database
#...
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

Varsayılan kurulum oldukça basittir. İşte her ayarın özetidir.

1. `ENGINE`(MOTOR), hangi veritabanı motorunun kullanılacağını Django'ya bildirir. Bu kitaptaki örneklerde SQLite kullanırken bunu varsayılan `django.db.backends.sqlite3` olarak bırakacağız.
2. `NAME` veritabanınızın adını Django'ya bildirir. Örneğin: `'NAME': 'mydb'`,

SQLite kullandığımızdan beri, startproject veritabanı dosyası için tam dosya sistemi yolu oluşturdu.

Varsayılan kurulum budur - bu kitabın kodunu çalıştırmak için herhangi bir şey değiştirmenize gerek yoktur, bunu yalnızca Django'daki veritabanlarını yapılandırmanın ne kadar basit olduğunu bir fikir vermek için ekledim. Django tarafından desteklenen çeşitli veritabanlarının nasıl kurulacağı hakkında ayrıntılı bilgi için bkz. Bölüm 21.

## Your First App
## İlk Uygulamanız

Artık bağlantının çalıştığını doğruladıktan sonra, tek bir Python paketinde birlikte yaşayan ve tam bir Django uygulamasını temsil eden modeller ve görünümler de dahil olmak üzere bir Django kodu demeti olan bir Django uygulaması oluşturmanın zamanı geldi. Buradaki terminolojiyi açıklamak değerlidir, çünkü bu yeni başlayanlara bir gezi niteliğindedir. Bölüm 1'de zaten bir proje yarattık, bu nedenle bir proje ile bir uygulama arasındaki fark nedir? Konfigürasyonun kodla karşılaştırılması farkıdır:

1. Bir proje, belirli bir dizi Django uygulaması ve bu uygulamaların yapılandırması örneğidir. Teknik olarak, bir projenin tek şartı, veritabanı bağlantı bilgilerini, yüklü uygulamalar listesini ve DIRS'ı tanımlayan bir ayar dosyası sağlamasıdır.
2. Bir uygulama, tek bir Python paketinde birlikte yaşayan taşınabilir bir dizi Django işlevselliğini, genellikle modelleri ve görünümleri içerir. Örneğin, Django, otomatik yönetici arabirimi gibi bir dizi uygulama ile birlikte gelir. Bu uygulamalar hakkında not etmeniz gereken en önemli nokta taşınabilir oldukları ve birden çok projede tekrar kullanılabilir olduklarıdır.

Django kodunuzu bu şemaya nasıl uyduğunuz konusunda çok az değişmez ve sıkı kurallar vardır. Basit bir Web sitesi oluşturuyorsanız, yalnızca tek bir uygulama kullanabilirsiniz. Bir e-ticaret sistemi ve mesaj panosu gibi birkaç ilgisiz parçayla karmaşık bir Web sitesi oluşturuyorsanız, muhtemelen bunları ayrı uygulamalara bölmek istersiniz, böylece ileride bunları tek tek tekrar kullanabilmenizi sağlarsınız. .

Gerçekten de, bu kitapta şimdiye kadar oluşturduğumuz örnek görünüm işlevleri ile kanıtlandığı üzere, mutlaka uygulamalar oluşturmanız gerekmez. Bu gibi durumlarda, yalnızca `view.py` adlı bir dosya oluşturduk, onu view işlevleriyle doldurduk ve URLconf dosyamızı bu işlevlere yönlendirdik. "Uygulama" yoktu.

Bununla birlikte, uygulama kuralıyla ilgili bir gereklilik var: Django'nun veritabanı katmanını (modelleri) kullanıyorsanız bir Django uygulaması oluşturmanız gerekir. Modeller, uygulamalar içinde yaşamalıdır. Böylece, modellerimizi yazmaya başlamak için yeni bir uygulama oluşturmamız gerekecek.

`mysite` proje dizini içinde (bu, `manage.py` dosyanızın bulunduğu dizinde, `mysite` uygulama dizini değil), bir kitap uygulaması oluşturmak için bu komutu yazın:

```python
python manage.py startapp books
```

Bu komut herhangi bir çıktı üretmez, ancak `mysite` dizini içinde bir `books` dizini yaratır. Bu dizinin içeriğine bakalım:

```python
books/
    /migrations
    __init__.py
    admin.py
    models.py
    tests.py
    views.py 
```

Bu dosyalar, bu uygulama için modeller ve görünümler içerecektir. En sevdiğiniz metin düzenleyicisinde `models.py` ve `views.py`'ye göz atın. Yorumlar ve `models.py` dosyalarındaki bir içe aktarma dışında her iki dosya da boştur. Bu, Django uygulamanız için boş karatahtadır.

## Defining Django Models in Python
## Python'da Django Modelleri Tanımlama

Birinci bölümde daha önce de değindiğimiz gibi "MTV" de "M", "Model" anlamına gelir. Bir Django modeli, veritabanındaki verilerin Python kodu olarak gösterildiği bir tanımdır. Veri düzeniniz - SQL CREATE TABLE ifadelerinizin karşılığıdır - bunun yerine SQL Python'tadır ve yalnızca veritabanı sütun tanımlamalarını içermektedir.

Django, sahne arkasında SQL kodunu çalıştırmak için bir model kullanır ve veritabanı tablolarındaki satırları temsil eden uygun Python veri yapılarını döndürür. Django ayrıca, SQL'in mutlaka idare edemediği üst düzey kavramları temsil etmek için modeller kullanır.

Veritabanlarını biliyorsanız, derhal düşünceniz "Veri modellerini Python'da SQL yerine tanımlamak gereksiz değil mi?" Olabilir. Django, çeşitli nedenlerle bu şekilde çalışır:

1. İçgözlem genel gider gerektirir ve eksiktir. Uygun veri erişim API'leri sağlamak için, Django'nun bir şekilde veritabanı düzenini bilmesi gerekiyor ve bunu başarmanın iki yolu var. İlk yol, açıkça Python'daki verileri açıklamak ve ikincisi, veri modellerini belirlemek için çalışma zamanında veritabanını incelemek olacaktır. Tablolarınızdaki meta veriler tek bir yerde yaşadığı için bu ikinci yöntem daha temiz görünüyor. Birkaç sorun getiriyor. Birincisi, çalışma zamanında bir veritabanına introspection açıkça yük gerektirir. Eğer çerçeve, bir istek işlediğinde her seferinde veritabanını incelemek zorunda kalırsa ya da sadece Web sunucusu başlatıldığında bu, kabul edilemez düzeyde bir yük getirirdi. (Bazıları, bu yük seviyesinin kabul edilebilir olmasına rağmen, Django'nun geliştiricileri Mümkün olduğunca fazla çerçeve yükü.) İkincisi, bazı veritabanları, özellikle de MySQL eski sürümleri, doğru ve tam içgözlem için yeterli meta verileri depolamıyor.

2. Python yazmak eğlencelidir ve her şeyi Python'da tutmak beyninizin "bağlamsal geçiş" yapması gereken süreyi sınırlar. Mümkün olduğunca uzun süre tek bir programlama ortamında / zihniyetinde kalırsanız üretkenliğe yardımcı olur. SQL, daha sonra Python yazmak zorunda kaldıktan sonra SQL yine yıkıcı olur.

3. Veritabanınızda değil kod olarak saklanan veri modellerine sahip olmak, modellerinizi sürüm denetimi altında tutmayı kolaylaştırır. Bu şekilde, veri düzenlerinizdeki değişiklikleri kolayca takip edebilirsiniz.

4. SQL, veri mizanpajı hakkında yalnızca belirli bir metadata düzeyi sağlar. Örneğin, çoğu veritabanı sistemi, e-posta adreslerini veya URL'leri temsil etmek için özel bir veri türü sağlamaz. Django modelleri yapar. Üst düzey veri türlerinin avantajı, daha yüksek üretkenlik ve daha fazla yeniden kullanılabilir koddur.

5. SQL, veritabanı platformlarında tutarsız. Örneğin bir Web uygulamasını dağıtıyorsanız, veri düzeninizi MySQL, PostgreSQL ve SQLite için ayrı CREATE TABLE deyimlerinin setinden daha açıklayan bir Python modülü dağıtmak çok daha pratiktir.

Bununla birlikte, bu yaklaşımın bir dezavantajı, Python kodunun gerçekte veritabanındaki ile senkronize edilmesinin mümkün olmasıdır. Bir Django modelinde değişiklik yaparsanız, veritabanınızı modelle tutarlı tutmak için veritabanınızda aynı değişiklikleri yapmanız gerekir. Taşıma işlemlerini bu bölümün ilerleyen bölümlerinde tartışırken bu sorunu nasıl halledeceğinizi size göstereceğim.

Son olarak, Django'nun mevcut bir veritabanını içgözlemleyerek modeller oluşturabilen bir yardımcı program içerdiğini unutmamalısınız. Bu, eski verilerle hızla kalkış ve yayın yapmak için kullanışlıdır. Bunu 21. Bölüm'de ele alacağız.

## Your First Model
## İlk Modeliniz

Bu bölümde ve devam eden bölümde devam eden bir örnek olarak, temel bir kitap / yazar / yayıncı veri düzenine odaklanacağım. Bunu örnek olarak kullanıyorum, çünkü kitaplar, yazarlar ve yayıncılar arasındaki kavramsal ilişkiler çok iyi biliniyor ve bu, giriş niteliğindeki SQL ders kitaplarında kullanılan yaygın bir veri düzenidir. Yazarlar tarafından yazılmış ve bir yayıncı tarafından üretilen bir kitap da okunuyorsunuz!

Aşağıdaki kavramları, alanları ve ilişkileri varsayacağım:

1. Bir yazarın adı, soyadı ve e-posta adresi vardır.
2. Bir yayıncının adı, sokak adresi, bir şehir, bir eyalet / il, bir ülke ve bir Web sitesi vardır.
3. Bir kitabın bir başlığı ve yayın tarihi vardır. Ayrıca bir veya daha fazla yazar (yazarlar ile çoktan çok ilişkisi vardır) ve tek bir yayıncı (yayıncılara biricik yabancı anahtar - bir-çok ilişki) vardır.

Django ile bu veritabanı düzenini kullanmanın ilk adımı, onu Python kodu olarak ifade etmektir. Startapp komutu tarafından oluşturulan models.py dosyasında aşağıdakileri girin:

```python
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

class Author(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=40)
    email = models.EmailField()

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()
```

Temelleri kapsayacak şekilde bu kodu hızlı bir şekilde inceleyelim. Dikkat etmeniz gereken ilk şey, her modelin django.db.models.Model alt sınıfı olan bir Python sınıfı tarafından temsil edilmesidir. Ana sınıf olan Model,
Bu nesneleri bir veritabanı ile etkileşime girme yeteneğine sahip yapmak için gerekli olan tüm makineleri içerir - ve bu da yalnızca kendi alanlarını tanımlamak için sorumlu olan modellerimizi güzel ve kompakt bir sözdiziminde bırakır.

İster inanın ister inanmayın, Django ile temel veri erişimi için yazmak istediğimiz kod budur. Her model genellikle tek bir veritabanı tablosuna karşılık gelir ve bir modelin her özniteliği genellikle bu veritabanı tablosundaki bir sütuna karşılık gelir. Öznitelik adı, sütunun adına karşılık gelir ve alanın türü (ör. `CharField`), veritabanı sütun türüne (ör. `Varchar`) karşılık gelir. Örneğin, `publisher` yayıncı modeli aşağıdaki tabloda (PostgreSQL `CREATE TABLE` sözdizimini varsayarak) eşdeğerdir:

```python
CREATE TABLE "books_publisher" (
    "id" serial NOT NULL PRIMARY KEY,
    "name" varchar(30) NOT NULL,
    "address" varchar(50) NOT NULL,
    "city" varchar(60) NOT NULL,
    "state_province" varchar(30) NOT NULL,
    "country" varchar(50) NOT NULL,
    "website" varchar(200) NOT NULL
);
```

Gerçekten de, Django CREATE TABLE deyimini otomatik olarak oluşturabilir, çünkü bir süre sonra size göstereceğiz. Veritabanı tablosu sınıfı başına bir sınıfa ayrılan kuralın istisnası çok-çoklu ilişkilerin olmasıdır. Örnek modellerimizde, Book'un yazarlar adı verilen ManyToManyField vardır. Bu, bir kitabın bir veya daha fazla yazar olduğunu ancak Kitap veritabanı tablosunda bir yazar sütununun bulunmadığını belirtir. Bunun yerine, Django, kitapların yazarlara eşleştirilmesini sağlayan bir çok-çoklu "birleştirme tablosu" - ek bir tablo oluşturur.

Alan türleri ve model sözdizimi seçeneklerinin tam listesi için Ek B'ye bakın. Son olarak, bu modellerin herhangi birinde açıkça birincil anahtarı tanımadığımıza dikkat edin. Aksi belirtilmediği sürece, Django otomatik olarak her modele id olarak adlandırılan bir otomatik artan tamsayı birincil anahtar alanı verir. Her bir Django modelinin tek sütunlu birincil anahtarı olması gerekir.

## Installing the Model
## Modelin Kurulumu

We’ve written the code; now let’s create the tables in our database. In order to do that, the first step is to activate these models in our Django project. We do that by adding the books app to the list of “installed apps” in the settings file. Edit the `settings.py` file again, and look for the `INSTALLED_APPS` setting. `INSTALLED_APPS` tells Django which apps are activated for a given project. By default, it looks something like this:

```python
INSTALLED_APPS = (
'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',
)
```

"books" uygulamamızı kaydettirmek için `INSTALLED_APPS` ürününe `"books"` ekleyin, böylece ayar şu şekilde görünür ("books" üzerinde çalışmakta olduğumuz "books" uygulamasıdır):

```python
INSTALLED_APPS = (
'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',
'books',
)
```

`INSTALLED_APPS` uygulamasındaki her uygulama tam Python yolu ile temsil edilir - başka bir deyişle, noktalarla ayrılmış paketlerin yolu, uygulama paketine götürür. Django uygulaması ayar dosyasında etkinleştirildiğine göre, veritabanımızda veritabanı tablolarını oluşturabiliriz. Önce, bu komutu çalıştırarak modelleri doğrulayalım:

```python
python manage.py check
```

Check komutu Django sistem kontrol çerçevesini çalıştırır - Django projelerini doğrulamak için statik kontroller dizisi. Her şey yolunda giderse, Sistem denetimi hiçbir sorun tanımlamadı mesajı görürsünüz (sessiz durumdadır). Eğer yapmazsan, model kodunu doğru yazdığınızdan emin olun. Hata çıktısı, kodda olan yanlış hakkında yararlı bilgi vermelidir. Modellerinizle ilgili herhangi bir sorun yaşandığınızı düşünüyorsanız, python manage.py onayını çalıştırın. Tüm yaygın model problemlerini yakalamak eğilimindedir.

Modelleriniz geçerliyse, modellerinizde bazı değişiklikler yaptığınızı Django'ya bildirmek için aşağıdaki komutu çalıştırın (bu durumda, yeni bir tane yaptınız):

```python
python manage.py makemigrations books 
```

Aşağıdakine benzer bir şey görmelisiniz:

```python
Migrations for 'books':
  0001_initial.py:
    - Create model Author
    - Create model Book
    - Create model Publisher
    - Add field publisher to book 
```

Geçişler, Django'nun modellerinizde (dolayısıyla veritabanı şemanızdaki) değişiklikleri saklaması biçimindedir - sadece disk üzerindeki dosyalardır. Bu durumda, kitapların uygulamanın 'taşıma' klasöründe `0001_initial.py` dosya adları bulacaksınız. Geçiş komutu, en son geçiş dosyanızı alacak ve veritabanı şemasını otomatik olarak güncelleyecektir, ancak önce hangi göçün çalışacağını bir SQL görelim. Sqlmigrate komutu, geçiş adlarını alır ve SQL'lerini döndürür:

```python
python manage.py sqlmigrate books 0001
```

Aşağıdakine benzer bir şey görmelisiniz (okunabilirlik için yeniden biçimlendirildi):

```python
BEGIN;

CREATE TABLE "books_author" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(40) NOT NULL,
    "email" varchar(254) NOT NULL
);
CREATE TABLE "books_book" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "title" varchar(100) NOT NULL,
    "publication_date" date NOT NULL
);
CREATE TABLE "books_book_authors" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "book_id" integer NOT NULL REFERENCES "books_book" ("id"),
    "author_id" integer NOT NULL REFERENCES "books_author" ("id"),
    UNIQUE ("book_id", "author_id")
);
CREATE TABLE "books_publisher" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "name" varchar(30) NOT NULL,
    "address" varchar(50) NOT NULL,
    "city" varchar(60) NOT NULL,
    "state_province" varchar(30) NOT NULL,
    "country" varchar(50) NOT NULL,
    "website" varchar(200) NOT NULL
);
CREATE TABLE "books_book__new" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "title" varchar(100) NOT NULL,
    "publication_date" date NOT NULL,
    "publisher_id" integer NOT NULL REFERENCES
    "books_publisher" ("id")
);

INSERT INTO "books_book__new" ("id", "publisher_id", "title",
"publication_date") SELECT "id", NULL, "title", "publication_date" FROM
"books_book";

DROP TABLE "books_book";

ALTER TABLE "books_book__new" RENAME TO "books_book";

CREATE INDEX "books_book_2604cbea" ON "books_book" ("publisher_id");

COMMIT;
```

Aşağıdakilere dikkat edin:

1. Tablo adları, uygulamanın adını (kitapları) ve modelin küçük adını (yayıncı, kitap ve yazar) birleştirerek otomatik olarak oluşturulur. Ek B'de detaylandırıldığı gibi, bu davranışı geçersiz kılabilirsiniz.

2. Daha önce de belirttiğimiz gibi, Django otomatik olarak her tablo için birincil anahtar ekler - kimlik alanları. Bunu geçersiz kılabilirsiniz. Sözleşmeye göre, Django, yabancı anahtar alan adına "_id" ekler. Tahmin edebileceğiniz gibi, bu davranışı da geçersiz kılabilirsiniz.

3. Yabancı anahtar ilişkisi, bir `REFERENCES` deyimiyle açıkça belirtilir.

4. Bu CREATE TABLE ifadeleri, kullandığınız veritabanına uyar. Bu nedenle, `auto_increment` (MySQL), `serial` (PostgreSQL) veya `integer primary key` (SQLite) gibi veritabanına özel alan türleri otomatik olarak sizin için ele alınır. Aynı şey sütun adlarının alıntılanması için de geçerli (ör. Çift tırnak işaretleri veya tek tırnak işaretleri kullanılarak). Bu örnek çıktı PostgreSQL sözdiziminde.

5. `sqlmigrate` komutu aslında tabloları oluşturmaz veya başka bir şekilde veritabanınıza dokunmaz - soruyu sorduysanız SQL Django'nun ne çalıştıracağını görebilmeniz için çıktıyı ekranda yazdırır. İsterseniz, bu SQL'yi veritabanı istemcinize kopyalayıp yapıştırabilirsiniz, ancak Django, SQL'i veritabanına kaydetmenin daha kolay bir yolunu sunar: `migrate` Komut:

```python
python manage.py migrate 
```

Bu komutu çalıştırın ve böyle bir şey göreceksiniz:

```python
Operations to perform:
  Apply all migrations: books
Running migrations:
  Rendering model states... DONE

  # ...

  Applying books.0001_initial... OK

  # ...
```

Django, tüm ekstraların (yukarıda açıklandığı gibi) ne olduğunu merak ediyor olsaydınız, ilk kez geçiş işlemini çalıştırdığınızda Django, dahili uygulamalarda Django'nun ihtiyaç duyduğu tüm sistem tablolarını oluşturacaktır.
Geçişler, Django'nun modellerinize yaptığınız değişiklikleri (alan ekleme, bir modeli silme, vb.) Veritabanınızın şemasına yayma yoludur. Çoğunlukla otomatik olacak şekilde tasarlandıklarından bazı uyarılar var. Taşıma işlemleri hakkında daha fazla bilgi için Bölüm 21'e bakın.
