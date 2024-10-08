# One-Definition-Rule

C++ dilinde "One Definition Rule" (ODR) olarak bilinen kural, bir programda her bir nesnenin, fonksiyonun, sınıfın, şablonun ve inline fonksiyonun yalnızca bir tanımının olması gerektiğini belirtir. 
ODR'nin amacı, belirsizlikleri ve çakışmaları önleyerek programın tutarlılığını ve daha doğru çalışmasını sağlamaktır. 
Yazılımsal bazı varlıkların proje içinde birden fazla bildirimi olabilir ancak bir tanımlaması olması gerekir. Tek bir tanımlaması olması zorunludur. 

C++ standartlarının da kullandığı bir terimdir.

Neler ODR’a tabii ?

* Fonksiyonlar
* Değişkenler
* Sınıflar
* Enum Türleri
* Şablonlar



Aynı kaynak dosyada aynı 2 tanım sentaks hatası, farklı kaynak dosyalarda aynı varlığın birden fazla tanımı doğrudan undefined behavior.
Eğer inline keywordünü yazarsak inline statüsüne girer yani link aşamasında linker bundan sadece 1 tane görecek ve ODR bozulmayacak.
Non-inline fonksiyonlar ve değişkenler, program genelinde tek bir tanıma sahip olmalıdır.
Inline fonksiyonlar ve değişkenler, birden fazla çeviri biriminde tanımlanabilir, ancak her tanımın aynı olması gerekir. (Token by token) 

Inline Anahtar Kelimesinin Rolü:
inline anahtar kelimesi, ODR'yi esneterek, farklı çeviri birimlerinde aynı tanımlara sahip fonksiyonların veya değişkenlerin olmasına izin verir. Anahtar noktalar şunlardır:

Bir inline fonksiyon, bir başlık dosyasında tanımlanabilir ve bu başlık dosyası farklı kaynak dosyalarına dahil edilse bile ODR'yi ihlal etmez.
Her inline fonksiyon tanımının aynı olması gerekir, böylece bağlayıcı (linker) bu tanımları aynı varlık olarak kabul eder.
Inline anahtar kelimesinin derleyiciye fonksiyonu inline etme (fonksiyon çağrısını fonksiyon koduyla değiştirme) önerisinde bulunduğunu, ancak esas amacının farklı çeviri birimlerinde aynı tanımlara sahip olmayı sağlamak olduğunu vurgular.

//.h
```C++

int x = 5; 
int x = 5;  // bu tanımlarda ODR ihlal ediliyor.ODR ihlal edilmesi Undefined behavior.
           // bildirim birden çok olabilir ama tanım tek olmalı
```

Öyle varlıklar var ki tanımlarının token by token aynı olması durumunda ODR ihlal edilmiyor.
```C++

void func(int x)
{

}
```

Projede 2 tane ayrı func fonksiyonu olabilir mi ? Hayır. 2 tane ayrı func fonksiyonunun olması duruma göre sentaks hatası olabilir duruma göre olmayabilir ama her halükarda ill-formed’dur. 

Sentaks hatası dendiğinde kastedilen derleyicinin compilation sürecinde diagnostic verebildiği durumlardardır. İll-formed bunun üstünde bir kavram. İll formed olması sonuçta dilin kurallarına aykırı yine (olmaması gereken bir durum) ama sentaks hatası biçiminde derleyicinin bir diagnostic mesaj verme zorunluluğu yok. 
Mesela projeyi oluşturan 2 tane kaynak dosya olsa ; furkan.cpp - bora.cpp

furkan .cpp
```C++
void foo(int x){}
```
Aynı projeye dahil bora.cpp'de böyle bir fonksiyon tanımlasam

bora .cpp 
```C++
void foo(int x){}
```

Ill-formed dur. 
Ama böyle bir şey yaparsam ne olur sorusunun cevabı compile time’da hata alırız olamaz ki. Derleyicinin çeviri birimi kaynak dosya(translation unit). 
Derleyici furkan.cpp yi ayrı derliyor bora.cpp yi ayrı derliyor. 
Dolayısıyla birden fazla kaynak dosyada birden fazla fonksiyonun tanımının bulunması bir sentaks hatası değil ama ill-formed. Linker bir hata verebilir fakat bazı durumlarda linker’da vermeyebilir. 

ODR’i aynı kaynak dosyada ihlal ederseniz sentaks hatası. 

// file1.cpp
```C++
int x = 5;
int x = 5; // ODR ihlali, sentaks hatası
```

ODR ihlali. ODR aynı kaynak dosyada ihlal edildi. İll formed olmakla birlikte aynı zamanda sentaks hatası. 

file1.cpp
```C++

class Myclass { 
};

class Myclass { 
};
```

Aynı kaynak dosyada ODR’ı ihlal ettiğimiz için sentaks hatası, derleyici diagnostic vermek zorunda. 

Farklı kaynak dosyalarda ODR’ı ihlal etsek bile derleyici bunun farkına varamaz. 

Öyle durumlar var ki, ODR görünürde ihlal edilmiş gibi görünmesine karşın orada istisnalar var. Bu istisnalar sayesinde ill-formed bir durum oluşmuyor. 

Bunlardan bir tanesi sınıflara ilişkin. Eğer bir class definition birden fazla kaynak dosyada token by token aynıysa bu ODR ihlali sayılmıyor.

//source1.cpp
```C++
class Nec {
	int foo(int)
	int bar(double)
};
```

//source1.cpp
```
class Nec {
	int foo(int)
	int bar(double)
};
```

token by token ne demek ? Derleyici aynı şekilde tokenizing yapacak. Derleyici açısından kodun en küçük birimine token denir. 
Bir sınıfın tanımını bir başlık dosyasına koymak kesinlikle ODR ihlali sayılmaz. Çünkü include ile dahil edildiği için token by token aynı olmak zorundadır. Tanımsız davranış oluşturmaz.
Koşullu derleme olayına dikkat etmek gerekir. 

Fonksiyonlar kesinlikle ODR’a tabii. 
Ben bir header file oluşturuyorum. 
// neco.h

```C++

void foo(int x)
{
}
```
Başlık dosyasına böyle bir fonksiyonun tanımını koymak tanımsız davranış. ODR'ı ihlal etmiş oluyoruz. Ya aynı proje içinde birden fazla kaynak dosya neco.h'yı include ederse, işte o zaman ODR çiğnenmiş olur. 
ODR’ı ihlal etmek tanımsız davranış. ODR violation = ODR ihlali. Fonksiyonun bildirimi birden fazla olabilir bunda bir sakınca yok ancak tanımlaması tek olmalıdır. Bu ihlal edilmemeli.

Başlık dosyasına .h ile .hpp vermek arasında bir fark yok. Konvansiyonel olarak fark var. 

Projede birden fazla kaynak dosyada fonksiyonun tanımının olması gerekiyor → fonksiyonun ODR ihlali için 

ODR ile konumuzun ne ilgisi var peki ? 

Şöyle bir fonksiyon tanımlasam ;
```C++

int foo(int x, int y)
{
	return x * x + y * y;
}
int bar(int a, int b)
{
	return foo(a,b);
}
```

Derleyici bir fonksiyon çağrısı ile karşılaştığında normal olarak ne yapıyor? 
Fonksiyona giriş çıkış kodlarını üretiyor ve obje koda hangi fonksiyonu çağırdığına yönelik referans bir isim yazıyor. Çağrılan fonksiyon ile çağıran fonksiyonun bağlantısını yapmak linker programının görevi. Linker obje koddaki kendisine yönelik yazılmış referansları takip ederek bir fonksiyonun bir fonksiyonu çağırması durumunda o obje kodları birleştiriyor. 

Böyle bir kod söz konusu söz olduğunda derleyici burada linker’a yönelik olarak oluşturacağı bar için oluşturacağı obje kodda Linkera bir referens isim mi yazar ? Hayır. 
Eğer engelleyen herhangi bir mekanizma yoksa derleyici inline expansion optimizasyonunu yapar. 

inline optimizasyon, compiler optimization^un en önemli bileşeni. En önemlisi. 

Derleyici bu optimizasyonu yapmasa maliyet artar. 
Zero cost abstraction ağırlıklı olarak inline expansion’a dayanıyor. 
inline expansionu disable edebilirsiniz. Bunların switchleri var. Debug sürecinde belirli optimizasyonları disable etmeyi tercih ederiz. 

Derleyicinin bu inline expansion dediğimiz optimizasyonu yapması hangi kriterlere bağlı ?
* Derleyicinin bunu yapmaktan bir fayda göreceğini bilmesi gerekiyor. Derleyici bir analiz yapıyor. Her fonksiyon çağrısının inline expand edilmesi avantaj getirmeyebilir. 
* Bazı inline expansion durumları derleyiciden derleyiciye değişiyor. Yani derleyiciye bağlı.
* Derleyici switchleri (Optimizasyonda neyi hedefliyorsunuz, code size ? zero optimization ?)
* Derleyicinin kodu görmesi gerekir. (Olmazsa olmaz bir koşul)

Clang derleyici inline expansion optimizasyonu yapmış ama gcc derleyicisi yapamamış. Bu olabilir mi ? Olabilir.
Derleyicinin muktedir olması lazım. Derleyiciyi implemente edenlerin başarısı. 
Mesela recursive bazı yapıları bazı derleyiciler inline expand edebiliyor ama bazılarının gücü yetmiyor. NRVO optimizasyonunda Clang gcc'den daha başarılı. 

Derleyici birçok durumda olduğu gibi fonksiyonun sadece bildirimi görseydi nereden bilsin ki inline expansion yapıp yapmayacağını ? fonksiyonun tanımını görmüyor. Fonksiyonun kodunu görmesi gerekiyor ama. 

Buradaki inline anahtar sözcüğünün anlamı ne ?
Bu tür fonksiyonlara inline function deniyor. 

```C++

inline int  foo(int x, int y)
{
	return x*x + y*y;

}
```

Inline anahtar sözcüğünü koyduğumuzda derleyici inline expansion yapması ricasında bulunmuş oluyorum. 

Bu doğru değil. Kesinlikle yanlış.

Fonksiyonu inline yapmanız veya yapmamanız derleyicinin inline expansion yapması üzerinde kesinlikle hiçbir belirleyiciliği yok. 

Inline function ODR ile ilgili. Derleyicinin inline expansion olanağını elde etmesi için fonksiyonun kodunu görmesi gerekir değil mi ? Aksi mümkün değil. 

Undefined behaviorun temel sebeplerinden biri compiler optimization'dur. Compilerın tek talebi kodda undefined behavior olmaması. 

Derleyicinin mutlaka fonksiyonun tanımını görmesi gerekmektedir. Bildirim değil. Tanım!!!!

Inline fonksiyonlar inline keyword ile header file içerisinde kullanılıyorlar. 
Bunu kaç kaynak dosya include ederse etsin func fonksiyonu hepsinde olacak ama one definiton rule ihlal edilmiyor. 
Böylece bir inline fonksiyonunu başlı dosyasına koyarak
* Tanımsız davranış engelleniyor.
* Derleyiciye inline optimizasyon yapma seçeneği sunuyoruz. 

Sonuç olarak inline fonksiyonlar:
* Tipik olarak başlık dosyasında tanımlanan
* ODR ihlali yapmayan
* Derleyiciye inline optimizasyon yapma imkanı sağlayan ama buna mecbur bırakmayan fonksiyonlardır. 

Bu optimizasyonu yapıp yapmadığını assembly koddan anlayabiliriz. 
 
İnline fonksiyonların negatif tarafları ? Kişinin inline fonksiyonu kullanmamak istememe  nedenleri; 

* Kodu ifşa etmiş oluyorum(expose etmiş oluyorum).Kodu herkese göstermiş oluyorum. Algoritmamı kimseye göstermek istemiyorum ama fonksiyonu inline yapıyorum. Eee nasıl göstermeyeceksin o zaman. O kodu herkes görecek. Saklama imkanı yok. Gizleme şansı yok. Endişe noktalarından biri bu olabilir. 


Başlık dosyasına bir fonksiyonun inline fonksiyon olarak tanımını koyarsam ODR’ı ihlal etmemiş oluyorum. 
External bir fonksiyonun tanımını başlık dosyasına koymanın yolu o fonksiyonu inline yapmak. Bu durumda ODR’ı ihlal etmemiş oluyoruz. Bu  başlık dosyasını include eden kodları derleyecek derleyiciye inline expansion optimizasyonu şansı veriyoruz. Bu şans kullanılmadığında isterse 100 tane kaynak dosya include etmiş olsun sentaks hatası olmayacak. 

Peki ya inline etmezse ? dil bunu garanti ediyor. 


Sınıfların üye fonksiyonlarını da inline yapabilirim. Inline expensina en uygun olan sınıfın üye fonksiyonları. Kodu küçük, one liner fonksiyonlar. 
```C++
int foo((int x, int y)
{
  return x * x + y * y;
}
```
Bu fonksiyonu inline yapmadım. Derleyici bu fonksiyona yapılan çağrıda inline expansion optimizasyonu uygulanabilir mi ? evet

```C++
inline int foo((int x, int y)
{
  return x * x + y * y;
}
```
Bu fonksiyon inline yaptım. Derleyici bu fonksiyona yapılan çağrıda inline xpensian yapmayabilir mi ? evet

Yani inline’ı kullanmazsanız inline expansion yapabilir
Inline’ı kullanırsanız inline expansion yapmayabilir. 

Eğer anlamı bu olsaydı Dünyanın en saçma anahtar sözcüğü olurdu :D 
Gereksiz bir anahtar sözcük olurdu. Doğrusu bu değil. İnline function ODR ile ilgili bir konu. 

Derleyicinin inline expansion olanağını elde etmesi için fonksiyonun kodunu görmesi gerekir değil mi ? evet
O zaman ben şimdi bir modül oluşturuyorum 
//serhat .h 
Burada client kodlar için bir interface var. 
Şimdi ben bu başlık dosyasında vereceğim fonksiyonlardan biri için inline expansion yapma olanağı vermek istesem - derlenecek client kodlara- mecbur tanımını buraya koymam gerekir. Tanımını buraya koymazsam derleyici nereden koyacak bunu? Ama tanımını buraya koyarsam ODR’ı ihlal etmiş olurum .O zaman benim öyle bir araca ihtiyacım var ki hem fonksiyonun tanımını başlık dosyasına koyup include eden kaynak dosyalara inline expansion sağlamak istiyorum ama aynı zamanda ODR’ı ihlal etmek istemiyorum. İşte inlineeee. 
Inline fonksiyon olduğu zaman bir inline fonksiyonun tanımı birden fazla kaynak dosyada token by token aynı olmak koşuluyla ODR ihlal edilmez. 

Ben bir başlık dosyasına fonksiyonu inline fonksiyon olarak tanımını koyarsam ODR’ı ihlal etmemiş oluyorum.

External fonksiyonun tanımını başlık dosyasına koymanın yolu o fonksiyonu inline yapmak. ODR’i ihlal etmiş olmuyoruz. 

Birden fazla kaynak dosya include etti. 

pelin.cpp
bora.cpp

Böylece bu kaynak dosyalarda func’a yapılan çağrıda inline expansion uygulayabilir derleyici. Peki ya uygulamazsa ? O zaman link zamanına geldiğimiz zaman pelin.cpp’de bir func bora.cpp’de bir func olacak mı ? Hayır. Dil bunu garanti ediyor. Link aşamasında sadece 1 tane instance olacak. 


// serhat.h 
```C++
inline int func(int x, int y)
{
	//
}
```

//pelin.cpp
```C++
#include "serhat.h"
void pel_func()
{
	auto fp = &func;
}
```
// bora.cpp
```C++
#include "serhat.h"
void bora_func()
{
	auto fp = &func;
}
```
RUn time söz konusu olduğunda pel_func çağrıldığında fp'nin değeri olan adresle, bora_func çağrıldığında fp'nin değeri olan adresin aynı olma garantisi var. 
Adreslerin aynı olma garantisi var. Inline fonksiyonunun önemli bir özelliği bu. 

Siz bir fonksiyonun tanımmını başlık dosyasına inline kullanarak eklerseniz; 

* ODR ihlali yok 
* Derleyiciye optimizasyon şansı veriyorsunuz
* Aynı adres olması garanti altına alınıyor. Link aşamasında 1 tane func olarak ele alacak. 


Bir başlık dosyam olsun, bora.h 
Burada fonksiyonu bildirirken 

```
int foo(int, int)
inline int foo(int x, int y)
```
Sentaks hatası yok. Bildirimde veya tanımda inline anahtar sözcüğünün bir kere geçmesi yeterli. 


Bundan sonraki kodların çok büyük çoğunluğunda zaman ekonomisi için fonksiyonların hemen hepsini inline olarak tanımlicam yani sınıf içinde tanımlıcam. Ama bu yanlış bir intiba vermesin. Üretimde hep böyle yapılmaz. Başlık dosyası ile uğraşmamak için. 


```C++
constexpr int foo(int x, int y)
{
	return x * x + y * y;
}
```

ODR ihlaline yol açar mı ? Hayır. Çünkü constexpr fonksiyonlar örtülüolarak inline (implicitly inline). Yazmış gibi muamele görecek. Constexpr fonksiyonları başlık dosyasına koymamız ODR’ı ihlal etmez. 

Başlık dosyasına inline fonksiyonların tanımlarını koyabilirim. Çoğu zaman kütüphaneler de member functionlar da olacak global fonksiyonlar da olacak. 

```C++
class Myclass {
public:
	void set(int a, int b);
private:
	int mx, my;
};
```

Sınıfların üye fonksiyonlarını da inline yapabilir miyim ? Evet, kesinlikle. Inline expansion'a en uygun olanlar neredeyse sınıfların üye fonksiyonları. Neredeyse bir garanti. Çoğu zaman one liner olan kodlar.
Bu tür kodları kendi irademle inline expansion yapmak istiyorum. Bunu yapmanın bir yolu şu;
* Fonksiyonun tanımını başlık dosyasına inline anahtar sözcüğü ile yazacaksınız.

Diğer yol ise; fonksiyon tanımını direkt üye fonksiyon içerisine koymak. Fonksiyonu doğrudan sınıfın içinde tanımlamak. 
```C++
class Myclass {
public:
	void set(int a, int b)
{
	mx = a;
	my = b;
}

private:
	int mx, my;
};
```

Myclass sınıfının set fonksiyonu bir inline fonksiyon.
Dilin kuralı diyor ki, eğer siz bir sınıfın static veya non static member functionunın tanımını sınıf içerisinde yaparsanız siz bu fonksiyonu implicitly inline yapmış oluyorsunuz. 


```C++
class Myclass {
public:
	void set(int x)
	{
		mx = x;
	}

	int get()const
	{
		return mx;
	}
private:
	int mx;
};

bool is_equal(const Myclass& m1, const Myclass& m2)
{
	return  m1.get() == m2.get();
}
```
ODR çiğnenir mi ? Evet. Kesinlikle böyle bir şey yapma. 

```
inline bool is_equal(const Myclass& m1, const Myclass& m2)
{
	return  m1.get() == m2.get();
}
```
Böyle olsaydı ODR çiğnenmeyecekti



Üye fonksiyonların sınıfların içinde tanımlanması —> inline fonksiyon

Fonksiyonun static olması implicitly inline olduğu anlamına gelmiyor. Ama ODR’ı ihlal etmiyor. 

// header file
```C++
static int foo(int x, int y)
{
	return x * x + y * y;
}
```
Başlık dosyasına böyle bir fonksiyonun tanımını koymam ODR ihlali olur mu ? Hayır, çünkü bu internal linkage'a ait. 
O zaman aşağıdaki gibi yazmamda bir sorun olur mu ? 
```C++
inline int foo(int x, int y)
{
	return x * x + y * y;
}
```
Sorun olur tabiki de. Böyle yazarsanız static keyword'ü ile yazarsanız bu başlık dosyasını include eden her kaynak dosyada ayrı bir foo fonksiyonu olacak. Adresleri farklı olacak. 
Static olması durumunda; bu kaynak dosyayı include eden her kaynak dosyada ayrı bir foo fonksiyonu olacak. Caner.cpp bu foo fonksiyonun adresini kullansa, ahmet.cpp’de bu foo fonksiyonunun adresini kullansa bunlar ayrı adresler olacak.
ODR'ın ihlal edilmemesi ayrı bir konu inline olması ayrı bir konu. 

