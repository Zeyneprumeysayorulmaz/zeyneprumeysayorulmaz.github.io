---
title: Kubernetes Pod Yaşam Döngüsü | Pod Statülerini Anlamak
---

Merhabalar, bu yazımda kubernetes yapısında yer alan podların yaşam döngüsünden ve baraberinde bu yaşam döngüsü boyunca karşılaştığımız pod statülerinden bahsedeceğim. Kubernetes ile uğraşanlar bilir. Pod yaşam döngüsünde bazı can sıkıcı durumlar olabiliyor. Şüphesiz **CrashLoopBackOff** bu durumlardan en can sıkıcı olanıdır ve bir yerde eminim karşınıza çıkmıştır. 😬

![]({{ 'assets/img/crashloopbackoff.jpg' | relative_url }})

İkinci sırada ise **Pod Pending**  bizi karşılıyor.  Bu durum can sıkıcı aynı zamanda tedirgin edici olabiliyor.


Öncelikli olarak bir podun yaşam döngüsünü ve statülerini inceleyelim. Sonrasında kabusumuz olan pending statüsünde kalan pod üzerinde ne yapabileceğimize bakalım.

## Pod Yaşam Döngüsü
![]({{ 'assets/img/podlifecycle.png' | relative_url }})

Bildiğimiz gibi podlar kubernetes' in en küçük birimidir. Her pod unique bir IP' ye sahiptir. Yaşam döngüsünde pod oluşturulur, çalışır ve hata ile karşılaşması durumunda ölür. Podlar **Pending** durumdan başlar, birincil konteynerlardan en az birinin **OK** vermesi durumunda **Running** statüsüne geçer ardından podlardaki herhangi bir container' ın hatayla sonlandırılıp sonlandırılmadığına bağlı olarak **Succeeded** veya **Failed**  aşamalarından geçerek tanımlanmış bir yaşam döngüsünü takip eder.

Bir pod çalışırken kubelet, bazı hataları işlemek için containerları yeniden başlatabilir. Bir pod içinde Kubernetes, farklı container durumlarını izler ve podu tekrar sağlıklı hale getirmek için hangi eylemin gerçekleştirileceğini belirler. Podlar yaşamları boyunca yalnızca bir kez schedule edilir. Bir pod, bir node'a programlandığında (atandığında), pod durana veya sonlandırılana kadar o node üzerinde çalışır.

Bu yaşam döngüsü boyunca podların içinde bulunduğu 5 farklı statü karşımıza çıkıyor.  Bu durumları inceleyelim.
## Pod Status


| Value | Description |
| -------- | -------- |
| Pending     | "Kubernetes Cluster' ı tarafından pod kabul edildi  ancak bir veya daha fazla container kurulmadı ve çalışmaya hazır hale getirilmedi." anlamına gelmektedir.      |
| Running     | "Pod bir node' a bağlandı ve tüm containerlar oluşturuldu. En az bir container hala çalışıyor veya başlatılıyor veya yeniden başlatılıyor." anlamına gelmektedir.     |
| Succeeded     | "Poddaki tüm containerlar titizlikle sonlandırıldı, işlemleri yaptı ve yeniden başlatılmayacak." anlamına gelen durumdur.     |
| Failed     | "Poddaki tüm conatinerlar sonlandırıldı ve en az bir container bir hata nedeniyle sonlandırıldı."  anlamına gelmektedir.     |
| Unknown     | "Bazı nedenlerden dolayı pod'un durumu elde edilemedi." anlamına gelir. Bu durum genelde, Pod'un çalışması gereken node ile iletişimdeki bir hata nedeniyle oluşur.     |


## Pod Status: Pending  
![]({{ 'assets/img/podpending.png' | relative_url }})

Bir podun yaşam döngüsünün Pending statüsünden başladığından, pod programlandığında ve containerlar başlatıldığında podun çalışma aşamasına geçtiğinden bahsetmiştik. Podların çoğu Pending statüsünden Running statüsüne yalnızca bir kaç saniye içerisinde geçer. Pending statüsünde pod, Kubernetes Cluster'ı tarafından kabul edilmiş fakat bir veya daha fazla container kurulmamış ve çalışmaya hazır hale getirilmemiş durumdadır. Pending durumundaki zaman zarfı içinde pod, programlanmayı bekler ve ağ üzerinden container imgelarını çekmeye çalışır. Her pod yaşam döngüsü boyunca ilk statü olan “Pending” durumunda bulunur ama pod’ların bu durumda uzun süre geçiriyor olması kuşku uyandırabilir. Eğer bir pod pending statüsünden running statüsüne ilerleyemezse yaşam döngüsü, statünün ilerletilmesine engel olan durum çözülene kadar duraklar.

Pending statüsünde olması gerekenden fazla kalan bir pod varsa bunun nedeni yüksek ihtimalle bir hatadır ve üzerine düşünmeniz gerekir. CrashLoopBackOff dan sonra en sık karşılaşılan hatalardan biridir. Bazı durumlarda çözümü oldukça basit olsa bile kök neden analizi yaparak sorunu anlamak ve çözümü uygulamaya çalışmak önemlidir. 

Aşağıdaki gibi kubectl komutunu yürütürseniz podları listeler ve durumları hakkında bilgi edinebilirsiniz.

```
$kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
```

Bir podun pending statüsünde takılı kalmasının yani çalışamamasının birden fazla nedeni olabilir.  Bu duruma neden olan en yaygın sorunlara bakalım.
* Pod herhangi bir node'a schedule edilmemiş veya edilememiş olabilir. En yaygın karşılaşılan  nedenler biridir.
* Container image'ları çekilirken sorunlar oluşmuş olabilir.
* Podun çalışması için gereken bağımlıklar (volume, secret veya configmap) ile ilgili problemler meydana gelmiş olabilir.

##### Schedule sorunları nedeniyle Pending statüsünde kalıyor olması durumunu inceleyelim.
Bir pod oluşturulduğunda Kubernetes Cluster'ının ilk yaptığı şey podu bir node üzerinde çalışacak şekilde programlamaktır. Bu prosedür genelde hızlı olmakla beraber pod, kendisini çalıştırabilecek yeterli kaynağa sahip bir node'a atanır. Bu işlemin yani Cluster'ın podu bir node'a programlamasının doğru bir şekilde yapılamamasının birden fazla nedeni vardır.

* Programlanacak node üzerinde yeterli kaynak olmayabillir. Node çalıştırabileceği maximum pod sayısına ulaşmamışsa podun cpu ve memory talebine göre programlama yapılabilir. Eğer podun kaynak taleplerini karşılayabilecek bir node yoksa mevcutta kullanılan kaynaklar boşa çıkana kadar pod pending statüsünde tutulur.
* Schedule edilemeyen nodelar da bir başka sorundur.  Node'un bu durumu değişmediği takdirde herhangi bir pod, o node'a programlanamaz.
* Taints, farklı nodelar üzerine atanabilecek podları sınırlamaya olanak tanıyan bir kubernetes  mekanizmasıdır. Node’ların belli özelliklere barındıran podlara sahip olabilmelerine olanak sağlarlar. Taint node tarafındaki key valueleri temsil ederken tolerations pod tarafındaki key value leri temsil eder. Bu noktada bir node üzerinde sorun varsa o node üzerinde yalnızca eşleşen toleration sahip podlar çalışabilir. Özetle  toleration, taint ile uyuşuyorsa ilgili pod o node'a girme hakkına sahip olur. Fakat taint ve toleration'ın uyuşması podun o node a girmesini garanti etmez.

Tüm bu nedenlerin kombinasyonu schedule sorunlarına neden olmaktadır. Node'un tahsis edilememesinin detayını incelemek için o pod hakkında gerçekleştirilen schedule işlemini incelemenizde fayda var. Message kısmında tam nedeni görebilirsiniz.

```
$ kubectl describe pod <POD-UID>
```

> Mevcutta çalışan iş yükünüz güncelleme aldıysa upgrade policy dikkat edilmesi gereken bir başka faktördür. Bu politika nedeniyle Kubernetes,  iş yükünün normalden daha fazla pod oluşturabilmesine izin verebilir ve yeni podları oluştururken eski podları bir süreliğine saklayabilir. Bu durumda iş yükü planlanandan daha fazla kaynak talep edecektir. Cluster'da yeterli kaynak yoksa güncelleme durdurulur ve tekrar başlatılana kadar podlar pending statüsünde kalır.

##### Image sorunları nedeniyle podun Pending statüsünde kalıyor olması durumunu inceleyelim.
Pod bir node'a tahsis edildiğinde, kubelet pod spesifikasyonundaki tüm containerları başlatmaya çalışacaktır. Bunu yapmak için  image'ı indirmeye ve ardından çalıştırmaya çalışacaktır.
Yine image'ın çekilmesini engeleyebilecek bir kaç hata karşımıza çıkıyor.
* Image name hatalı olabilir.
* Image tag'i hatalı olabilir.
* Repositoryde kimlik doğrulama gerekli olabilir.

Tüm bu yapılandırmaların doğru olduğundan emin olmanız önemlidir.

##### Dependency sorunları nedeniyle Pending statüsünde kalıyor olması durumunu inceleyelim. 
Pod başlamadan önce kubelet, diğer Kubernetes öğeleriyle olan tüm bağımlılıkları kontrol etmektedir. Bu bağımlılıklardan birinin bile  karşılanmaması durumunda pod, bağımlılıklar karşılanana kadar Pending statüsünde tutulur. Bu aşamada kaldıysa pod status **ContainerCreating** olarak gözükecektir. Events alanında yer alan message sütununu detaylı incelemenizde fayda olacaktır. 

Bir podun Pending statüsünde kalıyor olmasının nedenlerini anlamak, iş yüklerinizi Kubernetes üzerinde dağıtmak ve yönetmek noktasında önemlidir. Kubernetes'de dağıtımlarınızı yönetiyorsanız tüm süreçlerin sağlıklı ilerlediğinden emin olmanız ve sorunlarınıza hızlı aksiyon alabilmeniz önemlidir.

Bir sonraki yazıda görüşmek üzere..

![]({{ 'assets/img/one-does-not-simply-deploy-kubernetes.jpg' | relative_url }})
