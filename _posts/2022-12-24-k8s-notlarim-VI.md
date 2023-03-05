---
title: Kubernetes Notlarım VI- Liveness ve Readiness Probes
---

> Merhabalar Kubernetes yazı serisine devam ediyorum. Bu yazımda Liveness ve Readiness probes konularını ele alacağım. Keyifli okumalar..

![]({{ 'assets/img/livenessreadiness.png' | relative_url }})

Serinin önceki yazılarında pod yaşam döngüsüne sıkça değinmiştik. Bir pod oluştuğu zaman bu podun çalıştığı node üzerindeki kubelet servisi containerlarını oluşturur ve containerları gözlemeye başlar. Daha doğrusu container engine ile haberleşerek ona talimat verir ve containerlar oluşturulur. Sonrasında containerlar izlenmeye başlanır. Eğer restart policy always ya da onFailru seçili ise de container içerisinde çalışan uygulama bir şekilde durur ve cevap veremez hale gelirse container restart edilir.  Yeniden başlar ve sorun çözülmüş olur. Temel mekanizma bu şekildedir. Fakat bazı durumlarda container içerisinde başlatılan uygulama process olarak çalışıyor olsada aslında yapması gereken işlemi yapmıyor olabilir. Uygulama çalışmıyor olsa yani uygulama kapansa kubelet bunu tespit edecek ve containerı yeniden oluşturarak sorunu çözecektir. Fakat az önce de söylediğimiz gibi uygulama ayakta olmasına rağmen yapması gereken işlemi yapmıyorsa kubelet bunu tespit edemez, dolayısıyla containerı yeniden başlatarak sorunu çözme mekanizmasını çalıştırmaz. Kubernetesde buna çözüm olarak **Liveness Probe** özelliği sunulmaktadır.

### **Liveness Probes**

Liveness probelar ile bizler container içerisinde komutlar çalıştırarak ya da bir endpointe http request göndererek ya da bir porta tcp connection açarak uygulamamızın doğru çalışıp çalışmadığını kontrol edebiliyoruz. Bu sorgulamanın sonucunda beklediğimiz cevabı alıyorsak containerın sağlıklı olduğunu aksi halde bir sıkıntı olduğunu tespit edebiliyoruz. Kubelet bunun sonucuna göre de aksiyon alabiliyor.

Örneğin aşağıdaki pod obje tanımını inceleyelim.

![]({{ 'assets/img/livenessprobe.PNG' | relative_url }})

Yukarıdaki obje tanımı liveness-http isimli bir pod objesi oluşturacaktır. Podun içerisinde bir container çalışacak ve bu containerda google container registryden liveness isimli imajla oluşacaktır. LivenessProbe anahtarı ile liveness tanımı başlatılmış httpGet probe'u kullanılmıştır.
Kullanabileceğimiz 3 tip probe var:
* **HttpGet:** /healthz isimli endpointe 8080 portundan http get request gönderir. Cevap alabilirse (return code: 200-400 arası) uygulama sağlıklı çalışıyor demektir eğer 400 ve üstü bir cevap dönerse bir hata olduğu anlaşılmaktadır.

Yukarıdaki pod objesi oluştuğunda container çalışmaya başlayacak ve /server isimli uygulamayı çalıştıracaktır. Uygulama crash olursa kubelet bunu restart edecektir. Fakat uygulama çalışmaya devam ettiği halde bir sorun varsa kubelet bunu algılayamayacaktır.
Bu noktada liveness probe devreye giriyor. Liveness probe ile localhost/healthz endpointine httpGet isteği gönderilir, uygulama düzgün çalışıyorsa http 200 sonucu dönecektir, eğer servis ayakta olmasına rağmen bir aksilik varsa bu isteğin sonucu http 400 dönecektir. kubelet bunu algılayacak ve container restart ederek bu sorunu çözmeye çalışacaktır. Yukarıdaki initialDelaySeconds ve periodSeconds anahtarlarına değinelim; uygulama container çalıştıktan hemen sonra hizmet vermiyor olabilir bu durumda livenessProbe hemen başlarsa daha uygulama ayağa kalkmadan  hata alınacaktır. Bunu önlemek adına initialDelaySeconds parametresi ile liveness probe'a bu süre kadar bekle diyoruz. periodSeconds değeri de bu işin kaç saniyede bir yapılacağını belirtir.

Uygulama eğer bir web uygulaması ise bunun kontrolünü http sorgularıyla anlayabilir ve HttpProbe kullanabiliriz. Fakat http tabanlı olmayan bir uygulama ise burada httpProbe kullanamayız.  Bunun yerine bir script ile uygulamaya uygun sağlık kontrolü için sorgular yapılabilir.  Exec probe bize bu imkani verir.

* **Exec:** exec ile shell'de komut ya da uygulama çalıştırabiliriz. Bu uygulama olumlu sonuç verirse sağlıklı olduğunu anlarız, onun dışında bir cevap verirse de bir sorun olduğunu anlar ve container'ı restart ederiz.

![]({{ 'assets/img/execlivenessprobe.PNG' | relative_url }})

Örneğin yukarıdaki exec probe tanımını inceleyelim. Bu  örnekte exec probe ile komut çalıştırıyoruz. cat komutuyla /tmp dizininde healthy isimli bir dosyanın içeriğini kontrol ediyoruz. Eğer /tmp dizininde böyle bir dosya mevcutsa komut sonucu 0 dönecek ve olumlu olacaktır eğer böyle bir dosya yoksa exit code 1 olacaktır.

* **TCP Socket:**  Bazı durumlarda http get istekleri de komut çalıştırmak da uygulamanın sağlıklı olup olmadığını anlamamıza yardımcı olamayabilir. Örneğin MySql DB container'ı çalışıyor olsun. MySql process'i 3306 portundan erişilebilir, tcpSocket ile bu porta bir istek gönderip eğer cevap alınırsa uygulamanın düzgün çalıştığı teyit edilebilir. Olumsuz bir cevap alınırsa örneğin connection refused dönerse container kapatılacak ve restart edilecektir.

![]({{ 'assets/img/tcpsocketlivenessprobe.PNG' | relative_url }})
Liveness Probe'lar ile uygulamaya uygun bir probe seçilerek uygulamanın hatasız çalışıp çalışmadığı kontrol edilebilir. Uygulamanın çalışma mantığına uygun hataların tespit edilebileceği Liveness Probe tanımları yapılmalıdır.

### **Readiness Probes**

Tıpkı liveness Probe'lar gibi oluşturulur. httpGet, exec ya da tcpSocket olarak kurgulanabilir.

Kubelet, bir container'ın ne zaman trafiği kabul etmeye hazır olduğunu bilmek için readiness probeları kullanır. Bir pod, tüm containerlar hazır olduğunda hazır kabul edilir. Readiness probe sayesinde bir pod hazır olana kadar service arkasına eklenmez.

Örneğin 3 replica pod çalışan bir frontend servisimiz var. Bu senaryoda LoadBalancer tipinde bir servis objem ve arkasında 3 adet pod yer almakta. Bu frontend podları için deploymentta bir imaj değişikliği yapmam gerekti ve deployment rollingUpdate stratejsine göre çalışıyor. Böyle bir değişiklikte podlar sırasıyla silinecek ve yeni imajla ayağa kalkacaktır. Fakat frontend uygulaması pod ayağa kalktığı anda hazır olmuyorsa dışarıdan bir adresten bir dosya vb bir şey okuyor ve ihtiyaç duyuyorsa pod ayağa kalktığı anda uygulama hazır olmayabilir. Hazır olmayan bu uygulamayı direkt LoadBalancer servisinin arkasına almak istemeyiz. Bu durumda readiness probe kullanırsak uygulamanın yapısına göre gerekli kontroller yapıldığında ve sonuç olumlu olduğunda podları servis arkasına alabiliriz. Readiness check fail edene kadar bu pod servisin arkasında yer alacaktır eğer fail olursa pod, servisin endpoint listesinden çıkarılır.
Readiness probe ile podun servis sunmaya hazır olup olmadığını kontrol edebiliriz.

Deploymentta yeni imaj versiyonunu verdiğimizde kubernetes hemen bu imajla bir pod ayağa kaldıracak demiştik. Pod oluşturuldu ve readiness çek tanımı varsa henüz servise eklenmez, initalDelaySeconds kadar bekler ve ilk kontrolü yapar. Olumlu cevap alırsa servis arkasına alınarak trafik almaya başlar. Tam bu noktada eski pod terminate edilmeye başlanır. Burada önemli bir nokta var. Eski podu kubernetes kapatıyorken bu poda hala servisten trafik gelebilir, kullanıcılar bu poda bağlanmaya devam edebilir ve tam işlem yaparken bu pod kapanmış olursa kullanıcılara hata mesajı dönecektir. Kubernetes bunu engellemek adına, bir pod terminate edileceği zaman bu sürede poda yeni trafik gelmemesi için  pod servis arkasına alınır fakat kubernetes yinede bu podu hemen kapatmaz. Çünkü yeni trafik gelmese bile bu pod hala eski istekleri işlemeye devam edebilir. O nedenle bu işlemlerin bitmesini beklemek gerekir.
İşte bu noktada **terminationGracePeriodSeconds** anahtarı karşımıza çıkıyor.

**terminationGracePeriodSeconds:** kubernetes ile uğraşıyorsanız bu anahtarı pod tanımlarında sıkça görmüşsünüzdür. Varsayılan değeri 30 sn'dir. Peki nedir bu anahtar?
Siz ya da kubelet bir podu terminate etmek istediğiniz zaman bu podun içinde çalışan pid=1 process'ine hemen kill ol kapan komutunu göndermez. Bunun yerine mevcut çalışan process'ler tamamlandığında kendisini kapatması için bir sinyal gönderilir. Aslında container'a kendini düzgün şekilde kapatabilmesi için atadığımız süredir.

İşte tüm bu teknikler sayesinde kesintisiz versiyon geçişleri yapabilmekteyiz, diyorum ve bu yazımı noktalıyorum. 

Görüşmek üzere..

![]({{ 'assets/img/im-alive.gif' | relative_url }})
