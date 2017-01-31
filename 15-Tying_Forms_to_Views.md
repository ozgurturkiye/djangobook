# Tying Forms to Views
# Formları Görünümlerle Bağlama

İletişim formumuz, kullanıcıya göstermenin bir yolu yoksa bizim için pek iyi değil. Bunu yapmak için öncelikle mysite/views güncellemeliyiz:

```python
# views.py

from django.shortcuts import render
from mysite.forms import ContactForm
from django.http import HttpResponseRedirect
from django.core.mail import send_mail

# ...

def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            cd = form.cleaned_data
            send_mail(
                cd['subject'],
                cd['message'],
                cd.get('email', 'noreply@example.com'),
                ['siteowner@example.com'],
            )
            return HttpResponseRedirect('/contact/thanks/')
    else:
        form = ContactForm()
    return render(request, 'contact_form.html', {'form':
form})
```

Sonra, iletişim formumuzu oluşturmamız gerekiyor (bunu mysite/templates kaydedin):

```python
# contact_form.html

<html>
<head>
    <title>Contact us</title>
</head>
<body>
    <h1>Contact us</h1>

    {% if form.errors %}
        <p style="color: red;">
            Please correct the error{{ form.errors|pluralize }} below.
        </p>
    {% endif %}

    <form action="" method="post">
        <table>
            {{ form.as_table }}
        </table>
        {% csrf_token %}
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

Ve son olarak, `/contact/` adresindeki iletişim formumuzu görüntülemek için `urls.py` dosyamızı değiştirmemiz gerekiyor:

```python
 # ...
from mysite.views import hello, current_datetime, hours_ahead, contact

 urlpatterns = [

     # ...

     url(r'^contact/$', contact),
]
```

Biz (modifiye veri etkisi olabilir) `POST` formu oluştururken bu yana, biz Cross Site istek sahteciliğine dert etmenize gerek. Django buna karşı korumak için çok kolay kullanımlı sistemi ile birlikte geliyor, Neyse ki, çok zor endişelenmenize gerek yok. Kısacası, iç URL'ler hedefleyen tüm POST formları `{% csrf_token %}` şablonu etiketi kullanmalısınız. {% Csrf_token %} ile ilgili daha fazla bilgi Bölüm 19'da bulunabilir.

Bunu yerel olarak çalıştırmayı deneyin. Formu yükleyin, doldurulan alanlardan hiçbirini gönderin, geçersiz bir e-posta adresiyle gönderin, ardından nihai olarak geçerli verileri gönderin. (Tabii ki bir posta sunucusu yapılandırmadıysanız, `send_mail()` çağrıldığında bir `ConnectionRefusedError` alırsınız.)

## Changing How Fields Are Rendered
## Alanların Nasıl Oluşturulduklarını Değiştirme

Muhtemelen bu formun yerel olarak oluşturulduğunda fark edeceğiniz ilk şey, `message` alanının bir `<input type = "text">` olarak görüntülenmesi ve bir `<textarea>` olmasıdır. Bunu, alanın widget'ı ayarlayarak düzeltebilirsiniz:

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField()
    email = forms.EmailField(required=False)
    message = forms.CharField(widget=forms.Textarea)
```

Formlar çerçevesi, her alan için sunum mantığını bir widget grubuna ayırır. Her alan türü varsayılan bir widget'a sahiptir, ancak varsayılanı kolayca geçersiz kılabilir veya kendi alanınınıza özel bir widget sağlayabilirsiniz. Widgetler sunum mantığını temsil ederken `Field` sınıflarını "doğrulama mantığını" temsil ettiğini düşünün.

## Setting a Maximum Length
## Maksimum Uzunluğu Ayarlama

En yaygın doğrulama gereksinimlerinden biri, alanın belirli bir boyutta olup olmadığını kontrol etmektir. İyi bir önlem olarak, `subject`i 100 karakterle sınırlandırmak için `ContactForm`'umuzu geliştirmeliyiz. Bunu yapmak için, `CharField`'e bir `max_length` sağlayın, bunun gibi:

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    email = forms.EmailField(required=False)
    message = forms.CharField(widget=forms.Textarea)
```

İsteğe bağlı bir `min_length` bağımsız değişkeni de mevcuttur.

## Setting Initial Values
## Başlangıç Değerlerini Ayarlama

Bu forma bir gelişme olarak, `konu` alanına başlangıç değeri ekleyelim: "Sitenizi seviyorum!" (Biraz öneri gücü zarar veremez.) Bunu yapmak için, bir ilk oluşturduğumuzda `initial` ilk argümanı kullanabiliriz. `Form` örneği:

```python
def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            cd = form.cleaned_data
            send_mail(
                cd['subject'],
                cd['message'],
                cd.get('email', 'noreply@example.com'),
                ['siteowner@example.com'],
            )
            return HttpResponseRedirect('/contact/thanks/')
    else:
        form = ContactForm(
            initial={'subject': 'I love your site!'}
        )
    return render(request, 'contact_form.html', {'form':form})
```

Şimdi, `konu` alanı, bu tür açıklamayla önceden doldurulmuş olarak görüntülenecektir. Başlangıç verilerini iletme ile forma bağlanan verileri geçirme arasında bir fark olduğunu unutmayın. En büyük fark, yalnızca başlangıç verilerini geçiyorsanız, formun ilişkisiz olacağıdır; bu, herhangi bir hata mesajı taşımayacağı anlamına gelir.

## Custom Validation Rules
## Özel Doğrulama Kuralları

Geribildirim formumuzu başlattığımızı ve e-postaların yuvarlanmaya başladığını düşünün. Tek bir sorun var: Gönderilen mesajların bir kısmı yalnızca bir veya iki kelimeydi ve bu bize mantıklı gelebilecek kadar uzun değil. Yeni bir doğrulama politikası benimsemeye karar veriyoruz: dört kelime veya daha fazlası, lütfen.

Özel doğrulamayı bir Django formunda kanca yapmanın birkaç yolu vardır. Kuralımız tekrar tekrar kullanacağımız bir şeyse, özel bir alan türü oluşturabiliriz. Çoğu özel doğrulama tek seferlik işlerdir ve doğrudan `Form` sınıfına bağlanabilir. Mesaj alanıyla ilgili ek doğrulama istiyoruz, bu nedenle `Form` sınıfımıza bir `clean_message()` yöntemi ekleyin:

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    email = forms.EmailField(required=False)
    message = forms.CharField(widget=forms.Textarea)

    def clean_message(self):
        message = self.cleaned_data['message']
        num_words = len(message.split())
        if num_words < 4:
            raise forms.ValidationError("Not enough words!")
        return message
```

Django'nun form sistemi otomatik olarak adı `clean_` ile başlayan ve alan adı ile biten herhangi bir yöntemi arar. Böyle bir yöntem varsa, doğrulama sırasında çağırılır. Özellikle, belirli bir alan için varsayılan doğrulama mantığından sonra `clean_message()` yöntemi çağrılır (bu durumda, gerekli bir `CharField` için doğrulama mantığı).

Alan verileri zaten kısmen işlendiğinden, onu `self.cleaned_data`'dan çekiyoruz. Ayrıca, değerin var olduğunu ve boş olmadığını kontrol etmemiz konusunda endişelenmemize gerek yoktur; Bu varsayılan doğrulayıcı tarafından yapılır. Kelimelerin sayısını saymak için safça `len()` ve `split()` kombinasyonlarını kullanırız. Kullanıcı çok az kelime girdiyse, `forms.ValidationError` hatası fırlatırız :)

Bu özel duruma eklenen dize, hata listesinde bir öğe olarak kullanıcıya gösterilir. Metodun sonundaki alan için temizlenen değeri açıkça döndürmemiz önemlidir. Bu, özel doğrulama yöntemimizde değeri değiştirmemizi (veya farklı bir Python türüne dönüştürebilmemizi) sağlar. İade ifadesini unutursak, Hiçbiri iade edilmez ve orijinal değer kaybolur. `None` olur.

## Specifying labels
## Etiketleri belirtme

Varsayılan olarak, Django'nun otomatik olarak oluşturulan HTML formundaki etiketler, alt çizgiyi boşluklarla değiştirip ilk harfi büyük harfle yazarak oluşturulur; böylece `email` alanının etiketi `Email` olur. Django'nun modelleri, alanlar için varsayılan `verbose_name` değerlerini hesaplamak için kullandığı basit algoritmayla aynıdır. Bunu Bölüm 4'te ele almıştık. Fakat, Django'nun modellerinde olduğu gibi, belirli bir alan için etiketi özelleştirebiliriz. Sadece şu `label` etiketini kullanın:

```python
class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    email = forms.EmailField(required=False, label='Your e-mail address')
    message = forms.CharField(widget=forms.Textarea)
```

## Customizing Form Design
## Form Tasarımını Özelleştirme

`contact_form.html` şablonumuz, formu görüntülemek için `{{form.as_table}}` kullanıyor ancak formu daha ayrıntılı olarak kontrol ekranı üzerinde kontrol edebilmek için başka yollarla görüntüleyebiliriz. Formların sunumunu özelleştirmenin en hızlı yolu CSS'dir.

Özellikle hata listeleri bazı görsel iyileştirmelerle yapılabilir ve otomatik olarak oluşturulmuş hata listeleri, bunları CSS ile hedefleyebilmeniz için `<ul class = "errorlist">` 'ı kullanır. Aşağıdaki CSS gerçekten hatalarımızı ön plana çıkarmaktadır:

```css
<style type="text/css">
    ul.errorlist {
        margin: 0;
        padding: 0;
    }
    .errorlist li {
        background-color: red;
        color: white;
        display: block;
        font-size: 10px;
        margin: 0 0 3px;
        padding: 4px 5px;
    }
</style>
```

Formumuzun HTML'si bizim için üretilmeye elverişli ise de, çoğu durumda varsayılan işleme geçersiz kılmak isteyeceksiniz. {{ form.as_table }} ve arkadaşlar uygulamanızı geliştirirken yararlı kısayollardır, ancak bir formun görüntülenme biçimiyle ilgili her şey çoğunlukla şablonun içinde geçersiz kılınabilir ve muhtemelen bunu kendiniz bulacaksınız.

Şablonda `{{ form.fieldname }}` öğesine erişerek, her alanın widget'ı `(<input type = "text">, <select>, <textarea>` vb.) Tek tek oluşturulabilir ve bir alanla ilişkili herhangi bir hata {{ Form.fieldname.errors }} gibi.

Bu düşünceyle, aşağıdaki şablon koduyla iletişim formumuz için özel bir şablon oluşturabiliriz:

```html
<html>
<head>
    <title>Contact us</title>
</head>
<body>
    <h1>Contact us</h1>

    {% if form.errors %}
        <p style="color: red;">
            Please correct the error{{ form.errors|pluralize }} below.
        </p>
    {% endif %}

    <form action="" method="post">
        <div class="field">
            {{ form.subject.errors }}
            <label for="id_subject">Subject:</label>
            {{ form.subject }}
        </div>
        <div class="field">
            {{ form.email.errors }}
            <label for="id_email">Your e-mail address:</label>
            {{ form.email }}
        </div>
        <div class="field">
            {{ form.message.errors }}
            <label for="id_message">Message:</label>
            {{ form.message }}
        </div>
        {% csrf_token %}
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

{{ form.message.errors }} hatalar varsa bir `<ul class = "errorlist">` ve alan geçerli ise (veya form bağlanmamışsa) boş bir dize görüntüler. `form.message.errors` öğelerini bir Boolean olarak da ele alabilir ya da bir liste halinde yinelenebiliriz. Örneğin:

```html
<div class="field{% if form.message.errors %} errors{% endif %}">
    {% if form.message.errors %}
        <ul>
        {% for error in form.message.errors %}
            <li><strong>{{ error }}</strong></li>
        {% endfor %}
        </ul>
    {% endif %}
    <label for="id_message">Message:</label>
    {{ form.message }}
</div>
```

Doğrulama hataları durumunda, bu, `<div>` alanına bir "errors" sınıfı ekleyecek ve hataların listesini sırasız bir listede gösterecektir.

## What’s Next?
## Sırada ne var?

Bu bölüm, "çekirdek müfredat" olarak adlandırılan bu kitapta yer alan giriş materyalini tamamlar. Kitabın sonraki bölümünde, Bölüm 7-13, bir Django uygulamasının nasıl dağıtılacağı da dahil olmak üzere ileri düzeyde Django kullanımı hakkında daha ayrıntılı olarak incelenmektedir (Bölüm 13 ). Bu ilk yedi bölümden sonra, kendi Django projelerinizi yazmaya başlamak için yeterli bilgiye sahip olmanız gerekir. Bu kitaptaki materyalin geri kalan kısmı, eksik parçaların ihtiyacınız olduğunda doldurulmasına yardımcı olacaktır. Bölüm 7'den başlayacağız, iki katına geri dönüp, daha önce Bölüm 2'de tanıtılan görünümlere ve URLconf'lara daha yakından bir göz atacağız.
