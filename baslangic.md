  # Adımlar python3 sürümüne ve Django 1.8(LTS) sürümüne göre anlatılmıştır.
  
  1. Python package index kur. (sudo apt-get install python3-pip)
  2. Pip ile Virtual enviroment kur. (sudo pip3 install virtualenv)
  3. Proje için dizin oluştur ve içine gir(mkdir django_book sonra cd django_book gibi :))
  4. virtualenv yarat (virtualenv env_django_book) Benim isim biraz uzun oldu ama napalım
  5. Oluşturduğun virtualenv'ı aktif et (source env_django_book/bin/activate)
     Başarılı olduysa şöyle birşey gözükmeli (env_django_book)username@hostname:~/django_book$.
  6. Djangoyu kur. Biz burada 21 Aralık 2016 için 1.8(LTS) sürümünü kurduk (pip install django == 1.8.17)
  Ayrıca şu şekilde kurulmuş mu kontrol edebilirsin (django-admin --version)
  7. Projeyi oluştur. Sondaki boşluk ve noktaya dikkat (django-admin startproject mysite .)
  8. Gerekli veritabanı tablolarını oluştur. (python3 manage.py migrate)
  9. settings.py dosyasından sunucu ayarlarını yap. Burada izin verilen sunucuları düzenleyeceğiz aksi takdirde 
  siteye erişime izin verilmiyor. (nano ~/django_book/mysite/settings.py)
  Şu şekilde olacak ---- ALLOWED_HOSTS = ['your_server_domain_or_IP', 'second_domain_or_IP', . . .]
  10. Firewall dan 8000 numaralı portu aç. (sudo ufw allow 8000)
  11. Sunucuyu çalıştır. (python3 manage.py runserver 0.0.0.0:8000)
  12. Web tarayıcı adres satırına siteni alanadi.com:8000 olacak şekilde gir (server_ip_address:8000)
  13. Ta ta ta taaa tabi eğer çalıştıysa :D
  
  
  Digitalocean'ın kendi yardımları zaten anlatmış lakin biz yine de yazalım ki pratik olsun :) 
  https://www.digitalocean.com/community/tutorials/how-to-install-the-django-web-framework-on-ubuntu-16-04
  Ayrıca faydalandığımız kitap
  http://djangobook.com/installing-django/
