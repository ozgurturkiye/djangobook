# Django Form Validation
# Django Form Doğrulaması

## Simple Validation
## Basit Doğrulama

Arama örneğimiz, özellikle veri geçerliliği açısından makul derecede basittir; Yalnızca arama sorgusunun boş olmadığından emin olmak için kontrol yapıyoruz. Birçok HTML formu, değerin boş olmadığından daha karmaşık bir doğrulama düzeyi içerir. Web sitelerindeki hata mesajlarını hepimiz gördük:

1. "Geçerli bir e-posta adresi girin. 'Foo' bir e-posta adresi değil. "
2. "Lütfen geçerli beş basamaklı bir ABD posta kodu girin. '123' bir posta kodu değil. "
3. "Lütfen geçerli bir tarih, YYYY-AA-GG formatında giriniz."
4. "Lütfen en az 8 karakter uzunluğunda ve en az bir sayı içeren bir şifre girin."

arama() görünümünün, arama teriminin 20 karakterden az veya ona eşit olduğunu doğrular şekilde değiştirelim. (Örnek uğruna, daha uzun bir süre sorguyu yavaşlatabilecek bir şey olmasını söyleyin.) Bunu nasıl yapabiliriz?

Mümkün olan en basit şey, mantığı görünümde doğrudan gömmektir:

```python
def search(request):
    error = False
    if 'q' in request.GET:
        q = request.GET['q']
        if not q:
            error = True
        elif len(q) > 20:
            error = True
        else:
            books =
Book.objects.filter(title__icontains=q)
            return render(request, 'search_results.html',
                          {'books': books, 'query': q})
    return render(request, 'search_form.html',
        {'error': error})
```

Şimdi, 20 karakterden uzun bir arama sorgusu göndermeye çalışırsanız arama yapmanıza izin vermez; Bir hata mesajı alırsınız. Ancak `search_form.html` adresindeki şu hata mesajının "Lütfen bir arama terimi gönderiniz" yazıyor - bu nedenle her iki durumda da doğru olması için onu değiştirmeliyiz:

```html
<html>
<head>
    <title>Search</title>
</head>
<body>
    {% if error %}
        <p style="color: red;">
            Please submit a search term 20 characters or shorter.
        </p>
    {% endif %}

    <form action="/search/" method="get">
        <input type="text" name="q">
        <input type="submit" value="Search">
    </form>
</body>
</html>
```

Bununla ilgili çirkin bir şey var. Tek boyut uyan tüm hata mesajımız potansiyel olarak kafa karıştırıcıdır. Boş bir form gönderimi için hata mesajı neden 20 karakterlik bir sınırla ilgili bir şey söylemeliyiz?

Hata mesajları belirli, açık ve karışık olmayan olmalıdır. Sorun aslında, hata için basit bir boolean değer kullandığımız gerçeğidir, oysa bir hata mesajı dizisi listesi kullanmalıyız. Bunu nasıl düzeltebileceğimiz aşağıda açıklanmıştır:

```python
def search(request):
    errors = []
    if 'q' in request.GET:
        q = request.GET['q']
        if not q:
            errors.append('Enter a search term.')
        elif len(q) > 20:
            errors.append('Please enter at most 20 characters.')
        else:
            books =
Book.objects.filter(title__icontains=q)
            return render(request, 'search_results.html',
                          {'books': books, 'query': q})
    return render(request, 'search_form.html',
                  {'errors': errors})
```

Ardından, `search_form.html` şablonuna şimdi bir hata `boolean` değeri yerine bir hata `liste`si iletildiğini yansıtacak küçük bir düzenleme yapmamız gerekir:

```html
<html>
<head>
    <title>Search</title>
</head>
<body>
    {% if errors %}
        <ul>
            {% for error in errors %}
            <li>{{ error }}</li>
            {% endfor %}
        </ul>
    {% endif %}
    <form action="/search/" method="get">
        <input type="text" name="q">
        <input type="submit" value="Search">
    </form>
</body>
</html>
```

## Making a Contact Form
## İletişim Formu Yapma

Kitap arama formu örneğini birkaç kez tekrarladık ve güzel bir şekilde geliştirdik, yine de temelde basit: yalnızca bir alan, 'q'. Formlar daha karmaşık hale geldiğinde, kullandığımız her form alanı için yukarıdaki adımları tekrar tekrar yapmak zorundayız. Bu, insanın hatası için bir sürü zahmet ve bir sürü fırsat sunuyor. Bizim için şanslı olan Django geliştiricileri bunu düşündüler ve Django'ya form ve doğrulama ile ilgili görevleri yürüten daha üst düzey bir kütüphaneye yerleşiklerdi.

## Your First Form Class
## İlk Form Sınıfınız

Django, bu bölümde keşfettiğimiz birçok sorunu - HTML form görüntülemesinden doğrulamaya kadar işleyen `django.forms` adlı bir form kütüphanesine sahiptir. Django form çerçevesini kullanarak iletişim form uygulamamıza girelim ve yeniden çalışalım.

Form çerçevesini kullanmanın birincil yolu, üzerinde çalışmakta olduğunuz her HTML <form> için bir Form sınıfı tanımlamaktır. Bizim durumumuzda yalnızca bir tane <form> var, bu nedenle bir Form sınıfımız olacak. Bu sınıf, doğrudan istediğiniz yerlerde (doğrudan views.py dosyanızda) yaşayabilir ancak toplum kuralı, Form sınıflarını `forms.py` adlı ayrı bir dosyada tutmaktır.

Bu dosyayı `mysite/views.py` ile aynı dizinde oluşturun ve aşağıdakileri girin:

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField()
    email = forms.EmailField(required=False)
    message = forms.CharField()
```

Bu oldukça sezgisel ve Django'nun model sözdizimine benzemektedir. Formdaki her alan bir tür `Field` sınıfıyla temsil edilir - `CharField` ve `EmailField` burada kullanılan tek alan türüdür - bir `Form` sınıfının öznitelikleri olarak. Her alan varsayılan olarak zorunludur, bu nedenle e-postayı isteğe bağlı hale getirmek için `required = False` olarak belirtilir. Python interaktif yorumlayıcısına geçelim ve bu sınıfın neler yapabileceğini görelim. Yapabileceği ilk şey HTML olarak kendini göstermektir:

```python
>>> from mysite.forms import ContactForm
>>> f = ContactForm()
>>> print(f)
<tr><th><label for="id_subject">Subject:</label></th><td><input type="text" name="sub\
ject" id="id_subject" /></td></tr><tr><th><label for="id_email">Email:</label></th><t\
d><input type="text" name="email" id="id_email" /></td></tr><tr><th><label for="id_me\
ssage">Message:</label></th><td><input type="text" name="message" id="id_message" /><\
/td></tr>
```

Django, erişilebilirlik için <label> etiketlerinin yanı sıra her alana bir etiket ekler. Fikir, varsayılan davranışı mümkün olduğunca en iyi hale getirmektir. Bu varsayılan çıktı bir HTML <table> biçimindedir, ancak birkaç yerleşik çıkış vardır:

```python
>>> print(f.as_ul())
<li><label for="id_subject">Subject:</label>
<input type="text" name="subject" id="id_subject" /></li><li><label for="id_email">Em\
ail:</label> <input type="text" name="email" id="id_email" /></li><li><label for="id_\
message">Message:</label><input type="text" name="message" id="id_message"/></li>

>>> print(f.as_p())
<p><label for="id_subject">Subject:</label><input type="text" name="subject" id="id_s\
ubject"/></p><p><label for="id_email">Email:</label> <input type="text" name="email" \
id="id_email" /></p><p><label for="id_message">Message:</label><input type="text" nam\
e="message" id="id_message"/></p>
```

Açılış ve kapanış <table>, <ul> ve <form> etiketleri çıktıda bulunmadığına dikkat edin, böylece gerekirse ek satırlar ve özelleştirme ekleyebilirsiniz. Bu yöntemler, "tüm formun görüntülenmesi" gibi genel bir durum için kısayollardır. Ayrıca, belirli bir alan için HTML'yi görüntüleyebilirsiniz:

```python
Açılış ve kapanış <table>, <ul> ve <form> etiketleri çıktıda bulunmadığına dikkat edin, böylece gerekirse ek satırlar ve özelleştirme ekleyebilirsiniz. Bu yöntemler, "tüm formun görüntülenmesi" gibi genel bir durum için kısayollardır. Ayrıca, belirli bir alan için HTML'yi görüntüleyebilirsiniz:
```

Form nesnelerinin yapabileceği ikinci şey verilerin doğrulanmasıdır. Verileri doğrulamak için yeni bir Form nesnesi oluşturun ve alan adlarını veriye eşleyen bir veri sözlüğü verin:

```python
>>> f = ContactForm({'subject': 'Hello', 'email': 'adrian@example.com', 'message': 'N\
ice site!'})
```

Verileri bir Form örneği ile ilişkilendirdikten sonra, bir "sınır" formu oluşturdunuz demektir:

```python
>>> f.is_bound
True
```

Verisinin geçerli olup olmadığını öğrenmek için bağlı herhangi bir `Formda` `is_valid()` yöntemini çağırın. Her alan için geçerli bir değer geçtik, böylece `Form`un tamamı geçerli olur:

```python
>>> f.is_valid()
True
```

`E-post`a alanını geçemezsek, bu alan geçerli kalır, çünkü bu alan için `required=False` olarak belirlenmiştir:

```python
>>> f = ContactForm({'subject': 'Hello', 'message': 'Nice site!'})
>>> f.is_valid()
True 
```

Ancak, konu veya mesaj bırakırsak, Form artık geçerli değil:

```python
>>> f = ContactForm({'subject': 'Hello'})
>>> f.is_valid()
False
>>> f = ContactForm({'subject': 'Hello', 'message': ''})
>>> f.is_valid()
False 
```

Alana özel hata mesajları almak için aşağıya indirebilirsiniz:

```python
>>> f = ContactForm({'subject': 'Hello', 'message': ''})
>>> f['message'].errors
['This field is required.']
>>> f['subject'].errors
[]
>>> f['email'].errors
[]
```

Her bağlı Form örneği, alan adlarını hata mesaj listelerine eşleyen bir sözlük sağlayan bir errors niteliğine sahiptir:

```python
>>> f = ContactForm({'subject': 'Hello', 'message': ''})
>>> f.errors
{'message'`: ['This field is required.']}
```

Son olarak, verileri geçerli olarak bulunan `Form` örnekleri için `cleaned_data` özniteliği kullanılabilir. Bu, gönderilen verilerin "temizlendiği" bir sözlüğüdür. Django'nun form çerçevesi sadece verileri doğrulamakla kalmaz; Değerleri uygun Python türlerine dönüştürerek temizler:

```python
>>> f = ContactForm({'subject': 'Hello', 'email': 'adrian@example.com',
'message': 'Nice site!'})
>>> f.is_valid() True
>>> f.cleaned_data
{'message': 'Nice site!', 'email': 'adrian@example.com', 'subject':
'Hello'}
```

İletişim formumuz yalnızca dizge nesnelerine "temizlenmiş" dizelerle ilgilenir - ancak bir `IntegerField` veya `DateField` kullanırsak, form çerçevesi `cleaned_data`'nın verilen alanlar için uygun Python tam sayılarını veya `datetime.date` nesnelerini kullanmasını sağlar.
