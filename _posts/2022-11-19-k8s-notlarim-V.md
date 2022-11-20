---
title: Kubernetes Notlarım V- Kubernetes Ağ Altyapısı | Servisler
---

> Merhabalar, bu yazımda kubernetes ağ altyapısından bahsedecek ve servis objesine değineceğim. Keyifli okumalar..

Kubernetes üzerindeki podlarımız yani uygulamalarımız birbirleri ile ve dış dünya ile network üzerinden iletişim kurmazsa yaptıklarımız bir anlam ifade etmeyecektir. Sonuçta uygulamalarımızı Kubernetes üzerinde deploy ediyoruz ve kullanıcıların bu uygulamalara erişmesini istiyoruz.  Aynı şekilde uygulamalarımızın birbirleri ile ve dış dünya ile iletişim kurması gerekiyor.  

Bu yazımda kubernetes üzerinde iletişimin nasıl olduğunu ele alacak ve kubernetes'in ağ konusunu nasıl işlediğine değineceğim.

### Kubernetes Ağ Altyapısı
![]({{ 'assets/img/k8snetwork.PNG' | relative_url }})

Kubernetes ağ altyapısı yani networkü biraz zor ve ayrıntılı bir konudur. Fakat kubernetesi anlamak ve yönetmek noktasında kıymetlidir.  Burada mevcut bir kaç temel kural vardır:

* Kubernetes kurulumunda podlara ip dağıtılması için bir ip adres aralığı ya da kubernetes dilinde  **--pod-network-cidr** belirlenir.
* Kubernetesde her pod bu cidr bloğundan atanacak eşsiz bir ip adresine sahiptir.
* Aynı cluster içerisindeki tüm podlar varsayılan olarak birbirleriyle herhangi bir kısıtlama olmadan ve NAT yani network address translation olmadan haberleşebilirler.

Şimdi bu kurallar aklımızın bir köşesinde kala dursun, örnek bir cluster üzerinden konuyu inceleyelim.

![]({{ 'assets/img/network3.png' | relative_url }})

Diyelimki dört adet fiziksel sunucum var ve bu sunucularla bir kubernetes clusterı oluşturmak istiyorum. İnternet bağlantımı sağlayan bir router var, routerın iç networkü ise 192.168.1.0/24.  Routerın iç bacağı ise 192.168.1.1 ip adresine sahip.  Buraya bir switch bağladım, router ve dört fiziksel sunucuyu bu switche bağladım. Makinelere 192.168.1.0 networkünden bir ip adresi atadım ve hepsinin default gatewayi routerı görüyor.  Bu makineler aynı networkte ve dolayısıyla sıkıntısız haberleşebiliyorlar. Bu sunucuların hepsine linux dağıtımı bir işletim sistemi yüklediğimi de varsayalım.

Makinelerden bir tanesi master olarak diğer üçünü ise worker olarak kullanacağım.  Kubernetes kurulumunda podlara bir ip adres aralığı yani **pod-network-cidr** belirlenmesi gerektiğini yukarıda belirtmiştik. Dolayısıyla podlar için bir ip adres aralığı gerekiyor. Burada pod-network-cidr olarak **10.10.0.0/16** networkü set ettiğimizi düşünelim.  Kubernetes kurulumuna pod-network-cidr bilgisini girerek başladım, master node'u ayağa kaldırdım ve sonrasında worker nodeları clustera dahil ettim. 

Kurulum tamamlandığında her bir worker node üzerinde bir adet virtual bridge yaratılır. Bu virtual bridge bizim başlangıçta belirttiğimiz pod-network-cidr aralığında iletişim kurulacak şekilde ayarlanır. Yeni bir pod yaratıldığı zaman, o pod için linux kernelda ayrı bir network namespace oluşturulur ve podun sanal interfaceine pod-cidr aralığından bir ip adresi atanarak bulunduğu host üzerindeki bridge'e bağlanır. 

3 adet senaryo üzerinden buradaki trafiğin nasıl işlediğine bakalım.

##### Senaryo1: Podlar dış dünyadaki bir adrese gitmek isterse
Diyelimki pod-a dış dünyadan 1.1.1.1 ipsine sahip bir sunucu ile iletişim kurmak istesin. Bu durumda pod-a iletişim paketlerini bridge'e teslim edecek, paket burdan worker node'un eth0 interface'ine ulaşacak ve NAT lanacak. Sonrasında router'a gidecek burada yeniden NAT'lanacak oradan da 1.1.1.1'e ulaşmış olacak. Dolayısıyla pod-a üzerinde çalıştığı worker node'un ulaşabildiği her adrese ulaşabilecek. Bu senaryoda bir sorun yok.

##### Senaryo2: Pod-a aynı worker node üzerinde bulunan pod-b ile iletişim kurmak isterse
Her iki podda aynı worker node üzerinde bulunan aynı bridge'e bağlı ve aynı ip adres bloğundan ip adresine sahip. Dolayısıyla pod-a paketi bridge'e teslim edecek, bridgeden pod-b'ye gidecek. Aynı networkdeki iki makine gibi sorunsuz ve NATsız birbirleri ile konuşabilecekler. Bu senaryoda da bir sorun yok.

##### Senaryo3: Pod-a başka bir worker node üzerindeki pod ile iletişime geçmek isterse
Örneğin pod-c ile iletişime geçmek istesin. Başlangıçta podların birbirleriyle herhangi bir kısıtlama olmadan ve NAT olmadan haberleşebildiğinden bahsetmiştik. Peki bu kurala göre pod-a, pod-c ile nasıl NAT olmadan haberleşecek?  pod-a, pod-c ile iletişim kurmak istediğinde paketi bridge'e teslim edecek, bridge'de pod-c'nin ip adresi olmadığından trafiği eth0'a gönderecek, eth0 ip adresi kendi adres bloğu dışından olduğundan ve routing tanımlı olmadığından NAT'layıp router'a teslim edecek ve orada da paket kaybı olacak. Çünkü routerda bu paketi nereye göndereceği ile ilgili bilgi yok. Bu senaryoda ayrı worker nodelar üzerinde bulunan podlar birbirleri ile iletişim kuramıyorlar. Bu soruna çözüm bir kaç teknoloji var fakat bu çözümleri her network altyapısına uygun olacak şekilde kubernetes içerisinde sağlamak neredeyse imkansız.

Bu nedenle kubernetes kendi içinde bir network çözümü ile gelmez. Bunun yerine **Container Network Interface (CNI)** çözümüyle gelir. 

![]({{ 'assets/img/cni.png' | relative_url }})

**Cloud Native Computing Foundation** projesi olan **CNI**, Linux containerlarda ağ arabirimlerini yapılandırmak için eklentiler yazılabilmesini sağlayan spesifikasyonları belirler. CNI, container network altyapısının nasıl olması gerektiği ile standartlar yayınladı ve developerların bu standarda uygun networking çözümleri  oluşturabilmesine imkan sağladı. Böylece bu standartlara uygun network pluginleri yazılma imkanı doğdu. Kubernetes; CNI standardını benimsedi, kubernetes clusterlarda bu standarda uygun olarak yazılmış ve clusterın network çözümüne uygun olacak bir plugin seçilmesini söyledi. Burada sık tercih edilen, Calico gibi bir kaç CNI plugin mevcut.

Tüm bunlar yine aklımızın bir köşesinde kalsın, kubernetes kurulumunu tamamladıktan sonra bizim kullanacağımız CNI network plugin'i de yükleyerek kubernetes'in network yönetimini plugine bırakmasını sağlıyoruz. Örneğin bir pod oluştuktan sonra atanacak ip adresinin belirlenmesi, bunun poda atanması ve altyapıda ayarlanması gereken tüm işlerin tamamlanması CNI plugin'in görevidir. Böylece podlar farklı nodelar üzerinde olsalar da birbirleri ile NAT olmadan haberleşebilirler. Özetle Kubernetes'de podların birbirleri ile ve dış dünya ile iletişimleri yani **ingress trafiği** bu şekilde gerçekleşmektedir.

***Peki dış dünyadan bu podlara erişilmek istenirse?  😀
*
### Service Objesi
![]({{ 'assets/img/service.PNG' | relative_url }})

Servis objeleri, pod veya podlar üzerinde çalışan bir uygulamayı ağ hizmeti olarak göstermenin bir yoludur. 4 tipi vardır:
* **ClusterIP:** Servislerin birbirleri arasında iletişim kurabilmesi için kullanılan servis tipidir. ClusterIP tipinde bir servis objesi yaratıp bunu label selectorler aracılığıyla podlarla ilişkilendirebiliriz.  Bu obje yaratıldığı zaman cluster içerisindeki tüm podların ve kaynakların çözebileceği unique bir dns ismine sahip olur. Bunun yanında bizler her kubernetes kurulumunda **--service-cluster-ip-range** anahtarı ile bir sanal ip boğu belirleriz. Bir ClusterIP servis objesi yarattığımız zaman bu objeye bu ip bloğundan bir sanal ip adresi atanır ve ismiyle bu ip adresi clusterın dns mekanizmasında kaydedilir. Bu ip adresi virtual bir ip adrestir. kube-proxy tarafından tüm nodelardaki iptable'lara eklenir ve buraya ulaşan trafik servisin ilişkilendirildiği podlara yönlendirilir. Özetle; ClusterIP servisleri kubernetes altında servis discovery ve load balancing hizmetleri sunan obje tipidir.
* **NodePort:** Bu servis tipi ile node üzerinde dışarıdan erişilebilir bir port erişime açılır. Bir NodePort servisi yaratıp bunu label seectorler aracılığıyla podlarla eşleştirebilirsiniz. Bu obje yaratıldığında 30.000-32.762 aralığında bir port seçilir ve **tüm worker** nodelarda bu port sizin servisinize proxylenir. Kısacası worker node'un ip adresinin bu portuna gelen tüm paketler ilişkilendirdiğiniz podların publish ettiğiniz portuna yönlendirilir.
* **LoadBalancer:** Bu servis tipini sadece Azure Kubernetes Service ya da Google Kubernetes Engine gibi cloud servis sağlayıcıların managed kubernetes servislerinde kullanabiliriz. LoadBalancer tipinde servis objesi oluşturulduğunda cloud servis sağayıcı dış dünyadan erişilebilecek public bir ip adresine sahip bir loadbalancer oluşturur. Sonrasında bu loadbalancer'ı sizin servis objenizle ilişkilendirir. Dolayısıyla bu public ip adresine ulaşan paketler içeriden podlara yönlendirilir. 
* **ExternalName:** Bu servisler, selector ile değil bir dns adı ile eşleşirler. ExternalName tipinde yönlendirme işlemi bir proxying veya forwarding ile değil, DNS seviyesinde gerçekleşmektedir.

Servislerde diğer tüm kubernetes objeleri gibi yaml dosyaları ile oluşturulur. 

![]({{ 'assets/img/serviceyaml.PNG' | relative_url }})

Yukarıda örnek bir servis tanımı yer almakta. **kind** ile yine obje tipimizi belirtiyor ve **metadata** kısmında objeye ait bilgileri giriyoruz. Buradaki **name** kısmı önemli, isim artık cluster içerisinden bu servise ulaşacağımız isim olacak. **spec** kısmında bu servisin **ClusterIP** tipinde bir servis olduğunu, **selector** kısmında ise  **app: backend** anahtar veri eşleniğine sahip olan podlara trafiği yönlendirmesini söylüyoruz. Yani servis hangi podlara hizmet edeceğini selector ile seçmiş oluyor. 

**Ports** kısmında ise; **port: 5000** ile servisin dinleyeceği portu, **targetPort: 5000** ile de podun expose ettiği portu belirtiyoruz.

`kubectl apply -f service.yaml` komutu ile servis objesini apply edebilirsiniz.

`kubectl get services` komutu ile de servislerinizi listeleyebilirsiniz. Bu çıktıda yer alan servise ait Cluster-Ip bilgisi yalnızca cluster içerisinde geçerlidir. Cluster içerisinde herhangi bir kaynak bu ip'nin ilgili portuna istek gönderdiğinde bu istek bu servisin selector ile seçtiği podlardan bir tanesine yönlendirilecektir.

![]({{ 'assets/img/nodeport.PNG' | relative_url }})

Yukarıda ise NodePort servis tipinde bir servis obje tanımı yer almakta. Bu da aslında ClusterIP tipi gibi cluster içinden bu servisin 80 portuna gidersek arkadaki podların 80 portuna servis yönlendirilecek ama bunun yanında ek bir işlem daha var. **Tüm worker nodeların dış bacağında ilgili portu dinlemeye başlıyor.** Biz herhangi bir podun bu portuna ulaşırsak servis podlarından birine trafik yönlenecektir. 

Aşağıda ise LoadBalancer tipinde bir servis objesi yer almakta fakat bunu localde oluşturmak çok bir anlam ifade etmeyecektir. Yukarıda da bahsettiğimiz gibi bunu bir cloud servis sağlayıcıdan kullanıdığımızda anlamlı olacak servis tipidir. 

![]({{ 'assets/img/lbservice.PNG' | relative_url }})

Imperative yöntemle de servis objesi oluşturabiliriz. 

`kubectl expose deployment myapp --type=ClusterIP --name=myservice` --> kubectl expose komutu ile expose etmek istediğimiz objeyi ve adını belirtiyor sonrasında servis tipini ve adını belirliyoruz. 

Deployment konusunu ele alırken podları deploymentın değil ReplicaSet objesinin oluşturduğunu söylemiştik. Deployment replicaSet oluşturur, replicaSet podları oluşturur. Benzer bir durum service objesi için de geçerlidir. 

Bir servis oluşturduğumuz zaman bu servis **endpoint** adında bir obje oluşturur. Bu endpoint objesi, bizim belirlediğimiz selector tarafından seçilen podların ip adreslerini üzerinde barındırır. Service trafiğini nereye yönlendireceğini bu listeye göre düzenler. 

`kubectl get endpoints` ile oluşturduğunuz bütün servisler ile aynı isimle bir endpoint objesi oluştuğunu görebilirsiniz. 

Diyelimm ve konuyu burada noktalayalım. Bir sonraki yazıda görüşmek üzere..


Bir gelenek olsun, star wars temalı fotoğrafımızı da koyalım😉

![]({{ 'assets/img/yodaservice.jpg' | relative_url }})
