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

Şu ana kadar bir şablonun nasıl yükleneceğini, bir Context'i nasıl doldurduğunuzu ve işlenmiş şablonun sonucuyla bir HttpResponse nesnesi döndürdüğümüzü gösterdik. Sonraki adım, sabit kodlama şablonları ve şablon yolları yerine get_template () kullanmak üzere optimize etmekti. Django şablonlarının nasıl yüklendiğini ve tarayıcınıza nasıl işlendiğini anlamanız için bu işlemi gerçekleştirdim.

Uygulamada, Django bunu yapmak için çok daha kolay bir yol sağlar. Django'nun geliştiricileri, bu yaygın bir deyim olduğu için Django'nun tüm bunları bir satırda yapabilecek bir kısaya ihtiyacı olduğunu kabul ettiler. Bu kısayol django.shortcuts modülünde yaşayan `render()` adı verilen bir işlevdir.

Çoğu zaman, şablonları yüklemek ve Context ve HttpResponse nesnelerini manuel olarak oluşturmaktan ziyade render() öğesini kullanacaksınız - işvereniniz işinizi toplam yazılı kod satırıyla değerlendirmiyorsa, yani. :) Espiriye gel :D

Render() işlevini kullanmaya devam eden current_datetime örneğini şu şekilde yazdırabilirsiniz:

```python
from django.shortcuts import render
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    return render(request, 'current_datetime.html', {'current_date': now})
```

Ne fark var! Kod değişikliklerinde adım adım ilerleyelim:

* Artık get_template, Template, Context veya HttpResponse'u içe aktarmamız gerekmiyor. Bunun yerine, django.shortcuts.render'ı içe aktarırız. `import date time` hala geçerli.
* Current_datetime işlevi içinde hala hesaplarız, ancak yükleme, içerik oluşturma, şablon oluşturma ve HttpResponse oluşturma kalıpları render() çağrısı ile halledilir. Render() bir HttpResponse nesnesi döndürdüğünden, bu değeri kolayca view'e(görünüme) döndürebiliriz.

`Render()` ilk argümanı `request`, ikincisi kullanılacak `Template` adıdır. Üçüncü argüman, verilirse, o şablon için bir `Context` yaratmada kullanılacak bir sözlük olmalıdır. Üçüncü bir argüman sağlamazsan, render() boş bir sözlük kullanır.

## Template Subdirectories
## Template Altdizinleri

Tüm şablonlarınızı tek bir dizinde saklamanız hantal olabilir. Şablon dizininizin alt dizinlerine şablonlar depolamak isteyebilirsiniz ve bu sorun değildir.

Aslında bunu yapmanızı öneririm; Bazı daha gelişmiş Django özellikleri (Bölüm 10'da ele aldığımız jenerik görünüm sistemi gibi) bu şablon düzenini varsayılan bir kural olarak beklemektedir.

Şablon dizininizin alt dizinlerine saklamak kolaydır. get_template() işlevini çağırırken, şablon adından önce alt dizin adını ve eğik çizgiyi ekleyin:
```python
t = get_template('dateapp/current_datetime.html')
```

render() get_template() etrafında küçük bir sarmalayıcı olduğundan, render() işlevinin ikinci argümanıyla aynı şeyi yapabilirsiniz:

```python
return render(request, 'dateapp/current_datetime.html', {'current_date': now})
```
Alt dizin ağacınızın derinlik sınırı yoktur. İstediğiniz kadar alt dizin kullanmaya çekinmeyin.

* Windows kullanıcıları, ters bölüden ziyade eğik çizgi kullandığınızdan emin olun. get_template(), bir Unix tarzı dosya adı atamasını varsayar. (google bu kadar çevirdi bende düzeltmeye üşendim ;))

## The include Template Tag
## include Template Etiketi'ı :)

Artık şablon yükleme mekanizmasını tamamladığımıza göre, avantajlarından yararlanan yerleşik bir şablon etiketi sunabiliriz: {% include %}.

Bu etiket, başka bir şablonun içeriğini eklemenizi sağlar. Etikete verilen argüman, eklenecek şablonun adı olmalıdır ve şablon adı, tek veya çift tırnak işaretli bir değişken veya sabit kodlanmış (hard-coded) bir dize olabilir.

Birden çok şablonda aynı koda sahipseniz, çoğaltmayı kaldırmak için bir {% includ %} kullanmayı düşünün. Bu iki örnek, `nav.html` şablonunun içeriğini içerir. Örnekler eşdeğerdir ve tek veya çift tırnak işaretlerine izin verildiğini gösterir:
```python
{% include 'nav.html' %}
{% include "nav.html" %}
```
Bu örnek `includes/nav.html` şablonunun örneğini içerir:

`{% include 'includes/nav.html' %}`

Bu örnek, adı `template_name` değişkeninde bulunan şablonun içeriğini içerir:

`{% include template_name %}`

get_template() 'de olduğu gibi, şablonun dosya adı, geçerli Django uygulamasındaki "şablonlar" dizinine (`APPS_DIR` değeri `True` ise) yolu ekleyerek veya DIRS'den şablon dizinini istenen şablon adına ekleyerek belirlenir. Dahil edilen şablonlar, bunları içeren şablonun içeriği ile değerlendirilir.

Örneğin, bu iki şablonu göz önünde bulundurun:

```python
# mypage.html

<html>
<body>
    {% include "includes/nav.html" %}
    <h1>{{ title }}</h1>
</body>
</html>

# includes/nav.html

<div id="nav">
    You are in: {{ current_section }}
</div>
```

Eğer mypage.html dosyasını `current_section` içeren bir `context` işlerse, değişken "include edilen" şablonda beklediğiniz gibi kullanılabilir olacaktır.

Bir {% include %} etiketinde, verilen ada sahip bir şablon bulunamazsa, Django iki şeyden birini yapacaktır:

* DEBUG doğru olarak ayarlanırsa, bir Django hata sayfasında `TemplateDoesNotExist` istisnasını görürsünüz.
* DEBUG Yanlış olarak ayarlanırsa, etiket sessizce başarısız olur ve etiket yerine başka bir şey görüntülenmez.

Bilgi Notu: Dahil edilen şablonlar arasında paylaşılan bir bölge yoktur - her bir "include" tamamen bağımsız bir işleme sürecidir. Bloklar dahil edilmeden önce değerlendirilirler. Diğer bir deyişle blokları içeren bir şablonun, daha önce değerlendirilen ve işlenmiş bloklar içerdiği anlamına gelir; örneğin, uzayan(extending) bir şablon tarafından geçersiz kılınabilen bloklar.

## Template Inheritance
## Template(Şablon) Mirasları

Şu ana kadar şablon örneklerimiz küçük HTML parçacıklarıdır, ancak gerçek dünyada tüm HTML sayfalarını oluşturmak için Django şablon sistemini kullanıyor olacaksınız. Bu ortak bir Web geliştirme sorununa neden olur: Bir Web sitesinde, site genişliğinde gezinme gibi ortak sayfa alanlarının çoğaltılmasını ve fazlalığını nasıl azaltabilir?

Bu sorunu çözmenin klasik bir yolu, bir Web sayfasını başka bir web sayfasına "dahil etmek" için HTML sayfalarınıza yerleştirebileceğiniz yönerge içeren sunucu tarafı içerme yönergesini kullanmaktır. Nitekim, Django, açıklanan {% include %} şablon etiketiyle bu yaklaşımı desteklemektedir.

Fakat bu sorunu Django ile çözmenin tercih edilen yolu şablon devralma adlı daha şık bir strateji kullanmaktır. Özünde, kalıp devralma, sitenizin tüm ortak bölümlerini içeren bir temel "iskelet" şablonu oluşturmanıza ve çocuk şablonlarının geçersiz kılabileceği "bloklar" tanımlamanıza izin verir. `current_datetime.html` dosyasını düzenleyerek `current_datetime` görünümümüz için daha eksiksiz bir şablon oluşturarak bunun bir örneğini görelim:

```python
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<html lang="en">

<head>
    <title>The current time</title>
</head>
<body>
    <h1>My helpful timestamp site</h1>
    <p>It is now {{ current_date }}.</p>

    <hr>
    <p>Thanks for visiting my site.</p>
</body>
</html>
```

Bu iyi görünüyor, ancak başka bir görünüme yönelik bir şablon oluşturmak istediğimizde ne olur - 2. Bölüm'ün `hours_ahead` görünümü? Güzel, geçerli, tam bir HTML şablonunu yeniden oluşturmak istersek, aşağıdakine benzer bir şey oluştururuz:

```python
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<html lang="en">

<head>
    <title>Future time</title>
</head>
<body>
    <h1>My helpful timestamp site</h1>
    <p>
        In {{ hour_offset }} hour(s), it will be {{ next_time }}.
    </p>

    <hr>
    <p>Thanks for visiting my site.</p>
</body>
</html>
```
Açıkçası, bir sürü HTML'yi tekrar kopyaladık. Bir gezinme çubuğu, birkaç stil sayfası, belki de bir JavaScript gibi daha tipik bir siteniz olsaydı, her tür şablona her türlü gereksiz HTML koymaya tekrar tekrar koymayı bir düşünün.

Sunucu tarafı bu sorunun çözümünü içerir, her iki şablonda da ortak bitleri çarpıtarak onları her bir şablonda bulunan ayrı şablon snippet'lerine kaydedin. Belki de şablonun üst bölümünü `header.html` adlı bir dosyada saklarsınız:

```python
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<html lang="en">

<head>
```

Ve belki de alt bölümü footer.html adlı bir dosyada saklarsınız:

```python
    <hr>
    <p>Thanks for visiting my site.</p>
    </body>
</html>
```

Bir include tabanlı strateji ile üstbilgi ve altbilgi almak kolaydır. Dağınık ortası budur. Bu örnekte her iki sayfa da "My helpful timestamp site" başlıklı bir başlık içeriyor; ancak bu başlık, her iki sayfadaki başlık farklı olduğu için `header.html` içine sığmıyor. Üstbilgiye h1 eklediysek, başlığı eklememiz gerekir; bu da başlığı sayfa başına özelleştirmemize izin vermez.

Django'nun şablon kalıtım sistemi(Django’s template inheritance system) bu sorunları çözer. Bunu, sunucu tarafı kapsayan "iç-çıkış" bir sürümü olarak düşünebilirsiniz. Sık kullanılan snippet'leri tanımlamak yerine, farklı snippet'leri tanımlarsınız.

İlk adım, bir ana(base) şablon tanımlamaktır - alt şablonlarının daha sonra dolduracağı sayfanızın bir iskeleti. Devam eden örnek için temel bir şablon:

```python
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<html lang="en">

<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <h1>My helpful timestamp site</h1>
    {% block content %}{% endblock %}

    {% block footer %}
    <hr>
    <p>Thanks for visiting my site.</p>
    {% endblock %}
</body>
</html>
```
