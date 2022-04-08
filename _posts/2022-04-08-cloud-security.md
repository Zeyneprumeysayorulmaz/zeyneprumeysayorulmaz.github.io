---
title: Cloud Security ☁ | OWASP Cloud Top 10
---

Merhabalar, bu yazımda bulut güvenliği kavramından ve bir açık kaynak web uygulama güvenliği projesi olan OWASP tarafından paylaşılan En Önemli 10 Bulut Güvenliği Riski’ nden bahsedeceğim.

![]({{ 'assets/img/workload.png' | relative_url }})

Bulut güvenliği, bulut ortamınızın, bulutta tuttuğunuz uygulamaların ve verilerin korunmasına yönelik teknolojiler ve sistemlerden oluşur. Burada bulut sağlayıcının aynı zamanda müşterinin, sağlanan bulut hizmet modeline göre farklı rol ve sorumlulukları vardır. Bulut güvenliği; geleneksel BT güvenliğinden veri depolama, ölçeklendirme hızı, son kullanıcı sistemlerinin arayüzü gibi noktalarda farklılık gösterse de Confidentiality (C ), Integrity (I ) ve Availability (A ) olmak üzere temelde 3 hedefe (C-I-A) hizmet eder.

Cloud Security Alliance-CSA “bulut bilişim içinde güvenlik güvencesi sağlamak için en iyi uygulamaların kullanımını teşvik etme ve diğer tüm bilgi işlem biçimlerinin güvenliğini sağlamaya yardımcı olmak için bulut bilişim kullanımları hakkında eğitim sağlama” amaçlayan bir kuruluş olup yayınladığı raporlarda bulut bilişimi tehdit eden 7 unsurdan bahsetmektedir.

    ✔ Bulut bilişimin suistimal edilmesi ve kötüye kullanımı

    ✔ Güvenilir olmayan uygulama programlama ara yüzleri

    ✔ İçerideki kötü niyetli kişiler

    ✔ Paylaşılan teknoloji zayıflıkları

    ✔ Veri sızıntısı/kaybı

    ✔ Hesap, hizmet & trafik kaçırma

    ✔ Bilinmeyen Risk Profili

Bu maddelerden en önemlisinin veri sızıntısı/kaybı olduğuna dikkat çekerek veri kaybı ve veri ihlali olmak üzere 2 farklı kategoriye ayırmıştır. Bu kategoriler bulut bilişimde baş gösteren en önemli iki risk faktörü olarak karşımıza çıkmaktadır.

Bu noktada riskler bulut bilişim hizmet modeline göre değişim göstermekle beraber hizmet modellerinin birbiri ile bağlantısından dolayı bir katmanda gerçekleşen olay diğer katmanları da etkilemektedir.

IaaS, PaaS ve SaaS olmak üzere 3 bulut bilişim hizmet modelini ve güvenliğini ele alacak olursak;

SaaS hizmet modelinde bulut güvenliğinin sorumluluğu bulut sağlayıcı sorumluluğundadır. Müşteriye işlevsellik, süreklilik ve veri güvenliği sunmak esastır. Son kullanıcı platformundan müşteriye sunulan web uygulamalarında bulunan zafiyetler ve uygulamalara yönelik yapılan saldırılar tehdit oluşturmaktadır. Aynı zamanda bu modelde kullanılan Çoklu Kiracılık (Multi-Tenancy) risk oluşturabilmektedir. Birden fazla müşteriye hizmet eden bir uygulamada kullanıcılar birbirinden tamamen izole olmayabilir ve bunun sonunca kaynakların aşırı kullanımı veya kilitlenmesi gibi durumlar oluşabilmektedir. Bu noktada hizmetlerin kullanılabilirliği etkilenmekte ve DoS (Denial of Service) saldırılarına neden olmaktadır. Çoklu kiracılık modelinde yine yaşanan başka bir problem bant genişliğini ve aynı zamanda maliyeti düşürmek adına veri tekilleştirme yapılmasıdır. Saldırganlar bu amaçla geliştirilen araçları kullanarak bulutta depolanan dosyaya ve içeriğine ulaşabilmektedir. Erişilebilirlik ilkesini amaç edinen tüm modellerde olduğu gibi SaaS modelinde de verinin her yerden erişilebilir olması gereksinimi de ek bir güvenlik riskidir. Müşterilerin herhangi bir ağdan (halka açık güvenilir olmayan Wi-Fi bağlantıları gibi) ve cihazdan uygulamaya erişmesi saldırı vektörleri oluşturmaktadır.

PaaS hizmet modelinde müşterilere bir uygulama sunulmakla beraber bunu yönetebileceği bir ortam sağlanmaktadır. Müşteriye sunulan uygulama ve uygulamanın sunulduğu platform olmak üzere modelde bulunan bu iki ortamın güvenliği esastır. Uygulama güvenliği SaaS modelinde olduğu gibi güvenlik risklerini barındırmaktadır. Geliştiriciler, uygulama geliştirme yaşam döngüsünü ve beraberinde getirdiği değişikliklerle oluşacak süreci takibini yapmaktan sorumludur. Bu noktada alınan aksiyonlar kritiktir. Geliştiricilerin veri güvenliği, güvenli uygulama geliştirme gibi konularda bilinçli olması beklenmektedir. Aksi takdirde platform ve uygulama tarafında ciddi saldırı vektörleri oluşabilmektedir.

IaaS modelinde fiziksel ve sanal sunucular, depolama üniteleri ve bant genişliği gibi bileşenler cluster mimarisiyle sunulmaktadır. Müşteriler talepleri doğrultusunda satın aldıkları kaynaklar üzerinde yönetim sağlamaktadır. Bu modelde oluşan güvenlik zafiyetlerinin genellikle sanallaştırma kaynaklı olduğu görülmektedir. Fiziksel donanımlar üzerinde yapılan sanallaştırma işleminde yeni bir katman ortaya çıktığından saldırganlar bu katmanı kullanabilmektedir. Saldırgan gözünden bakıldığında sisteme girilebilecek daha fazla port, servis, işletim sistemi vs. ortaya çıkmakta bu da ayrı bir güvenlik gereksinimi oluşturmaktadır. Sanal makine kaynaklarının paylaşılması, sanal makineler üzerinde alınan ekran görüntüleri, sanal makinelerin başka bir sunucu üzerine taşınması gibi operasyonel anlamda yapılan tüm faaliyetler beraberinde güvenlik riskleri oluşturabilmektedir.
OWASP Cloud Top 10 Risks

OWASP bir açık kaynak web uygulama güvenliği projesidir. Yayınlamış olduğu raporda bulut sistemlerine yönelik en önemli 10 güvenlik riskinden bahsetmektedir.

![]({{ 'assets/img/owasp-top-10.png' | relative_url }})

R1. Accountability & Data Risk (Hesap Verilebilirlik & Veri Riski): Bir organizasyonun bünyesinde bulunan veri merkezindeki tüm katmanlarda tam kontrol ve güvenlik organizasyonun sorumluluğundadır. Kuruluş sahip olduğu verileri mantıksal ve fiziksel olarak korumakla yükümlüdür. Hizmet modeline göre değişiklik göstermekle beraber hizmetlerini barındırmak için public cloud kullanmayı tercih eden kuruluşlar hizmetlerinin kontrolünü kaybedebilir. Kuruluşlar bu noktada riskleri ve kritik sistemlerini iyi analiz etmelidir. Veri yedekleme, veri kurtarma gibi durumların öngörülmesi ve gerekli aksiyonların alınmasından emin olmalıdır.

R2. User Identity Federation (Kullanıcı Kimliği Federasyonu): Kuruluşlar hizmetleri taşırken kullanıcı kimlikleri üzerinde tam kontrol elde etmelidir. Kimlik ve erişim yönetimi (Identity & Access Management) bu noktada önem arz eder. Kimlik sağlayıcılar(IdP) ve servis sağlayıcılar(SP) arasında kullanıcı bilgilerini güvenli bir şekilde değiş tokuş etmemize olanak sağlayan XML tabanlı bir çözüm olan SAML bulut sistemleri ve uygulamalar arası erişim için sık tercih edilen yöntemdir. Doğru uygulanmaması halinde risk teşkil etmektedir.

R3. Legal & Regulatory Compliance (Yasal & Mevzuata Uygunluk): Bir ülkede güvenli olarak algılanan veriler başka bir ülkede güvenli olarak algılanmayabilir. Mevzuata uygunluk önemlidir. Örneğin Avrupa Birliği’ nin ağır gizlilik yasaları vardır; burada ABD’ de depolanan veriler Avrupa Birliği için uygun olmayabilir. Yine Türkiye’ deki KVKK gibi mevzuatlar gereği verileri yurt dışında bir bulut sağlayıcıda tutmak sorun teşkil edebilmektedir.

R4. Business Continuity & Resiliency (İş Sürekliliği & Esneklik): İş sürekliliği, bir BT kuruluşunun işlerini sürdürülebilir kılmak için gerçekleştirdiği faaliyetlerdir. Bulut kullanan bir kuruluş olması durumunda bulut faaliyetlerinin iş sürekliliği ve sorumluluğu bulut sağlayıcıya aittir. Buradaki risk sağlayıcının uygun sürekliliğe sahip olmaması riskidir. Hizmet Sürekliliği ve QoS (Quality of Service) bulut sağlayıcı tarafından önerilen sözleşmeye dayalı çözümler sağlanmalıdır.

R5. User Privacy & Secondary Usage of Data (Kullanıcı Gizliliği & İkincil Veri Kullanımı): Kullanıcılar sosyal medyalar gibi uygulamaları veya web siteleri kullandığında kişisel veriler bulutta depolanmaktadır. Sosyal medyaların çoğu kullanıcıların kişisel verilerini nasıl ele alacakları konusu belirsizdir. Kullanıcılar, bulut sağlayıcılarıyla hangi verilerin onlar tarafından ikincil amaçlar için kullanılıp kullanılamayacağından emin olmalıdır. Gelen giden URL’ ler, tıklamalar vs. ile doğrudan veya dolaylı olarak kullanıcı verilerinden veri çıkarılabilmektedir. Çoğu sosyal siteler ikincil kullanım amacıyla kullanıcı verilerini işlemektedir. Bu gibi durumlar da risk faktörü olarak karşımıza çıkmaktadır.

R6. Service & Data Integration (Servis & Veri Entegrasyonu): Organizasyonlar tescilli verilerin son kullanıcı ve bulut veri merkezi arasında aktarılırken yeteri kadar korunduğundan emin olmalıdır. Verilerin İnternet üzerinden iletildiği bir bulut bilgi işlem modeli kullanan kuruluşlar için verilerin aktarım sırasında ele geçirilmesi her kuruluş için endişe kaynağı olsa da, risk çok daha büyüktür. Güvenli olmayan veriler, iletim sırasında müdahaleye ve tehlikeye açıktır.

R7. Multi-tenancy & Physical Security (Çoklu Kiracılık & Fiziksel Güvenlik): Bulutta çoklu kiracılığın, kaynakların ve hizmetlerin birden çok istemci (CPU, ağ iletişimi, depolama/veritabanları, uygulama yığını) arasında ortak kullanımı olduğundan bahsetmiştik. Kiracıların kasıtlı olarak ve kasıt olmaksızın birbirinden izole olmaması halinde risk faktörleri oluşmaktadır.

R8. Incidence Analysis & Forensics (Olay Analizi & Adli Bilişim): Bir güvenlik olayı olduğunda, bulut sağlayıcısında barındırılan uygulamalar ve hizmetler, günlük kaydı, bulunabilecek birden çok ana bilgisayar ve veri merkezi arasında dağıtılabileceğinden olay analizleri zorlaşmaktadır.
Bu dağıtımın çeşitli ülkelerde meydana gelmesi halinde farklı yasalara tabi olunduğu unutulmamalıdır. Ayrıca, günlük dosyalarıyla birlikte birden fazla müşteriye ait veriler aynı donanımda birlikte bulunabilir ve dolayısıyla adli kurtarma için kanun uygulayıcı kurumlar tarafında bir endişe teşkil edebilir.

R9. Infrastructure Security (Altyapı Güvenliği): Altyapı sağlamlaştırılmalı ve güvenli bir şekilde yapılandırılmalıdır. Uygulamalar, Sistemler ve Ağlar, katmanlama ve güvenlik bölgeleri ile mimarileştirilmeli ve yapılandırılmalıdır. Erişim, yalnızca gerekli ağ ve uygulama protokollerine izin verecek şekilde yapılandırılmalıdır. Yönetici erişimi rol tabanlı olmalı ve bilinmesi gerekenler temelinde verilmelidir. Düzenli risk değerlendirmeleri tercihen bağımsız bir tarafça yapılmalıdır. Yama/güvenlik güncellemeleri için bir politika ve süreç mevcut olmalıdır.

R10. Non-production Environment Exposure (Canlı Ortam Dışına Maruz Kalma): Yazılım uygulamaları geliştiren BT organizasyonu, tasarım, geliştirme ve test operasyonları için dahili olarak non-production ortamlar kullanır. Bu ortamlar genellikle production ortamıyla aynı ölçüde güvence altına alınmaz. Bir kuruluş, bu tür non-production ortamlar için bir bulut sağlayıcısı kullanıyorsa, bazı riskler söz konusu olabilmektedir.

Tüm bunların ışığında çıkarılabilecek sonuçlardan bir tanesi; var olan her sistemde veya yapıda güvenlik risklerinin var olabileceğidir. Sonuçta;

“Hiçbir sistem güvenli değildir.”😉

(Who Am I, 2014)

![]({{ 'assets/img/whoami.png' | relative_url }})

Reference:
[OWASP Cloud Top 10](https://owasp.org/www-pdf-archive/OWASP_Cloud_Top_10.pdf)
