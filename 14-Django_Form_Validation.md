# Django Form Validation
# Django Form Doğrulaması

## Simple Validation
## Basit Doğrulama

Arama örneğimiz, özellikle veri geçerliliği açısından makul derecede basittir; Yalnızca arama sorgusunun boş olmadığından emin olmak için kontrol yapıyoruz. Birçok HTML formu, değerin boş olmadığından daha karmaşık bir doğrulama düzeyi içerir. Web sitelerindeki hata mesajlarını hepimiz gördük:

1. "Geçerli bir e-posta adresi girin. 'Foo' bir e-posta adresi değil. "
2. "Lütfen geçerli beş basamaklı bir ABD posta kodu girin. '123' bir posta kodu değil. "
3. "Lütfen geçerli bir tarih, YYYY-AA-GG formatında giriniz."
4. "Lütfen en az 8 karakter uzunluğunda ve en az bir sayı içeren bir şifre girin."
