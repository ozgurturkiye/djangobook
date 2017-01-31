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

