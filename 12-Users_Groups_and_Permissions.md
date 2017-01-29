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

