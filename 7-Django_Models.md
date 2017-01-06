# Django Models
# Django Modelleri

Bölüm 2'de, dinamik Web sitelerini Django ile kurmanın temellerini ele aldık:Bunlar görünümler ve URLconf oluşturmaydı. Açıkladığımız gibi, bazı rasgele mantık(arbitrary logic) yapmaktan ve ardından bir yanıt döndürmekten bir görünüm sorumludur. Örneklerden birinde, keyfi mantığım mevcut tarih ve saati hesaplamaktı.

Modern Web uygulamalarında, keyfi mantık genellikle bir veritabanı ile etkileşimi gerektirir. Sahnenin arkasında, veritabanı odaklı bir Web sitesi bir veritabanı sunucusuna bağlanır, bazı verileri dışarı alır ve bir Web sayfasında görüntüler. Site, site ziyaretçilerinin veritabanını kendi başlarına doldurmalarının yollarını da sağlayabilir.

Birçok karmaşık Web sitesi bu ikisinin bir kombinasyonunu sağlar. Örneğin, Amazon.com, bir veritabanı odaklı siteye mükemmel bir örnektir. Her bir ürün sayfası temel olarak HTML biçiminde biçimlendirilmiş Amazon ürün veritabanına bir sorgudur ve bir müşteri incelemesi yayınladığınızda incelemelerin veritabanına eklenir.
