---
title: Golden Ticket, Silver Ticket ve Kerberoasting Nedir?
---

Golden Ticket, Silver Ticket ve Kerberoasting kavramlarından bahsetmeden önce Kerboros' un nasıl çalıştığına değinmek istiyorum.  **Kerberos**, domain ortamında **bilet (ticket)**  mantığını kullanarak kimlik doğrulama ve yetkilendirmeyi sağlayan protokoldür. UDP 88 portundan hizmet veren Kerberos üzerinden bir çok saldırı gerçekleştirilmektedir. Bu saldırıları anlamak için öncelikle Kerberos' un nasıl çalıştığını anlamak önemlidir.

![Kerberos](http://www.koraykey.com/wp-content/uploads/Kerberos_Install.jpg)

### Kerberos mekanizmasında 3 adet aktör bulunmaktadır:

**İstemci (client)**

**Sunucu (server)**

**Etki alanı kontrolcüsü (Domain Contoller)**


* İstemci kimliğini doğrulatmak istemekte, sunucu istemcinin talep ettiği servisi sağlamakta, DC ise hem istemci hemde sunucu için güvenilir iletişimi sağlamaktadır.
* Kullanıcının logon bilgilerini (kullanıcı adı, parola) girmesiyle başlayan bir senaryo üzerinden anlatacak olursak client makine girilen bu bilgileri şifrelemekte ve DC' ye iletmektedir (aslında DC' ye şifrelenmiş bir şekilde iletilen yalnızca kullanıcı adı bilgisidir). 
* Active Directory Domain Controller' ın bir bileşeni olan Anahtar Dağıtım Merkezi (Key Distribution Center -kısaca KDC diyeceğiz), client tarafından gönderilen şifrelenmiş bilgi ile kendi veritabanında bulunan bilgiyi karşılaştırır. Bilgileri doğrulayan KDC, identification işlemini tamamlamış olur. 
* Bir kullanıcı logon olacak ise mutlaka grup üyelikleri DC tarafından sorgulanır. Kullanıcının üyesi olduğu tüm group blgileri  global katalog üzerinden sorgulanır ve listelenir.

**Domain ortamında ilk kurulan DC global katolog görevini üstlenir. Global katalog bir veya birden fazla DC' nin görevi olabilir. 
Bu DC, domain ortamında tüm bilgileri üzerinde tutar. Universal grup üyelikleri de yalnızca global katalogda bulunur.**

* KDC, iletişimin güvenli sağlanması için bir session key ve ticket granting ticket (bilet sağlayan bilet - TGT) üretir.

**TGT, network' te bir kaynağa erişmek istendiğinde ihtiyaç duyulan ticket'ı alabilmek için kullanılmaktadır.**
* KDC, bu bilgileri şifreler ve client' a gönderir böylece domaine logon olma işlemi bitmiş olur. Client artık var olan session key ve tgt bilgileri ile bir servis bileti alabilir, domain kaynaklarına erişebilir ve KDC ile güvenli iletişim kurabilir.
* Yine domaine üye (aynı domaine) servis sağlayan bir sunucuya erişmek isteyen client' ın olduğu senaryo üzerinden gidecek olursak; client elindeki TGT ile DC üzerindeki TGS (Ticket Granting Service)' ye erişir ve TGS, client' ın sunucuya erişebilmesi için bir çift "session ticket" verir. Session ticket ile client sunucuya erişim sağlar. Servisi sağlayan sunucunun üzerinde çalışan LSA (local security authority) serivisi, session ticket' ı kullanarak bir "access token" yaratır. LSA serivisi bir dizi erişim denetim listeleri vasıtasıyla access token üzerindeki "sid" (kullanıcının dahil olduğu grup ve kullanıcı haklarını gösterir) bilgilerini karşılaştırır. Son olarak; kullanıcının hangi hak ve yetkilerle kaynağa erişim yapması gerektiği belirlenir.

Kerberos mekanizmasından bahsettiğimize göre Pass the Ticket saldırılarına geçebiliriz. PTT saldırıları, Kerberos protokolünün açıklığını kullanarak gerçekleştirilir.  NTLM ve LM' deki sıkıntılar dolayısıyla geliştirilen bu protokoldede saldırganlar açısından farklı saldırı vektörleri mevcut. 

Öncelikle Golden Ticket veya Silver Ticket oluşturulabilmesi için domain parola hash'lerinin tamamının çalınması gererektiğini bilmemiz gerekir. Tabi bizim bu noktada özellikle ilgilendiğimiz "KRBTGT" kullanıcısının parola hash' idir.

> KRBTGT: Bir etki alanı oluştuğunda varsayılan olarak bu kullanıcıda oluşur. Fakat günlük operasyonlarda genlde kullanılmadığından çoğu sistemcinin böyle bir kullanıcının varlığından haberi yoktur.

Domain parola hashlerini elde etmemiz gerektiğinden bahsetmiştik. Peki bu bilgiler nasıl elde edilebilir? 

#### Domain Parola Hash' lerini Elde Etme Yolları
- "vssadmin" aracı ile Volume Shadow Copy almak: Normalde işletim sistemleri NTDS.dit dosyasına  (domain kullanıcılarının parola hashlerini de içinde barındıran domain veri yapılarını içinde saklayan dosya) erişimi engeller. Kopyasının alınması suretiyle offline olarak erişmiş olup ele geçirilebilir.
- PoweSploit NinjaCopy powershell scripti gibi benzer araçlar ve scriptler ile diske raw formatta erişmek: Dosya sistemlerinin hiçbiri ile biçimlendirilmemiş diskin ham hali olan raw formatta diski görüp NTDS.dit dosyası ele geçirilebilir.
- Mimikatz'in DCSync (Domain Controller' ların kendi aralarındaki yedekleme mekanizması) özelliğini kullanmak: Bu yöntemde Mimikatz gibi başka araçlarda kullanılmaktadır. Saldırgan Backup Domain Controller gibi gibi davranarak birincil domain controller' daki domain verilerini kendi üzerine çekebilir.

**Bu işlemler sıradan kullanıcının gerçekleştirebileceği işlemler değildir. Domain Admin kullanıcılarından birisinin parolasının çalınmş olması gereklidir.**
