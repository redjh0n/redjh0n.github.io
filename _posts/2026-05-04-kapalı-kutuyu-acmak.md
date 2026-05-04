---
layout: post
title: "📦 Kapalı Kutuyu Açmak: Bir Yazılımın İçini Okumak Mümkün mü?"
date: 2026-05-04
categories: [Bilgisayar Bilimleri, Reverse Engineering, Siber Güvenlik]
tags: [tersine-muhendislik, decompile, malware-analizi]
image:
  path: /assets/img/posts/04-05-2026/box.png
---

<style>
  /* Bu blok sadece bu yazıdaki resimleri hedef alır ve küçültür */
  article img {
    max-width: 500px !important; /* Resimleri 500 piksele sabitler */
    width: 100% !important;
    height: auto !important;
    display: block !important;
    margin: 20px auto !important; /* Ortalar */
    border-radius: 10px;
  }
</style>

Bu konuya tam başlamadan önce biraz programlama dillerinden bahsetmemiz gerekiyor. Bilgisayarlar özünde sadece 0 ve 1 rakamlarından anlar ve iletişimi de bu şekilde sağlar. Buna machine code (makine kodu) diyoruz. Biz insanlar için ise bu sayılar pek anlamlı ve kullanılabilir değildir; işte bu yüzden programlama dilleri icat edilmiştir.

![binary](/assets/img/posts/04-05-2026/binary.png)

Programlama dilleri genel olarak düşük, orta ve yüksek seviye olmak üzere üçe ayrılır. Düşük seviyeli diller, assembly’den ve machine code dediğimiz 0 ve 1’lerden oluşur. Assembly dediğimiz dil ise 0 ve 1’lere sembolik adlandırmalar atanan ve bu adlar ile yazılmaya çalışılan, yazması ve anlaması çok uzun süren allahın belası bir dildir...

![Assembly dilinde hello world yazılması](/assets/img/posts/04-05-2026/assembly.png)


Orta seviyeli diller (C, C++) ise daha makineye hitap eden, yüksek seviyeli diller (Python, C#) ise daha insancıl olan dillerdir. Özetle, dilin seviyesi düştükçe makineye; yükseldikçe ise insana olan yaklaşımı artar.

Peki bu orta ve yüksek seviyeli diller arasındaki fark nedir? diye sorarsanız;

Orta seviyeli dillerde bellek yönetimini kendiniz yapmanız gerekiyor. Yani heap’te bir değişken oluşturursanız onu sonradan kendiniz serbest bırakmalısınız. Bırakmazsanız memory leak (bellek sızıntısı) sorunu yaşarsınız ve bilgisayarınızın belleği gereksiz yere şişmiş ve alan kaplamış olur. Bu dillerin artısı ise bilgisayar diline olan yakınlığından dolayı yazdığınız programların genellikle yüksek performansla çalışmasıdır.

Yüksek seviyeli dillerde ise garbage collector (çöp toplayıcısı) dediğimiz bir yapı vardır. Bu yapı her zaman çalışır ve kullanılmayan verileri kendi kendine temizler; sizin müdahalenize gerek yoktur. (Bu arada bu, sizin müdahale edemeyeceğiniz anlamına gelmez!) Bu dillerin avantajı ise insanlara daha çok hitap etmesinden dolayı daha hızlı veya daha kolay kod yazabilmeyi sağlamasıdır (umarım öyledir :d).

Şimdi dillerden bahsettiğimize göre yazdığımız kodların nasıl makine koduna, yani makinenin anlayacağı dile çevrildiğinden bahsedelim.

![compiler](/assets/img/posts/04-05-2026/compiler.png)

Makineye yazdıklarımızı tercüme eden yapıya compiler (derleyici) diyoruz ama tam olarak öyle sayılmaz. Derleyici tek başına çalışmıyor ve bir süreçten geçiyor. Bu süreç aşağıdaki maddelerde detaylıca verilmiştir:

1- Preprocessor (Önişlemci)
Sizin kodlarınızda bulunan boşlukları ve yorum satırlarını silip, eklediğiniz kütüphaneleri veya sabit tanımladığınız değerleri bulup yerlerine koyan yapıdır. Kütüphanelerin asıl yerlerini bulup derleyiciye iletir, sabit değerlerin kullanıldığı yerleri bulup sabit değer (örneğin 3.14 ise) tüm yerlere yazar. İş yükünü hafifletir.
Örnek görmek isterseniz, bir C programı yazıp "gcc -E kod.c -o kod.i" komutunu çalıştırdıktan sonra herhangi bir editör ile kod.i dosyasını okuyarak neler olduğunu canlı olarak görebilirsiniz.

2- Compiler (Derleyici)
Yukarıdaki çıktıyı alır, syntax (dil bilgisi) veya mantıksal hatalar (sayı ile yazıyı toplamaya çalışmak gibi) var mı diye bakar. Eğer varsa derlemeyi durdurur, size hata çıktısını verir ve sizin hataları düzeltip tekrar derlemeniz gerekir. Böylece derleme işlemi baştan başlar. Ek olarak 
x = 10 + 5; gibi bir satır olduğunu düşünelim. Derleyici bu satıra bakar ve gereksiz işlemler ile işlemciyi meşgul etmek istemez, bu değeri direkt 15 olarak alır. Bu işlemlerden sonra kodu assembly diline çevirir.

3- Assembler (Çevirici)
Assembly dili ile yazılan çıktıyı alır ve 0 ve 1’lere çevirir.

4- Linker (Bağlayıcı)
Aşağıda detaylıca bahsedildi.


Şimdi biz kodları yazdık, sonra 0 ve 1’lere çevirdik ama kodlar için ekrana bir şey yazdırmak veya input (girdi) almak istediğimizde işletim sistemi ile çalışmamız gerekir. Çünkü bizim genel sürecimizi yöneten, donanım ve yazılım arasındaki bağlantıyı sağlayan kısım işletim sistemidir ve onun kernel’ıdır.

Bu durumda son olarak linker (bağlayıcı) dediğimiz kısımda ise derleme sonucu oluşan 0 ve 1’lerimiz işletim sisteminin kullandığı hazır kodlarla bütünleştirilir ve paketlenir. Bunun sonucunda bize çalıştırılabilir bir formatta verilir. Biz bu dosyaları çalıştırdığımızda ise bu veriler RAM’e yüklenir, RAM de bunları işlemciye yönlendirir ve programımız çalışır.
Örnek: .exe dosyaları veya Linux ortamında çalıştırılan .elf türündeki dosyalar.

![](/assets/img/posts/04-05-2026/linker.png)

Özetle yorum satırlarından tutun, yazdığınız değişken isimlerine kadar hiçbir şey özünde bilgisayar için bir şey ifade etmez. O sadece 0 ve 1’leri ister (ne anlatırsanız anlatın, anlattıklarınız karşınızdakinin anlayabileceği ve anlamak istediği kadardır tarzı bir şey demişti şair).

Bu nedenlerle orta seviyeli dilleri decompile etmeye çalıştığımız zaman orijinal kodu genelde elde edemeyiz; sadece kullandığımız araçlar (IDA Pro, Ghidra vb.) yardımı ile tahmini bir sözde kod (pseudocode) elde ederiz. Yani bizim kodumuzda parola veya kullanıcı adı gibi kelimeler geçerken sözde kodda rastgele r1, r2 gibi anlamsız değişkenler vardır.

![](/assets/img/posts/04-05-2026/ghidra.png)

Burada bahsedilen sözde kod da assembly’den gelmektedir. Decompiler’lar 0 ve 1’leri disassembler yardımı ile sadece assembly diline tamamen çevirebilirler, gelişmiş decompiler ise bu assembly kodlarını okur, tahmin eder ve mantık yürüterek sözde kod şeklinde size vermeye çalışırlar.

Sanırım burada az çok neden orta seviyeli dillerin tam olarak decompile edilemediğini anlamış olduk. Şimdi ise yüksek seviyeli dillere geçeceğiz.

Yüksek seviyeli dilleri genel olarak ikiye ayırıyoruz: sanal makine kullananlar (C#, Java) ve anında yorumlananlar (Python, JavaScript). Önce kısa olacak diye anında yorumlanan dillerden bahsedelim. Bu dillerin derlenme süreçleri yine aşağıda bahsedeceğim sanal makine dillerine benzerdir ama sadece çıktı olarak çalıştırılabilir dosya vermezler, yorumlayıcı ile çalıştırılırlar. Bu dosyaları direkt bir editör ile okuyabildiğimiz için decompiler olayına gerek yoktur.

Sanal makine dilleri ise C ve C++ gibi dillerin aksine birçok platforma uygun olarak çalışabilirler. Örnek olarak yazdığımız bir C kodu o an kullandığımız işletim sistemi mimarisine uygun olarak derlenir ve çalıştırılabilir hâle gelir ama C# veya Java gibi dillerde ise yazılan ve derlenen programlarımız daha önce sisteme yüklenilen sanal makineler (.NET, JVM) aracılığı ile hemen hemen desteklenen her ortamda çalışacaktır. Bu arada burada sanal makine deyince aklınıza işletim sistemleri gelmesin, bunlara sanal makine yanında runtime environment (çalışma zamanı ortamı) da denmektedir.

C# veya Java gibi dillerin derlenme süreci orta seviyeli dillere göre farklıdır, tamamen 0 ve 1’lere çevrilmezler; bunun yerine bir ara dile çevrilir ve sonrasında çalıştırılabilir bir dosya verirler.

![](/assets/img/posts/04-05-2026/il.png)

1- Burada bahsedilen ara dil aslında işlemcinin anladığı bir şey değildir; sadece daha önce bilgisayarımıza kurduğumuz sanal makinenin anlayabileceği bir şeydir. Bu ara dilde yazdığımız kodlardan bir şeyler silinmez, sadece bazı syntax kısımları sistemin daha iyi anlayabileceği hâle çevrilir. Özetle bizim yazdığımız kodun neredeyse aynısı ama sadece biraz daha kendince derli toplu ve paketlenmiş hâlidir. Bu ara dile bytecode veya IL (Intermediate Language) denir.

2- Ara dile çevrildikten sonra ise bize .exe, .dll veya .jar dosyası verir. Ama burada verilen .exe’yi C++ dilindeki .exe ile karıştırmayın. C++’taki .exe’miz sadece 0 ve 1’lerden oluşurken, buradan elde ettiğimiz .exe 0 ve 1’leri içermez (native olarak machine code içermez); ara dil için bir taşıyıcı veya bir nevi referans diyebiliriz.

3- Son olarak program çalıştırıldığında, dosya doğrudan makine kodu içermediği için işlemci bunu anlayamaz ve çalıştıramaz. İşte bu noktada JIT (Just In Time) dediğimiz yaklaşım devreye girer ve çalıştırdığımız dosyanın sanal makinesi bu işi üstlenerek program içindeki ara dili okumaya başlar, anlık olarak tercüme edip programın çalışmasını sağlar.

![JIT](/assets/img/posts/04-05-2026/jit.png)

Bu ara dil olayının artısı, sistem fark etmeksizin (Linux, Windows, macOS) sanal makine olduğu sürece yazdığınız kodun çalışabilmesidir.

Eksisi ise (ki bizim en çok işimize yarayan kısımdır aslında), .exe, .dll veya .jar dosyalarındaki ara dili çeşitli araçlar ile okuyarak asıl kodun büyük çoğunluğunu görebilmemizdir. Bu arada neden tamamını değil de büyük çoğunluğunu görebiliyoruz diye sorarsanız; bunun nedeni, bazı yapıların (örneğin foreach) arka planda while veya daha düşük seviyeli yapılara çevrilmesi ve yorum satırlarının tamamen silinmesidir.

Genel olarak özet geçmek gerekirse:

Derlenen dil C, C++ ise okuyabileceğimiz şey ya assembly’dir ya da decompiler araçlarının tahmini sözde kodlarıdır. Derlenen dil C#, Java gibi diller ise yine decompiler araçları ile kodun neredeyse tamamını okuyabiliriz.

Peki o zaman nasıl oluyor da yazılan her yazılım kopyalanamıyor, çalınamıyor veya okunamıyor?

Burada ise devreye obfuscation dediğimiz metot girer. Bu metot yardımı ile yazılımcılar kodlarını decompiler’lar tarafından anlaşılması zor hâle getirir. Örneğin değişken isimlerini değiştirir, kendi içinde verileri şifreler ve çalışmamak üzere sahte kodlar ekler. Bunlar çeşitli obfuscator araçlar yardımı ile yapılabilir. 

Obfuscation metodu tamamen güvenli ve kırılamaz bir yol değildir; sadece okumayı ve anlamayı zorlaştırmak, zaman kaybını artırmak için yapılır.

![Obfuscation edilmiş bir kod.](/assets/img/posts/04-05-2026/obfuscation.png)

Bir yazılımın içinin okunmasının ne kadar mümkün olup olmadığını, sanırım buraya kadar okuduysanız anlamışsınızdır. Yazı burada son buluyor, umarım sizler için faydalı olur. Sonraki yazılarda görüşmek üzere :)