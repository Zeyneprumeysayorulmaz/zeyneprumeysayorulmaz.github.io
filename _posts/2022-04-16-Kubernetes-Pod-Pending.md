---
title: Kubernetes Pod Yaşam Döngüsü | Pod Statülerini Anlamak
---

Merhabalar, bu yazımda kubernetes podlarının yaşam döngüsünden ve baraberinde bu yaşam döngüsü boyunca karşılaştığımız pod statülerinden bahsedeceğim. Kubernetes ile uğraşanlar bilir. Pod yaşam döngüsünde bazı can sıkıcı durumlar olabiliyor. Şüphesiz CrashLoopBackoff bu durumlardan en can sıkıcı olanıdır ve bir yerde eminim karşınıza çıkmıştır. 

![]({{ 'assets/img/crashloopbackoff.jpg' | relative_url }})

ikinci sırada ise Pod Pending bizi karşılıyor.  Bu durum can sıkıcı aynı zamanda tedirgin edici olabiliyor.


Öncelikli olarak bir podun yaşam döngüsünü ve statülerini inceleyelim. Sonrasında kabusumuz olan pending statüsünde kalan pod üzerinde ne yapabileceğimize bakalım.

## Pod Yaşam Döngüsü
Bildiğimiz gibi podlar kubernetis' in en küçük birimidir. Her pod unique bir IP' ye sahiptir. Yaşam döngüsünde pod oluşturulur, çalışır ve hata ile karşılaşması durumunda ölür. Podlar Pending durumdan başlar, birincil konteynerlardan en az birinin OK vermesi durumunda Running statüsüne geçer ardından podlardaki her hangi bir container' ın hatayla sonlandırılıp sonlandırılmadığına bağlı olarak Succeeded veya Failed  aşamalarından geçerek tanımlanmış bir yaşam döngüsünü takip eder.
Bir Pod çalışırken, kubelet, bazı hataları işlemek için containerları yeniden başlatabilir. Bir Pod içinde Kubernetes, farklı container durumlarını izler ve podu tekrar sağlıklı hale getirmek için hangi eylemin gerçekleştirileceğini belirler. Pod'lar yaşamları boyunca yalnızca bir kez schedule edilir. Bir Pod, bir Node'a programlandığında (atandığında), Pod durana veya sonlandırılana kadar o Node üzerinde çalışır.
Bu yaşam döngüsü boyunca 5 çeşit Pod statüsü vardır. Gelelim bu statülere..
## Pod Status


| Value | Description |
| -------- | -------- |
| Pending     | Kubernetes Cluster' ı tarafından pod kabul edildi  ancak bir veya daha fazla container kurulmadı ve çalışmaya hazır hale getirilmedi anlamına gelir      |
| Running     | Pod bir Node' a bağlandı ve tüm containerlar oluşturuldu. En az bir container hala çalışıyor veya başlatılıyor veya yeniden başlatılıyor anlamına gelmektedir.     |
| Succeeded     | Poddaki tüm containerlar titizlikle sonlandırıldı, işlemleri yaptı ve yeniden başlatılmayacak anlamına gelen durumdur.     |
| Failed     | Poddaki tüm conatinerlar sonlandırıldı ve en az bir container bir hata nedeniyle sonlandırıldı.      |
| Unknown     | Bazı nedenlerden dolayı Pod'un durumu elde edilemedi anlamına gelir. Bu durum genelde, Pod'un çalışması gereken Node ile iletişimdeki bir hata nedeniyle oluşur.     |


## Pod Status: Pending  
![]({{ 'assets/img/podpending.png' | relative_url }})

Bir podun yaşam döngüsünün Pending statüsünden başladığından, pod programlandığında ve containerlar başlatıldığında podun çalışma aşamasında geçtiğinden bahsetmiştik. Podların çoğu Pending statüsünden Running statüsüne yalnızca bir kaç saniye içerisinde geçer. Pending statüsünde Pod, Kubernetes Cluster'ı tarafından kabul edilmiş fakat bir veya daha fazla container kurulmamış ve çalışmaya hazır hale getirilmemiş durumdadır. Pending durumundaki zaman zarfı içinde Pod, planlanmayı bekler ve ağ üzerinden container imgelarını çekmeye çalışır. Her pod’un yaşam döngüsü boyunca ilk statü olan “Pending” durumunda bulunur ama pod’ların bu durumda uzun süre geçiriyor olması kuşku uyandırabilir. Eğer bir pod pending statüsünden running statüsüne ilerleyemezse yaşam döngüsü, statünün ilerletilmesine engel olan durum çözülene kadar duraklar.
Pending statüsünde olması gerekenden fazla kalan bir pod varsa bunun nedeni yüksek ihtimalle bir hatadır ve üzerine düşünmeniz gerekir. CrashLoopBackOff dan sonra en sık karşılaşılan hatalardan biridir. Bazı durumlarda çözümü oldukça basit olsa bile kök neden analizi yaparak sorunu anlamak ve çözümü uygulamaya çalışmak önemlidir. 

Aşağıdaki gibi kubectl komutunu yürütürseniz podları listeler ve durumları hakkında bilgi edinebilirsiniz.

```
$kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
```

Bir podun pending statüsünde takılı kalmasının yani çalışamamasının birden fazla nedeni olabilir.  Bu duruma neden olan en yaygın sorunlara bakalım.
* Pod herhangi bir Node'a schedule edilmemiş veya edilememiş olabilir. En yaygın karşılaşılan  nedenler biridir.
* Container imageları çekilirken sorunlar oluşmuş olabilir.
* Podun çalışması için gereken bağımlıklar (volume, secret veya configmap) ile ilgili problemler meydana gelmiş olabilir.

##### Schedule sorunları nedeniyle Pending statüsünde kalıyor olması durumunu inceleyelim
Bir pod oluşturulduğunda Kubernetes Cluster'ının ilk yaptığı şey podu bir node üzerinde çalışacak şekilde programlamaktır. Bu prosedür genelde hızlı olmakla beraber pod, kendisini çalıştırabilecek yeterli kaynağa sahip bir node'a atanır. Bu işlemin yani Clusterın podu bir node'a programlanmasının doğru bir şekilde yapılamamasının birden fazla nedeni vardır.

* Programlanacak node üzerinde yeterli kaynak olmayabillir. Node çalıştırabileceği maximum pod sayısına ulaşmamışsa podun cpu ve memory talebine göre programlama yapılabilir. Eğer podun kaynak taleplerini karşılayabilecek bir node yoksa mevcutta kullanılan kaynaklar boşa çıkana kadar pod pending statüsünde tutulur.
* Schedule edilemeyen nodelar da bir başka sorundur.  Node'un bu durumu değişmediği takdirde herhangi bir pod o node'a schedule edilemez.
* Taints farklı nodelar üzerine atanabilecek podları sınırlamaya olanak tanıyan bir kubernetes  mekanizmasıdır. Node’ların belli özelliklere sahip podlara sahip olabilmelerine olanak sağlarlar. Taint node tarafındaki key valueleri temsil ederken tolerations pod tarafındaki key value leri temsil eder. Bu noktada bir nodeda sorun varsa o nodeda yalnızca eşleşen toleransa sahip podlar çalışabilir. Özetle  toleration taint ile uyuşuyorsa ilgili pod o node a girme hakkına sahip olur. Fakat taint ve tolerationın uyuşması podun o node a girmesini garanti etmez.

Tüm bu nedenlerin kombinasyonu schedule sorunlarına neden olmaktadır. Nodeun tahsis edilememesinin detayını incelemek için o pod hakkında gerçekleştirilen schedule işlemini incelemenizde fayda var. Message kısmında tam nedeni görebilirsiniz.
```
$ kubectl describe pod <POD-UID>
```

> Mevcutta çalışan iş yükünüz güncelleme aldıysa upgrade policy dikkat edilmesi gereken bir başka faktördür. Bu politika nedeniyle Kubernetes,  iş yükünün normalden daha fazla pod oluşturabilmesine izin verebilir ve yeni podları oluştururken eski podları bir süreliğine saklayabilir. Bu durumda iş yükü planlanandan daha fazla kaynak talep edecektir. Cluster da yeterli kaynak yoksa güncelleme durdurulur ve tekrar başlatılana kadar podlar pending statüsünde kalır.

##### Image sorunları nedeniyle podun Pending statüsünde kalıyor olması durumunu inceleyelim
Pod bir Node'a tahsis edildiğinde, kubelet pod spesifikasyonundaki tüm containerları başlatmaya çalışacaktır. Bunu yapmak için  image'ı indirmeye ve ardından çalıştırmaya çalışacaktır.
Yine imageın çekilmesini engeleyebilecek bir kaç hata karşımıza çıkıyor.
* Image name hatalı olabilir.
* Image tag'i hatalı olabilir.
* Repositoryde kimlik doğrulama gerekli olabilir.

Tüm bu yapılandırmaların doğru olduğundan emin olun.

##### Dependency sorunları nedeniyle Pending statüsünde kalıyor olması durumunu inceleyelim. 
Pod başlamadan önce kubelet, diğer Kubernetes öğeleriyle olan tüm bağımlılıkları kontrol etmektedir. Bu bağımlılıklardan birinin bile  karşılanmaması durumunda, pod, bağımlılıklar karşılanana kadar Pending statüsünde tutulur. Bu aşamada kaldıysa pod status ContainerCreating olarak gözükecektir. Events alanında message sütununu detaylı incelemenizde fayda olacaktır. 

Bir podun Pending statüsünde kalıyor olmasının nedenlerini anlamak, iş yüklerinizi Kubernetes üzerinde dağıtmak ve yönetmek noktasında önemlidir. Kubernetesde dağıtımlarınızı yönetiyorsanız tüm süreçlerin sağlıklı ilerlediğinden emin olmanız ve sorunlarınıza hızlı aksiyon alabilmeniz önemlidir.

![]({{ 'assets/img/one-does-not-simply-deploy-kubernetes.jpg' | relative_url }})
