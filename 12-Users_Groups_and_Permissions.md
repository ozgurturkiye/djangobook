# Users, Groups and Permissions
# Kullanıcılar, Gruplar ve İzinler

Süper kullanıcı olarak oturum açtığınızdan, herhangi bir nesneyi oluşturmak, düzenlemek ve silmek için herhangi bir erişim hakkına sahipsiniz. Doğal olarak, farklı ortamlar farklı izin sistemleri gerektirir; herkes süper kullanıcı olmamalıdır. Django'nun yönetici sitesi, belirli kullanıcılara yalnızca ihtiyaç duydukları arayüz bölümlerine erişmek için kullanabileceğiniz bir izin sistemi kullanıyor. Bu kullanıcı hesaplarının, yönetici arayüzü dışında kullanılmak üzere yeterince kapsamlı olması gerekiyordu, ancak şimdilik bunları yalnızca admin kullanıcı hesapları olarak ele alacağız.

Bölüm 11'de, Django kimlik doğrulama sistemi ile kullanıcıları site çapında (yani yalnızca yönetici sitesi değil) nasıl yöneteceğinizi ele alacağız. Kullanıcıları ve izinleri yönetici arayüzü üzerinden başka herhangi bir nesne gibi düzenleyebilirsiniz. Bunu, bu bölümün başında, admin'in Kullanıcı ve Grup bölümleri ile oynadığımızda gördük.

Kullanıcı nesnelerinin, kullanıcıya yönetici arayüzünde ne yapmalarına izin verileceğini tanımlayan bir dizi alanla birlikte beklediğiniz standart kullanıcı adı, şifre, e-posta ve gerçek ad alanlarına sahiptir.
İlk olarak, üç tane boolean bayrakları var:

* "Aktif" bayrak, kullanıcının aktif olup olmadığını kontrol eder. Bu işaret kapalıysa ve kullanıcı giriş yapmaya çalışırsa, geçerli bir şifreyle olsa bile girişine izin verilmeyecektir.

* "Personel" bayrağı, kullanıcının yönetici arayüzüne (başka bir deyişle o kullanıcının kuruluşunuzdaki bir "personel üyesi" olarak kabul edilip edilmediğini) girmesine izin verilip verilmeyeceğini denetler. Aynı kullanıcı sistemi, genel (yani, yönetilmeyen) sitelere erişimi kontrol etmek için kullanılabilir (bkz. Bölüm 11), bu bayrak, genel kullanıcılar ve yöneticiler arasında ayrım yapmaktadır.

* "Süper kullanıcı" bayrağı, kullanıcıya yönetici arayüzündeki herhangi bir öğeyi eklemek, oluşturmak ve silmek için tam erişim sağlar. Bir kullanıcının bu bayrağı ayarlanmış olması durumunda o kullanıcı için tüm normal izinler (veya bunların eksikliği) dikkate alınmaz.

"Normal" yönetici kullanıcıları, yani aktif, süper kullanıcılı olmayan personel üyeleri, atanmış izinler aracılığıyla yönetici erişimi alırlar. Yönetici arayüzü üzerinden düzenlenebilen her nesne (ör. Kitaplar, yazarlar, yayıncılar) üç izin vardır: oluşturma izni, düzenleme izni ve silme izni. Bir kullanıcıya izin atamak, kullanıcı erişimine, bu izinlerle tanımlanan işlemi yapmaya izin verir. Bir kullanıcı oluşturduğunuzda, o kullanıcının izinleri yoktur ve kullanıcıya belirli izinler vermek size bağlı kalır.

Örneğin, bir kullanıcıya yayıncı ekleme ve değiştirme izni verebilirsiniz, ancak bunları silmek için izin vermezsiniz. Bu izinlerin nesne başına değil modele göre tanımlandığını unutmayın; bu nedenle "John, herhangi bir kitabı değiştirir" demesine izin verirler, ancak "John, Apress tarafından yayınlanan tüm kitaplarda değişiklik yapabilir" demesine izin vermez. "İkinci işlevsellik, nesne başına izinler biraz daha karmaşıktır ve bu kitabın kapsamı dışındadır ancak Django belgelerinde bulunmaktadır.

* Uyarı! Kullanıcıları ve izinleri düzenleme erişimi de bu izin sistemi tarafından kontrol edilir. Birisini kullanıcıları düzenlemeye izin verirseniz, kendi izinlerini düzenleyebilir, ki bu sizin istediğiniz gibi olmayabilir! Bir kullanıcıya kullanıcı düzenleme izni vermek aslında bir kullanıcıyı süper kullanıcıya dönüştürmektir.

Kullanıcıları gruplara da atayabilirsiniz. Bir grup, yalnızca o grubun tüm üyelerine uygulanacak bir izin kümesidir. Gruplar, kullanıcıların bir alt kümesine aynı izinleri vermek için kullanışlıdır.

## When and Why to Use the Admin Interface – And When Not to
## Yönetici Arayüzünü Ne Zaman ve Niye Kullanalım - Ve Ne Zaman Kullanmayalım

Bu bölüm üzerinde çalıştıktan sonra, Django'nun yönetim sitesini nasıl kullanacağınız hakkında iyi bir fikriniz olmalıdır. Ancak ne zaman ve neden onu kullanmak isteyebileceğinizi ve ne zaman kullanmayacağınız belirtmek istiyorum.

Django'nun yönetici sitesi özellikle teknik olmayan kullanıcılar veri girişi yapabilmek için ihtiyaç duydukları zaman parlar; Sonuçta, bu özelliğin arkasındaki amaç budur. Django'nun ilk geliştirildiği gazetede, tipik bir çevrimiçi özelliğin geliştirilmesi -örneğin, belediye arzında su kalitesi üzerine özel bir rapor- böyle bir şeye gidiyor:

* Projeden sorumlu muhabir, geliştiricilerden biriyle görüşür ve mevcut verileri açıklar.
* Geliştirici, bu verilere uyacak şekilde Django modelleri tasarlar ve daha sonra yönetici sitesini muhabir için açar.
* Muhabir, yönetim alanını kayıp veya yabancı alanları işaret etmesini denetler - daha sonradan daha iyi. Geliştirici, modelleri tekrar tekrar değiştirir.
* Modeller üzerinde mutabakata varıldığında muhabir, yönetici sitesini kullanarak veri girişi yapmaya başlar. Aynı zamanda, programcı herkese açık erişilebilir görünümleri / şablonları geliştirmeye odaklanabilir (eğlenceli kısım!).

Başka bir deyişle, Django'nun yönetici arayüzünün varlığı, içerik üreticilerinin ve programcıların eşzamanlı çalışmalarını kolaylaştırıyor. Bununla birlikte, bu belirgin veri girme görevlerinin ötesinde, yönetici sitesi birkaç durum daha yararlıdır:

* Veri modellerini inceleme: Birkaç model tanımladıktan sonra, bunları yönetici arayüzünde çağırıp bazı kukla veriler girmek oldukça yararlı olabilir. Bazı durumlarda, bu, veri modelleme hataları veya modellerinizle ilgili diğer sorunlar ortaya koyabilir.
* Edinilen verileri yönetme: Dış kaynaktan (ör. Kullanıcılar veya Web tarayıcıları) gelen verilere dayanan uygulamalar için yönetici sitesi, bu verileri incelemek veya düzenlemek için kolay bir yol sağlar. Veritabanınızın komut satırı yardımcı programının daha az güçlü fakat daha kullanışlı bir versiyonu olarak düşünebilirsiniz.
* Hızlı ve kirli veri yönetimi uygulamaları: Kendinize hafif bir veri yönetimi uygulaması oluşturmak için (örneğin, giderleri takip etmek için) yönetici sitesini kullanabilirsiniz. Yalnızca kendi ihtiyaçlarınıza göre bir şeyler inşa ediyorsanız, genel kullanım için değil, yönetici sitesi size uzun bir yol kat edebilir. Bu anlamda, bir elektronik tabloların güçlendirilmiş, ilişkisel bir versiyonu olarak düşünebilirsiniz.
* Ancak yönetici sitesi her şeyden önce ve sonunda değil. Verilere genel bir arayüz olarak tasarlanmamıştır, verilerinizin sofistike sıralanmasına ve aranmasına izin vermeye yönelik değildir. Bu bölümün başında söylediğimiz gibi, güvenilir site yöneticileri içindir. Bu tatlı noktayı göz önünde bulundurmak etkin yönetici sitesi kullanımının anahtarıdır.

## What’s Next?
## Sıradaki ne?

Şimdiye kadar birkaç model oluşturduk ve verileri düzenlemeye yönelik birinci sınıf arayüzü yapılandırdık. Bir sonraki bölümde, Web geliştirme sürecinin gerçek "et ve patatesleri" ne geçeceğiz: form oluşturma ve işleme.
