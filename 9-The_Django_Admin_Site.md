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
