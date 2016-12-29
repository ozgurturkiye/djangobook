# Templates in Views
# Views in içinde Şablonlar

Şablon sistemini kullanmanın temellerini öğrendiniz; Şimdi bu bilgileri bir görünüm oluşturmak için kullanalım.

Önceki bölümde başlattığımız mysite.views'deki current_datetime görünümünü hatırlayın. Şuna benziyor:

```python
from django.http import HttpResponse
import datetime

def current_datetime(request):

    now = datetime.datetime.now()
    html = "<html><body>It is now %s</body></html>" % now
    return HttpResponse(html)
```

Django'nun şablon sistemini kullanmak için bu görünümü değiştirelim. İlk başta böyle bir şey yapmayı düşünebilirsiniz:

```python
from django.template import Template, Context
from django.http import HttpResponse
import datetime

def current_datetime(request):

    now = datetime.datetime.now()
    t = Template("<html><body>It is now {{ current_date }}.</body></html>")
    html = t.render(Context({'current_date': now}))
    return HttpResponse(html)
```

Tabii, şablon sistemini kullanıyor ancak bu bölümün giriş bölümünde belirttiğimiz sorunları çözmedi. Şablon hala Python kodunda gömülü olduğundan, veri ve sunumun gerçek ayrımına ulaşılamıyor. Şablonu, bu görünümün yükleneceği ayrı bir dosyaya koyarak bunu düzeltelim.

```python
from django.template import Template, Context
from django.http import HttpResponse
import datetime

def current_datetime(request):

    now = datetime.datetime.now()
    # Simple way of using templates from the filesystem.
    # This is BAD because it doesn't account for missing files!
    fp = open('/home/djangouser/templates/mytemplate.html')
    t = Template(fp.read())
    fp.close()

    html = t.render(Context({'current_date': now}))
    return HttpResponse(html)
```

Bununla birlikte, bu yaklaşım, bu nedenlerden dolayı verimli değildir:

* Kayıp bir dosyayı ele almaz. Mytemplate.html dosyası yoksa veya okunamıyorsa, open () çağrısı bir IOError istisnasını ortaya çıkaracaktır.
* Şablon yerini sabit kodlar. Bu tekniği her görüntüleme işlevi için kullanırsanız, şablon konumlarını çoğalttınız demektir ve bundan bahsetmiyorum bile, bir sürü yazım gerektiriyor!
* Çok sıkıcı bir genelge kodu içerir. Her şablon yüklediğinizde open (), fp.read () ve fp.close () işlevlerine çağrı yazmaktan daha iyi işleriniz var.

Bu sorunları çözmek için, şablon yükleme ve şablon dizinleri kullanacağız.

## Template Loading
## Şablon yükleme

Django hem şablonunuzda yükleme çağrıları hem de şablonlarınızdaki fazlalıkları kaldırmak amacıyla dosya sisteminden şablonlar yüklemek için kullanışlı ve güçlü bir API sunar. Bu şablon yükleme API'sını kullanabilmek için önce şablonlarınızı şablonlarınızı depoladığınız çerçeveye bildirmeniz gerekecektir. Bunu yapmanın yeri, ayarlar dosyanızda - ROOT_URLCONF ayarını tanıttığımda geçen bölümde bahsettiğim settings.py dosyası. Takip ediyorsanız settings.py dosyasını açın ve TEMPLATES ayarını bulun. Bu, yapılandırmalar listesi olup, her biri için bir tane:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            # ... some options here ...
        },
    },
]

```

BACKEND, Django'nun şablon arka uç API'sini uygulayan bir şablon motor sınıfının noktalı bir Python yoludur. Dahili arka uçlar django.template.backends.django.DjangoTemplates ve
Django.template.backends.jinja2.Jinja2.

Çoğu motor, şablonları dosyalardan yüklediğinden, her bir motorun üst düzey yapılandırması üç ortak ayar içerir:

* DIRS, şablon kaynak dosyalarını arama sırasında aramak zorunda olduğu dizinlerin bir listesini tanımlar.
* APP_DIRS, motorun yüklü uygulamalar içindeki şablonları aramasını isteyip istemediğini bildirir. APPS_DIRS, True olarak ayarlandığında, kurallara göre DjangoTemplates, INSTALLED_APPS'in her birinde bir "şablon" alt dizini arar. Bu, Şablon motorunun DIRS boş olsa bile uygulama şablonlarını bulmasını sağlar.
* OPTIONS, arka uça özgü ayarları içerir.

Nadiren de olsa, aynı arka planın çeşitli örneklerini farklı seçeneklerle yapılandırmak mümkündür. Bu durumda, her motor için benzersiz bir NAME tanımlamanız gerekir.

## Template Directories
## Şablon Dizinleri

`DIRS`, varsayılan olarak boş bir listedir. Django'nun şablon yükleme mekanizmasını şablonları nerede bulacağını söylemek için, şablonlarınızı depolamak istediğiniz dizini seçin ve bunu DIRS'a ekleyin:

```python
'DIRS': [
           '/home/html/example.com',
           '/home/html/default',
       ],
```

Dikkat edilmesi gereken birkaç nokta var:

* Hiçbir uygulaması olmayan çok basit bir program oluşturmadığınız sürece, DIRS'i boş bırakmak daha iyi. Varsayılan ayarlar dosyası APP_DIRS öğesini True olarak yapılandırır, böylece Django uygulamanızda bir "şablonlar" alt dizinine sahip olmanız daha iyi olur.
* Örneğin, proje kökünde bir dizi ana şablon almak istiyorsanız. mysite/templates, i DIRS'ta ayarlamanız gerekir:

```python
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```

* Şablonlarınız dizininizin 'templates' olarak çağırılmasına gerek yoktur, bu arada - Django kullandığınız isimler üzerinde herhangi bir kısıtlama koymaz - ancak konvansiyona sadık kalırsanız proje yapınızı anlamanız çok daha kolaylaşır. Varsayılan değerle gitmek istemiyorum veya herhangi bir nedenden ötürü o dizinde bulunan dizin ve şablonlar Web sunucunuzun çalıştığı kullanıcı hesabı tarafından okunabildiği sürece istediğiniz herhangi bir dizini belirtebilirsiniz.
* Windows'daysanız, sürücü harfini ekleyin ve aşağıdaki gibi ters eğik çizgiler yerine Unix biçimli eğik çizgi kullanın:

```python
  'DIRS':
  [
  'C:/www/django/templates',
  ]
```

Django uygulaması henüz oluşturulmadığından, aşağıdaki örnek için beklendiği gibi çalışması için DIRS'yi `[os.path.join (BASE_DIR, 'templates')]` olarak ayarlamanız gerekecektir. DIRS ile bir sonraki adım, şablon yollarını sabit kodlamadan ziyade Django'nun şablon yükleme işlevini kullanmak için görünüm kodunu değiştirmektir. `current_datetime` görünümüne geri dönelim, şu şekilde değiştirelim:

```python
from django.template.loader import get_template
from django.template import Context
from django.http import HttpResponse
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    t = get_template('current_datetime.html')
    html = t.render(Context({'current_date': now}))
    return HttpResponse(html)
```

Bu örnekte şablonu dosya sisteminden manuel olarak yüklemek yerine `django.template.loader.get_template()` işlevini kullanıyoruz. `get_template()` işlevi, bağımsız değişkeni olarak bir şablon adı alır, şablonun dosya sisteminde nerede yaşadığını belirler, bu dosyayı açar ve derlenmiş bir `Template` nesnesi döndürür. Bu örnekteki şablonumuz `current_datetime.html` ancak bu `.html` uzantısıyla ilgili özel bir şey yok. Şablonlarınızı uygulamanız için mantıksal olan her türlü uzantıyı verebilir veya uzantıları tamamen bırakabilirsiniz.

Şablonun dosya sisteminizdeki yerini belirlemek için `get_template()` sırayla şuralara bakacaktır:

* `APP_DIRS` `True` olarak ayarlanırsa ve DTL'yi kullandığınızı varsayıyorsa, geçerli uygulamada bir "templates" dizini arar.
* Geçerli uygulamanızda şablonunuzu bulamazsa, `get_template()`, DIRS şablon dizinleri ile get_template() öğesine gönderilen şablon adını birleştirir ve şablon bulana kadar sırayla adımlar atar. Örneğin, DIRS dosyanızın ilk girişi `'/home/django/ mysite/templates'` olarak ayarlanırsa, yukarıdaki `get_template()` çağrısı `/home/django/mysite/templates/current_datetime.html` şablonunu arar.
* `get_template()` belirtilen ada sahip şablonu bulamazsa, bir `TemplateDoesNotExist` istisnası fırlatır.

Bir şablon istisnasının neye benzediğini görmek için, Django geliştirme sunucusunu tekrar çalıştırın ve Django projenizin dizininde `python manage.py runserver` çalıştırın. Ardından, tarayıcınızı `current_datetime` görünümünü etkinleştiren sayfaya yönlendirin (ör. Http://127.0.0.1:8000/time/). DEBUG ayarınızın `True` olarak ayarlandığını ve henüz bir `current_datetime.html` şablonunu oluşturmadığınızı varsayarsak, `TemplateDoesNotExist` hatasını vurgulayan bir Django hata sayfası görürsünüz (Şekil 3-1).

Bu hata sayfası, hata ayıklama bilgilerinin ek bir parçası olan Bölüm 2'de anlattığım hata sayfasına benzemektedir: "Şablon yükleyici postmortem" bölümü. Bu bölüm, Django'nun hangi şablonlarını yüklemeye çalıştığını ve her denemenin başarısız olmasının nedenini (ör. "Dosya mevcut değil") anlatır. Şablon yükleme hatalarını ayıklamaya çalışırken bu bilgi çok değerlidir. Akışı sürdürmek için şu şablon kodunu kullanarak `current_datetime.html` dosyasını oluşturun:

```python
It is now {{ current_date }}.
```

Bu dosyayı `mysite/templates` kaydedin (daha önce yapmadıysanız "templates" dizini oluşturun). Sayfayı Web tarayıcınızda yenileyin ve tam olarak oluşturulmuş sayfayı görmeniz gerekir.

## render()
