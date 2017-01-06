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


