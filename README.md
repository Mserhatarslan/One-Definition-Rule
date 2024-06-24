# One-Definition-Rule

C++ dilinde "One Definition Rule" (ODR) olarak bilinen kural, bir programda her bir nesnenin, fonksiyonun, sınıfın, şablonun ve inline fonksiyonun yalnızca bir tanımının olması gerektiğini belirtir. 
ODR'nin amacı, belirsizlikleri ve çakışmaları önleyerek programın tutarlılığını ve düzgün çalışmasını sağlamaktır. Yazılımsal bazı varlıkların proje içinde  birden fazla bildirimi olabilir ama bir tanımlaması olması gerekir. Tek bir tanımlaması olması zorunludur. 

C++ standartlarının da  kullandığı bir terimdir.

Neler ODR’a tabii ?

* Fonksiyonlar
* Değişkenler
* Sınıflar
* Enum Türleri
* Şablonlar

  
Aynı kaynak dosyada aynı 2 tanım sentaks hatası, farklı kaynak dosyalarda aynı varlığın birden fazla tanımı doğrudan undefined behavior.
Eğer inline keywordünü yazarsak inline statüsüne girer yani link aşamasında linker bundan sadece 1 tane görecek ve ODR bozulmayacak
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
int x= 5;  // bu tanımlarda ODR ihlal ediliyor.ODR ihlal edilmesi Undefined behavior.
           // bildirim birden çok olabilir ama tanım tek olmalı
```

Öyle varlıklar varki tanımlarının token by token aynı olması durumunda ODR ihlal edilmiyor.
```C++

void func(int x)
{

}
```

Projede 2 tane ayrı func fonksiyonu olabilir mi ? Hayır. 2 tane ayrı func fonksiyonunun olması duruma göre sentaks hatası olabilir duruma göre olmayabilir ama her halükarda ill-formed’dur. 

Sentaks hatası dendiğinde kastedilen derleyicinin compilation sürecinde diagnostic verebildiği durumlardardır. İll-formed bunun üstünde bir kavram. İll formed olması sonuçta dilin kurallarına aykırı yine ama sentaks hatası biçiminde derleyicinin bir diagnostic verme zorunluluğu yok. 


furkan .cpp
```C++
void foo(int x){}
```

bora .cpp 
```C++
void foo(int x){}
```

Ill-formed dur. 
Ama bakın böyle bir şey yaparsam ne olurun sorusu compile time’da hata alırız olamaz ki. Derleyicinin çeviri birimi kaynak dosya(translation unit). 
Derleyici furkan.cpp yi ayrı derliyor bora.cpp yi ayrı derliyor. 
Dolayısıyla birden fazla kaynak dosyada birden fazla fonksiyonun tanımının bulunması bir sentaks hatası değil ama ill-formed. Linker bir hata verebilir fakat bazı durumlarda linker’da vermeyebilir. 

Odr’i aynı kaynak dosyada ihlal ederseniz sentaks hatası. 

// file1.cpp
```C++
int x = 5;
```
// file1.cpp
```C++
int x= 5; // ODR ihlali, undefined behavior
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

Öyle durumlar var ki ,ODR görünürde ihlal edilmiş gibi görünmesine karşın orada istisnalar var. Bu istisnalar sayesinde ill-formed durumu oluşmuyor. 

Bunlardan bir tanesi sınıflara ilişkin. Eğer bir class definition birden fazla kaynak dosyada token by token aynıysa bu ODR ihlali sayılmıyor.
token  by token ne demek ? 
```C++

class Nec {
	int foo(int)
	int bar(double)
};
class Nec {
	int foo(int)
	int bar(double)
};
```

token by token. Derleyici aynı şekilde tokenizing yapacak. 
Bir sınıfın tanımını bir başlık dosyasına koymak kesinlikle ODR ihlali sayılmaz. 
Fonksiyonlar kesinlikle ODR’a tabii. 
Ben bir header file oluşturuyorum. 

// neco.h
```C++

void foo(int x )
{
}
```

ODR’ı ihlal etmek tanımsız davranış. 

ODR violation. 

Belirli varlıkların bildirimi birden fazla kez yazılabilir ama tanımı tek olmalı.
Bu ihlal edilmemeli.


Header içinde bunu tanımlarsak one definition rule bozmaya çok açık hale geliyor kod.
bunu aynı projede 2 yada daha fazla kaynak dosya include ederse, aynı varlığın birden fazla tanımı olacak.Bu da UNDEFİNED BEHAVİOR.
```C++

void func(int x)
{
	
}
```

Eğer class token by token diğer kaynak dosyalarda da varsa one definition rule ihlal edilmemiş oluyor.
ÖR : Class definition header file a konursa ve bunu diğer modüller include ederse burada one definition rule ihlal edilmemiş oluyor.


Başlık dosyasına .h ile .hpp vermek arasında bir fark yok. Konvansiyonel olarak fark var. 

Projede birden fazla kaynak dosyada fonksiyonun tanımının olması gerekiyor → fonksiyonun ODR ihlali için 

ODR ile konumuzun ne ilgisi var peki ? 

Şöyle bir fonksiyon tanımlasam ;
```C++

int foo(int x, int y)
{
	return x x + y y;
}
int bar(int a, int b)
{
	return foo(a,b);
}
```



Derleyici bir fonksiyon çağrısı ile karşılaştığında normal olarak ne yapıyor ? fonksiyona giriş çıkış kodlarını üretiyor ve obje koda hangi fonksiyonu çağırdığına yönelik referans bir isim yazıyor. Çağrılan fonksiyon ile çağıran fonksiyonun bağlantısını yapmak linker programının görevi .Linker obje koddaki kendisine yönelik yazılmış referansları takip ederek bir fonksiyonun bir fonksiyonu çağırması durumunda o obje kodları birleştiriyor. 

Böyle birr kod söz konusu söz olduğunda derleyici burada linker’a yönelik olarak oluşturacağı bar için oluşturacağı obje kodda linkera bir referens isim mi yazar ? hayır. 
Eğer engelleyen herhangi bir engel yoksa derleyici inline expansion optimizasyonunu yapar. 

inline optimizasyon, compiler optimization^un en önemli bileşeni. En önemlisi. 

Hocanın yazdığı kodu buraya yaz. 

Derleyici bu optimizasyonu yapmasa maliyet artar. 
zero cost abstraction ağırlıklı olarak inline expansion’a dayanıyor. 
inline expansionu disable edebilirsiniz. Bunların switchleri var. Debug sürecinde bir optimizasyonları disable etmeyi tercih ederiz. 

Derleyicinin bu inline expansion dediğimiz optimizasyonu yapması hangi kriterlere bağlı ?
Derleyicinin bunu yapmaktan bir fayda göreceğini bilmesi gerekiyor. Derleyici bir analiz yapıyor. 
Derleyici analizi
Bazı inline expansion durumları derleyiciden derleyiciye değişiyor. Yani derleyiciye bağlı
Derleyici switchleri (neyi hedefliyorsunuz, code size ? zero optimization ?)
Derleyicinin kodu görmesi gerekir. 


Clang derleyici inline expansion optimizasyonu yapmış ama gcc derleyicisi yapamamış. Bu olabilir mi ? olabilir derleyicinin muktedir olması lazım. Derleyiciyi implemente edenlerin başarısı. 
Mesela recursive bazı yapıları bazı derleyiciler inline expand edebiliyor ama bazılarının gücü yetmiyor. 

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

Bu doğru değil. Yanlış .

Fonksiyonu inline yapmanız veya yapmamanız derleyicinin inline expansion yapması üzerinde kesinlikle hiçbir belirleyiciliği yok. 

inline function ODR ile ilgili. Derleyicinin inline enpansion olanağını elde etmesi için fonksiyonun kodunu görmesi gerekir değil mi ? aksi mümkün değil. 

Undefined behaviorun temel sebeplerinden biri compiler optimization'dur. Compilerın tek talebi kodda undefined behavior olmaması. 

BURADAKİ TEKNİKLERE İNLİNE EXPANSİON DENİYOR.
İNLİNE EXPANSİON  = BİR FONKSİYON ÇAĞRISI YAPILMIŞ FAKAT DERLEYİCİ NORMALDE FONKSİYON ÇAĞRISI KARŞILIĞINDA, 
FONKSİYONA GİRİŞ(STACK DURUMU, FONKSİYONLARIN ARGÜMANLARININ PARAMETRELERE GEÇİRİLECEĞİ DEĞİŞKENLERİ, FONKSİYONUN RETURN DEĞERİNİN ADRESİ...), 
FONKSİYONDAN ÇIKIŞ(STACK İN ESKİ HALE GETİRİLMESİ GİBİ ...) VE LİNKERDA BİR REFERANS İSİM YAZIYOR.

Derleyicinin mutlaka fonksiyonun tanımını görmesi gerekmektedir. Bildirim değil. Tanım!!!!

Inline fonksiyonlar inline keyword ile header file içerisinde kullanılıyorlar. 
Bunu kaç kaynak dosya include ederse etsin func fonksiyonu hepsinde olacak ama one definiton rule ihlal edilmiyor. 
Böylece bir inline fonksiyonunu başlı dosyasına koyarak
* Tanımsız davranış engelleniyor.
* Derleyiciye inline optimizasyon yapma seçeneği sunuyoruz. 
BÖYLECE BİR İNLİNE FONKSİYONUNU BAŞLIK DOSYASINA KOYARAK

Sonuç olarka inline fonksiyonlar:
* Tipik olarak başlık dosyasında tanımlanan
* ODR ihlali yapmayan
* Derleyiciye inline optimizasyon yapma imkanı sağlayan ama buna mecbur bırakmayan fonksiyonlardır. 

Bu optimizasyonu yapıp yapmadığını assembly koddan anlayabiliriz. 

Dezavantajları da olabilir.
* Fonksiyonun tanımı tamamen görülüyor.
* Return değeri olarak user defined tür kullanırsak, bu türlere ilişkin bildirimleri de derleyicinin görmesi lazım. Bu durumda fazladan başlık dosyası include etmek gerekebilir. 


Başlık dosyasına bir fonksiyonun inline fonksiyon olarak tanımını koyarsam ODR’ı ihlal etmemiş oluyorum. 
External bir fonksiyonun tanımını başlık dosyasına koymanın yolu o fonksiyonu inline yapmak. Bu durumda ODR’ı ihlal etmemiş oluyoruz. Bu  başlık dosyasını include eden kodları derleyecek derleyiciye inline expansion optimizasyonu şansı veriyoruz. Bu şans kullanılmadığında isterse 100 tane kaynak dosya include etmiş olsun sentaks hatası olmayacak. 


Peki ya inline etmezse ? dil bunu garanti ediyor. Kayıttan dinleyip yaz. 



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



derleyicinin inline expansion olanağını elde etmesi için fonksiyonun kodunu görmesi gerekir değil mi ? evet
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

Böylece bu kaynak dosyalarda func’a yapılan çağrıda inline expansion uygulayabilir derleyici. Peki ya uygulamazsa ? o zaman link zamanına geldiğimiz zaman pelin.cpp’de bir func bora.cpp’de bir func olacak mı ? hayır. Dil bunu garanti ediyor. Link aşamasında sadece 1 tane instance olacak. 



Adreslerin aynı olma garantisi var. Inline fonksiyonunun önemli bir özelliği bu. 

ODR ihlali yok 
Derleyiciye optimizasyon şansı veriyorsunuz
Aynı adres olması garanti altına alınıyor. 
Bir başlık dosyam olsun, burada fonksiyonu bildirirken 

Sentaks hatası yok. 


Sentaks hatası yok. İnline anahtar sözcüğünün bir kere geçmesi yeterli. 


Sınıfların üye fonksiyonlarını da inline yapabilir miyim ? Evet. 
Derleyiciye inline expansion yapma şansı vermek istiyorum. 

Bunun birkaç yolu var. 
Yollardan biri şu, fonksiyonun tanımını başlık dosyasına inline anahtar sözcüğü ile koyacaksınız. 





Inline fonksiyon mu ? evet. Ama inline yok ? olsun. 
Dilin kuralı diyor ki, eğer siz bir sınıfın static veya non static member function’unu tanımını sınıf içinde yaparsınız implicit inline yapmış oluyorsunuz. 
set fonksiyonu inline fonksiyon. 

Bundan sonraki kodların çok büyük çoğunluğunda zaman ekonomisi için fonksiyonların hemen hepsini inline olarak tanımlicam yani sınıf içinde tanımlıcam. Ama bu yanlış bir intiba vermesin. Üretimde hep böyle yapılmaz. Başlık dosyası ile uğraşmamak için. 


Inline fonksiyonların ODR ile olan ilişkisini gördük. 

//bora.h

Inline fonksiyon mu ? hayır. 
ODR ihlaline yol açar mı ? açar. 
Birden fazla kaynak dosya bu başlık dosyasını include ettiğinde ODR ihlaline yol açar. 

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

Diğer yol ise; fonksiyon tanımını direkt üye fonksiyon içerisine koymak.
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
ODR çiğnenir mi ? evet. Kesinlikle böyle bir şey yapma. 

```
inline bool is_equal(const Myclass& m1, const Myclass& m2)
{
	return  m1.get() == m2.get();
}
```
Böyle olsaydı ODR çiğnenmeyecekti



Üye fonksiyonların sınıfların içinde tanımlanması —> inline fonksiyon

Fonksiyonun static olması implicit inline olduğu anlamına gelmiyor. Ama ODR’ı ihlal etmiyor. 



ODR ihlali olur mu ? Hayır. Çünkü bu internal linkage’ ait. 
Static olması durumunda; bu kaynak dosyayı include eden her kaynak dosyada ayrı bir foo fonksiyonu olacak. Caner.cpp bu foo fonksiyonun adresini kullansa, ahmet.cpp’de bu foo fonksiyonunun adresini kullansa bunlar ayrı adresler olacak. 







// header.h
```C++

#ifndef HEADER_H
#define HEADER_H
inline void func() {
    // Fonksiyon gövdesi
}
#endif // HEADER_H
```

// file1.cpp
```C++

#include "header.h"
```

// file2.cpp
```

#include "header.h"
```


Bu örnekte, inline fonksiyon func(), ODR'yi ihlal etmeden birden fazla çeviri biriminde tanımlanabilir, çünkü her tanım aynıdır.

Non-inline Fonksiyonlar ve Değişkenler: Program genelinde yalnızca bir tanıma sahip olmalıdır.
Inline Fonksiyonlar ve Değişkenler: Birden fazla çeviri biriminde tanımlanabilir, ancak tanımlar aynı olmalıdır.
Şablonlar: Inline fonksiyonlara benzer şekilde çalışır ve birden fazla aynı tanıma izin verir.
Bu kurallara uymak, C++ programlarının derleme hatalarından kaçınmasını ve tutarlı bir şekilde çalışmasını sağlar.
