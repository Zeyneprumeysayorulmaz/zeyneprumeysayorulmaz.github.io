---
title: TryHackMe Attacking Kerberos - WriteUp
---

Bu yazımda TryHackMe platformunda bulunan Attacking Kerberos adlı odanın çözümünden bahsedeceğim. Kerberos' a saldırmanın tüm temellerini içerien bu odada;

* Kerbrute ve Rubeus gibi araçları kullanarak enumeration
* Kerberoasting
* Rubeus ve Impacket ile AS-REP Roasting
* Golden/Silver Ticket Saldırıları
* Pass the Ticket
* Mimikatz kullanarak Skeleton key attacks

konularına yer verilmektedir.
Odayı tamamlayabilmek için temel düzeyde Active Directory, Windows komut satırı, Kerberos ve beraberinde Kerberos' a yönelik saldırı senaryolarına hakim olmak gerekir. Bir önceki yazıda Kerberos mekanizmasından ve Kerberos' a yönelik saldırı tekniklerinden bahsetmiştik. Yazıya [buradan](https://zeyneprumeysayorulmaz.github.io/Pass-the-ticket/) ulaşabilirsiniz.

Yapılması gereken 9 adet görev bulunmakta, sırasıyla görevleri tamamlayalım.

## **Task 1: Introduction** 

What does TGT stand for?

>Ticket Granting Ticket

What does SPN stand for?

> Service Principal Name

What does PAC stand for?

> Privilege Attribute Certificate

What two services make up the KDC?

> AS, TGS

## **Task 2:  Enumeration w/ Kerbrute**

Bu bölümde Kerbrute aracını kullanacağız. Kerbrute, Kerberos ön kimlik doğrulamasını kötüye kullanarak geçerli aktif dizin kullanıcılarını brute force uygulamak ve enumeration için kullanılan popüler bir araçtır. Bölüme geçmeden önce saldırgan makinede host dosyasına makinenin IP adresi ile DNS etki alanı adını ekleyelim aksi halde saldırıları gerçekleştiremeyiz.

2. numaralı resmi eklee

Kerbrute aracını talimatlarda da verilmiş olan [bu adresten](https://github.com/ropnop/kerbrute/releases) saldırgan makinemize indirebilirsiniz. Talimatları takip ederek dosyayı yeniden adlandıralım ve çalıştırma yetkisi verelim.

3. resmi ekleeee

Kullanıcılar üzerine gerçekleştirilen enumeration, hedef etki alanında hangi kullanıcı hesaplarının olduğunu ve ağa erişmek için hangi hesapların potansiyel olarak kullanılabileceğini bilmemizi sağlar. Bu işleme geçmeden önce talimatta verilen wordlisti makinemize [bu adresten](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/User.txt) indirelim.

4. resimmm

`./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt` komutu ile User.txt wordlistini kullanarak etki alanı denetleyicisinde var olan kullanıcı hesaplarına brute force gerçekleştirelim. Bu işlemden sonra tüm soruların yanıtlarını elde edeceğiz.

5. resiimmm

Şimdi soruları yanıtlayabiliriz.

How many total users do we enumerate?

> 10

What is the SQL service account name?

> SQLService

What is the second "machine" account name?

> Machine2

What is the third "user" account name?

> User3

## **Task 3: Harvesting & Brute-Forcing Tickets w/ Rubeus**

Bu görevi başlatmak için makineye RDP veya SSH yapmamız gerekmekte. Bunun için gerekli bilgiler aşağıdaki gibi verilmekte.

- Username: Administrator 

- Password: P@$$W0rd 

- Domain: controller.local

`ssh Administrator@<hedef-makine-ip>` komutu ile ssh yapabiliriz.

Rubeus, Kerberos'a saldırmak için güçlü bir araç ve kekeo aracının bir uyarlamasıdır. Rubeus, Kerberos'a saldırmak için çok yönlü bir araç olmasını sağlayan çok çeşitli saldırılara ve özelliklere sahiptir. Hash'i aşma, bilet talepleri ve yenilemeleri, bilet yönetimi, bilet çıkarma, Harvesting etme, bileti geçme, AS-REP Roasting ve Kerberoasting gibi yöntemlere sahiptir. Rubeus'un zaten hadef makinede var olduğunu söyleniyor.

### **Harvesting Tickets w/ Rubeus**
Harvesting, KDC'ye aktarılan biletleri toplar ve onları bilet geçişi saldırısı gibi diğer saldırılarda kullanılmak üzere kaydeder. Şimdi Rubeus ile TGT'leri toplayalım ve Rubeus bizim için bu bilgileri kaydetsin.

8.1 i eklee

### **Brute-Forcing / Password-Spraying w/ Rubeus -**
Rubeus, hem brute force ile hem de password spraying ile  kullanıcı hesaplarını kullanabilir. Hesaplara brute force uygularken, belirli bir kullanıcı hesabı için hangi parolanın işe yaradığını görmek üzere bir parola listesi (wordlist) kullanırsınız. Password spraying de ise Parola1 gibi tek bir parola verirsiniz ve domainde bulunan tüm kullanıcı hesaplarına "sprey" yaparak o parolanın hangisinde olabileceğini bulursunuz.

Bu saldırıda, Kerberos tabanlı belirli bir şifreyi kullanacak, bulunan tüm kullanıcılara püskürtecek ve bir .kirbi bileti verecektir. Bu bilet, KDC'den servis bileti almak için kullanılabilecek bir TGT'dir ve Pass the Ticket gibi saldırılarda kullanılabilecektir.

Rubeus ile parola püskürtmeden önce, Windows ana bilgisayar dosyasına etki alanı denetleyicisi etki alanı adını eklemeniz gerekir. Echo komutunu kullanarak makineden ana bilgisayar dosyasına IP ve etki alanı adını ekleyebilirsiniz:

`echo MACHINE_IP CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts`

Daha sonra Rubeus' un bulunduğu dizine gidelim ve aşağıdaki gibi Rubeus' a püskürtmesi için bir parola verelim. Ardından Rubeus bize belirtilen kullanıcı için .kirbi TGT'yi verecektir. Soruları yanıtlarına ise bu işlemden sonra ulaşacağız.

8. resimmmm

Which domain admin do we get a ticket for when harvesting tickets?

> Administrator

Which domain controller do we get a ticket for when harvesting tickets?

> CONTROLLER-1

## **Task 4: Kerberoasting w/ Rubeus & Impacket**

Bu görevde  Kerberoasting'i ele alacağız. Kerberoasting, kullanıcının kayıtlı bir SPN'ye sahip herhangi bir hizmet için bir hizmet bileti talep etmesine ve ardından bu bileti hizmet parolasını kırmak için kullanmasına olanak tanır. Hizmetin kayıtlı bir SPN'si varsa Kerberoastable olabilir, ancak saldırının başarısı parolanın ne kadar güçlü olduğuna ve izlenebilir olup olmadığına ve kırılan hizmet hesabının ayrıcalıklarına bağlıdır.

Saldırıyı gerçekleştirmek için hem Rubeus hem de Impacket kullanacağız. Bu saldırı için kullanılabilecek Rubeus, Impacket, kekeo, Invoke-Kerberoast gibi bir çok araç mevcuttur.
Rubeus zaten hedef makinede Downloads dizininde mevcut. Bu görevde hem Rubeus ile hem de Impacket ile Kerberoasting saldırısını gerçekleştireceğiz.

#### Method 1 - Rubeus

Hizmetin kayıtlı bir SPN'i varsa Kerberoastable olduğunu söylemiştik. Aşağıdaki gibi `Rubeus.exe kerberoast` komutunu çalıştıralım ve SQL ve HTTP servis hesaplarının hash'lerini elde edelim.

9. resimm
9.1. resimm 

Bu hash'leri kopyalayıp saldırgan makinemizde hashcat, John the Ripper vb. araçlarla kırabiliriz.Talimatlarda hash'i kolay kırabilmek adına bir wordlist verilmiş. [Buradan](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/Pass.txt) indirebilirsiniz.

9.2. resimm

Bu metodu kullanabilmek için Rubeus.exe' nin hedef makinede var olması gerekmekte fakat Impacket da böyle bir koşul yok. Bu işlemi uzaktan bir kimlik bilgisini kullanarak gerçekleştirebiliriz.

#### Method 2 - Impacket

Impacket'ı indirmek için /opt dizinine geçelim ve paketi [indirelim](https://github.com/SecureAuthCorp/impacket/releases/tag/impacket_0_9_19). Impacket dizinine gidip gerekli bağımlılıkları indirelim.

10. resimm 

GetUserSPNs.py'nin bulunduğu dizine gidelim ve `sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip MACHINE_IP -request` komutunu çalıştıralım. Bu komut hedef etki alanında bulabileceği tüm kerberoastable hesaplar için Kerberos hash'lerini verecektir. Tıpkı Rubeus da olduğu gibi fakat Rubeus daki gibi hedef makinede olması gerekmez. Böylece SQL ve HTTP servis hesaplarının hash'leri elde edilir.

11. resimmm

Daha sonra hash'leri ayrı ayrı bir txt dosyasına kaydedip dilediğiniz araçla kırabilrisiniz.

13. 14. resimmm

What is the HTTPService Password?

> Summer2020

What is the SQLService Password?

> MYPassword123#

## **Task 5: AS-REP Roasting w/ Rubeus**

Kerberoasting'e çok benzeyen AS-REP Roasting, Kerberos ön kimlik doğrulamasının devre dışı bırakıldığı kullanıcı hesaplarının krbasrep5 hash'lerini verir. Kerberoasting'den farklı olarak, bu kullanıcıların hizmet hesapları olması gerekmez.

Rubeus'un AS-REP Roasting ve Kerberos ön kimlik doğrulaması devre dışı bırakılmış kullanıcılara saldırma için çok basit ve anlaşılması kolay bir komutu olduğundan, Rubeus'u kerberoasting ve harvesting ile aynı şekilde kullanmaya devam edeceğiz. Rubeus'tan hash'i elde ettikten sonra, krbasrep5 hash'ini kırmak için hashcat kullanacağız.

AS-REP Roasting için kekeo ve Impacket's GetNPUsers.py gibi başka araçlar da vardır. Rubeus'un kullanımı daha kolaydır çünkü AS-REP Roastable kullanıcılarını otomatik olarak bulurken GetNPUsers ile kullanıcıları önceden enumeration gerçekleştirmek ve hangi kullanıcıların AS-REP Roastable olabileceğini bilmeniz gerekir.

Rubeus'un hedef makinede var olduğunu biliyoruz. Rubeus'un bulunduğu dizine gidelim. `Rubeus.exe asreproast` komutu ile güvenlik açığı bulunan kullanıcıları arayan AS-REP Roasting komutu çalışacak ve ardından güvenlik açığı bulunan kullanıcı Hash'leri elde edilecektir.

15. resiimmm

Daha sonra elde ettiğimiz bu hash'leri hedef makineden saldırgan makinemize kopyalayalım ve bir txt dosyasına yazalım.(Talimatlarda belirtildiği hash üzerinde küçük bir oynama gerekli. İlk satır `$krb5asrep$23$User` olacak şekilde `$krb5asrep$`'dan sonra `23$`ekleyelim.) Görev 4'teki wordlisti kullanarak `hashcat -m 18200 hash.txt Pass.txt` hash'leri kırabiliriz.

16. 17. resimleeeer

### **ÖNEMLİ !!**
- Güçlü bir parola politikası olmalı. Güçlü bir parola ile, hash'lerin kırılması daha uzun sürer ve bu saldırıyı daha az etkili hale getirir.

- Gerekli olmadıkça Kerberos Ön Kimlik Doğrulamasını kapatmayın, bu saldırıyı tamamen ortadan kaldırmanın Ön Kimlik Doğrulamayı açık tutmak dışında neredeyse başka bir yolu yoktur.

What hash type does AS-REP Roasting use? 

> Kerberos 5 AS-REP etype 23

Which User is vulnerable to AS-REP Roasting?

> User3

What is the User's Password?

> Password3

Which Admin is vulnerable to AS-REP Roasting?

> Admin2

What is the Admin's Password?

> P@$$W0rd2

## **Task 6:  Pass the Ticket w/ mimikatz** 

> No answer needed

Mimikatz, bir aktif dizin ağının içinden kullanıcı kimlik bilgilerini elde etmeye yarayan ve sık kullanılan sömürü sonrası araçtır. LSASS belleğinden bir TGT'yi elde etmek için mimikatz'ı kullanmak faydalı olacaktır.

Pass the Ticket'a Genel Bakış  -

TGT'nin, makinenin LSASS hafızasından dump edilmesiyle **Pass the Ticket** gerçekleştirilir. Yerel Güvenlik Yetkilisi Alt Sistem Hizmeti (LSASS), kimlik bilgilerini bir aktif dizin sunucusunda depolayan ve Kerberos biletini diğer kimlik bilgisi türleri ile birlikte ağ geçidi bekçisi olarak hareket etmek ve sağlanan kimlik bilgilerini kabul etmek veya reddetmek için depolayabilen bir bellek işlemidir. Kerberos Biletlerini LSASS belleğinden tıpkı hash'leri elde ettiğimiz gibi dump edebiliriz. Biletleri mimikatz ile dump ettiğimizde, LSASS hafızasında bir alan adı yöneticisi bileti varsa, alan adı yöneticisi kazanmak için kullanılabilecek bir .kirbi bileti verecektir. Bu saldırı, etrafta güvenli olmayan etki alanı hizmeti hesabı biletleri varsa, privilege escalation ve lateral movement için mükemmeldir. Saldırıda; bir etki alanı yöneticisinin biletini elde edersiniz ve ardından o etki alanı yöneticisi olarak hareket etmenize izin veren mimikatz, PTT saldırısını kullanarak bu biletin kimliğine bürünürsünüz, böylece etki alanı yöneticisine yükselmenize olanak tanınır. Bilet saldırısını, var olan bir bileti yeniden kullanmak, burada herhangi bir bilet oluşturmak veya yok etmek değil, etki alanındaki başka bir kullanıcıdan gelen mevcut bir bileti yeniden kullanmak ve o bileti taklit etmek gibi bir geçiş düşünebilirsiniz.

Mimikatz hedef makinede Downloads dizininde yer almakta, dizine gidip mimikatz'i çalıştıralım. `privilege::debug` komutunu yürütelim ve mimikatz'in doğru çalıştığından emin olalım. Çıktının `'20' OK`  olması gerekiyor, aksi takdirde düzgün çalışmıyordur.

18. resimm

`sekurlsa::tickets /export` komutu ile tüm .kirbi biletlerini şu anda bulunduğunuz dizine aktaralım.

`kerberos::ptt <ticket>` komutu ile elde edilen biletleri mimikatz içinde çalıştırablım. Verilen bileti önbelleğe almakta ve taklit etmektedir.

`klist` komutu önbelleğe alınmış biletleri listelemekte ve bu işlemler doğrulamaktadır.

**ÖNEMLİ !!** Etki alanı yöneticilerinizin etki alanı denetleyicisi dışında herhangi bir şeye oturum açmasına izin vermeyin. Bu çok basit bir önlem olabilir, ancak birçok etki alanı yöneticisi hala düşük seviyeli bilgisayarlarda oturum açar ve etrafta saldırmak ve yanal olarak hareket etmek için kullanabileceğimiz biletler bırakır.

Pass the Ticket Saldırılarını anladığımıza göre Task 7'ye geçebiliriz.

## **Task 7: Golden/Silver Ticket Attacks w/ mimikatz** 

` lsadump::lsa /inject /name:krbtgt` komutu ile krbtgt kullanıcısının hash'ini dump edebiliriz.

Silver Ticket oluşturmak için, bir etki alanı yöneticisi hesabının veya SQLService hesabı gibi bir hizmet hesabının hash'ini dump için /name: öğesini değiştirmek gerekmektedir.

23. resiiimm

`Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:` komutu Silver Ticket oluşturmak için Golden Ticket oluşturma komutudur. sid, krbtgt ve id bilgilerini çıktıya uygun olarak değiştirin. 

`misc::cmd` mimikatz'de verilen biletle daha ayrıcalıklyüksek yetkili bir komut istemi açacaktır.

24. resiimm

Erişim yetkisi ve erişebileceğiniz kaynaklar, bileti almaya karar verdiğiniz kullanıcının ayrıcalıklarına bağlı olacaktır. Eğer bileti krbtgt'den aldıysanız, TÜM ağa erişiminiz vardır (**Golden Ticket**).  **Silver Ticket**'ta yalnızca o kullanıcının erişebildiği kişilere erişimi vardır, eğer bir etki alanı yöneticisiyse neredeyse tüm ağa erişebilir, ancak Golden Ticket'tan biraz daha az yükselir.

Soruları yanıtlayacak olursak;

SQL servisine ait NTLm hash'ini elde etmek için `lsadump::lsa /inject /name:sqlservice` komutunu çalıştırmamız yeterli olacaktır.

25. resimm

Administrator' un hash'ini elde etmek içinse `lsadump::lsa /inject /name:Administrator` komutunu yürütmek yeterlidir.

26. resiiimmm

What is the SQLService NTLM Hash?

> cd40c9ed96265531b21fc5b1dafcfb0a

What is the Administrator NTLM Hash?

> 2777b7fec870e04dda00cd7260f7bee6

## **Task 8: Kerberos Backdoors w/ mimikatz**

> No answer needed

Mimikatz, Golden ve Silver Ticket kullanarak erişimi sürdürmenin yanı sıra, Kerberos Backdoors özelliğine sahiptir ve bu çok daha inceliklidir. Çünkü kendisini etki alanı ormanının belleğine yerleştirerek bir ana parola ile makinelerden herhangi birine erişmesine izin vererek bir kök kullanıcı setine benzer şekilde davranır.

Kerberos Backdoors, AS-REQ'nun şifrelenmiş zaman damgalarını doğrulama şeklini kötüye kullanan bir Skeleton Key yerleştirerek çalışır. Bir Skeleton Key yalnızca Kerberos RC4 şifrelemesi kullanılarak çalışır.

Bir mimikatz Skeleton Key'in varsayılan karma değeri 60BA4FCADC466C7A033C178194C03DF6'dır ve bu da parolayı -"mimikatz" yapar.

Skeleton Key'e Genel Bakış -

Skeleton Key, yukarıda söylediğim gibi AS-REQ şifreli zaman damgalarını kötüye kullanarak çalışır, zaman damgası, kullanıcıların NT hash'i ile şifrelenir. Etki alanı denetleyicisi daha sonra, bir iskelet anahtarı implante edildiğinde, etki alanı denetleyicisi, etki alanı ormanına erişmenizi sağlayan hem kullanıcı NT karma değerini hem de iskelet anahtar NT karma değerini kullanarak zaman damgasının şifresini çözmeye çalışır.

#### Installing the Skeleton Key w/ mimikatz -

Mimikatz'i çalıştırdıktan sonra `misc::skeleton` komutunu yürütmeniz yeterlidir.

#### Forest' a erişim -

Varsayılan kimlik bilgileri "mimikatz" dir.

**Örnek:** `net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz` --> Böylece Paylaşım Yönetici parolasına gerek kalmadan erişilebilir olacaktır.

**Örnek:** `dir \\Desktop-1\c$ /user:Machine1 mimikatz` --> Kullanıcıların Desktop-1'e ne erişimi olduğunu bilmeden Desktop-1 dizinine erişmek mümkündür.

Skeleton Key, bellekte çalıştığı için kendi kendine kalıcı olmaz, diğer araçlar ve teknikler kullanılarak komut dosyası yazılabilir veya kalıcı hale getirilebilir.

## **Task 9: Conclusion**

> No answer needed
