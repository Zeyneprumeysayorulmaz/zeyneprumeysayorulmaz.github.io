---
title: Kubernetes Notlarım IV- Kubernetes Objeleri | Deployment | ReplicaSet | Rollout
  ve Rollback
---

> Merhabalar, hız kesmeden serinin dördüncü yazısını kaleme almak istedim.  Bu yazımda deployment ve replicaset objelerine değinecek beraberinde ise rollout ve rollback özelliklerine değineceğim. Keyifli okumalar.
 
![]({{ 'assets/img/deployment.png' | relative_url }})
 
Pod bölümünde, podların kubernetesin en temel objesi olduğuna değinmiştik. Bizler kubernetesde container imajına çevirdiğimiz uygulamalarımızı pod olarak deploy ederiz. Genelde kubernetes üzerinde singleton (tekil) olarak adlandırılan tekil, yönetilmeyen podlar yaratmayız. Podları yöneten üst seviye objeler yaratırız ve podlar bu objeler tarafından yaratılıp, yönetilir. Bu objelerin en sık kullanılanlarından bir tanesi deployment objesidir.
### ****Peki neden tekil podlar (singleton pods) yaratmayız?

Örneğin bir frontend uygulamamız var ve bunu production ortamında deploy ettik. Uygulamamızı container imajı haline getirdik  ve bunu kubernetesde deploy etmek için bir pod tanımı yarattık. Diyelim ki 3 worker nodelu bir yapımız var ve kubeScheduler bu podu en uygun nodeu seçerek o node üzerinde oluşturdu.  Fakat bir süre sonra bu podun oluşturulduğu worker node bozuldu ve erişilemiyor. Dolayısıyla poda da erişemiyoruz. Bu durumda pod Terminating statüsüne geçer  ve öylece kalır. Pod çalışmadığından uygulama da çalışmayacaktır. 

Daha önceki yazımda da bahsetmiştim. Bir pod yarattığımız zaman kubeScheduler bu podun çalışacağı en uygun nodeu bulur ve orada bu podu çalıştırır.  Eğer container seviyesinde bir sıkıntı çıkarsa ve restartPolicy, onFailru ya da  always olarak seçildiyse container restart edilir ve sorun çözülür. Ama node seviyesinde bir sıkıntı olmuşsa veya kaynak sıkıntısı sebebiyle o pod durdurulursa kubeScheduler bu pod için başka bir node arayışına girmez ve atama yapmaz. Bu nedenle node erişilemez duruma gelirse podda Terminating durumuna geçer ve o şekilde kalır.  Diyalim ki bu soruna çözüm için replica olarak 3 pod yarattık ve nodeSelector kullanarak bu 3 podun farklı nodelarda çalışmasını sağladık. Son durumda 3 worker node üzerinde çalışan 3 aynı pod yarattık.  Önlerine de bir Load Balancer koyduk ve birine bir şey olması durumunda diğer iki tanesine erişilebilmesini sağladık.  Fakat burada da şu sorun karşımıza çıkar: Bu uygulama üzerinde geliştirmelerim devam ediyor diyelim. Yeni bir versiyon çıktım ve yeni bir container imajı yarattım. Bu imajı her pod tanımında tek tek düzeltmem ve bu yeni imajla oluşturmam gerekir. Bu oldukça zahmetli bir iştir. Süreç ilerledikçe eski versiyona dönmek gibi veya label eklemek gibi yeni senaryolar oluştuğunda bu yapıda işler oldukça zorlaşır. Çünkü her pod tanımında bunu düzeltmem gerek ve fazlasıyla bir iş yükü. 

İşte bu noktada benim bu işleri manuel yapmamdansa deployment objesi kullanmam daha sağlıklı olacaktır. **Deployment**, bir veya birden fazla podu bizim belirlediğimiz desired state'e göre oluşturan ve sonrasında bu istenilen durumu mevcut durum ile sürekl karşılaştırıp gerekli düzeltmeleri yapan bir obje tipidir. Bizler bir deployment objesi için tanım yapar ve bu tanımın içinde oluşturmak istediğimiz podun hangi özelliklerde, kaç adet olacağını belirtiriz.  Bu deployment objesi oluşturulduğu zaman belirtilen tanımda ve addette pod oluşturulur.

> Özetle, bizler her ne kadar kubernetes  üzerinde podlar oluşturuyor olsak da bu podları direkt pod tanımlarıyla oluşturmayız.  Oluşturmak istediğimiz podları daha üst seviye objeler aracılığıyla oluşturur ve uzun vadede yapmamız  gerekecek bir çok manuel işi otomatize etmiş oluruz. Deployment ise bu objeler içerisinde en sık kullanılan objedir. Neredeyse iş yüklerinin tamamı deployment objesi olarak deploy edilir.

![]({{ 'assets/img/deployment2.png' | relative_url }})

### Deployment Objesi
`kubectl create deployment uygulama1 --image=nginx:latest --replicas=3` komutu ile imperative olarak bir deployment objesi oluşturabilirim. Oluşturmak istediğim obje tipinin deployment olduğunu söyledim, buna bir isim verdim ve oluşturmak istediğim container imajının nginx olmasını , --replicas ile de kaç adet pod oluşturacağımı belirttim.

`kubectl get deployment` ile bu obje tanımına bakabiliriz. 

Örneğin replicas olarak 3 belirttiğiniz bir deploymentta manuel olarak bir podu sildiniz veya silindi diyelim.

`kubectl delete pod <pod-name>` 

Burada deployment controller devreye girer ve silinen podun yerine desired state'te belirtildiği gibi bir pod daha oluşturur. Yani tanımda verilen istenen durum ile mevcut durum arasında fark varsa bunu hemen düzeltir ve istenen duruma getirir. aslında bunu ReplicaSet yapıyor ama şimdilik böyle diyelim, aşağıda detaylı değinelim.

Mevcut deployment üzerinde değişiklik yapmak istersem örneğin imaj değiştirmek istiyorum diyelim;

`kubectl set image deployment/uygulama1 nginx=httpd` burada uygulama1 adlı deployment objesindeki imaj değerinin nginx yerine httpd olmasını belirttim. Deployment objesi istenen değişiklikleri yapmak için sırasıyla podları kapatır ve yeni imajla ayağa kaldırır.

`kubectl scale deployment uygulama1 --replicas=4` ile çalışan pod sayısının 4 olmasını söyleyebilirim.  **scale** komutuyla pod sayısında değişiklikler yapabiliriz.

`kubectl delete deployments uygulama1` silmek için delete kullanabiliriz.

Kubectl ile imperative yöntemle işlemleri yapabildik, peki deklaretive yöntemle bunu nasıl yapıyoruz?

Örneğin aşağıdaki gibi bir deployment objesini tanımlayan dosya oluşturabilir ve bunu k8s api'sine deklare edebiliriz.

![]({{ 'assets/img/deploymentyaml.PNG' | relative_url }})

Obje yaml dosyalarında 4 temel alan olduğuna daha önceki yazılarda değinmiştik.  **kind** ile objenin deployment olduğunu belirtiyoruz.  Bu obje **apps/v1** isimli apide bulunuyor.  Bu nedenle **apiVersion** kısmında bunu yazıyoruz. **metadata** kısmında deploymenta bir isim veriyor ve dilersek label tanımlarını yapabiliyoruz.  **spec** kısmında deployment objesi ile ilgili özellikleri belirtiyoruz.  **replicas** anahtarı ile kaç replica olması gerektiğini belirtiyoruz.  Bir önceki yazımda k8s'in objeler arasındaki bağı labellar ile kurduğunu söylemiştim. **selector** kısmında, bu deploymenta ait podlar ile  bağ kurabilmesi için label kullandık. **matchlabels** ile deploymenta, senin yöneteceğin podlar app: frontend labelına sahip podlar diyoruz aslında. Sonrasında template altında da bunu belirtiyorum ki yeni yaratılacak podlara da bu eklensin ve seçme işlemi tamamlansın. **template** kısmında ise bu deployment objesi tarafından oluşturulacak podların özelliklerini belirtiyoruz. Farkettiyseniz bu template altındaki kısımlar benim pod tanımımdaki anahtarlar. 

Selector kısmı önemli ve mecburi bir alan.
*  Bir deploymentta en az bir selector tanımı olmalı ve aynı labellar template kısmında da bulunmalı. Yoksa deployment yaml dosyasını apply edemezsiniz.
*  Her deployment objesinde farklı label ve selector kullanmalısınız. Aynı labellara sahip birden fazla deployment objesi yaratırsanız, deploymentlar hangi podların kendilerine ait olduklarını karıştırabilir.
*  Aynı labelları kullandığınız başka objeler yaratırsanız sıkıntı çıkarabilir.

`kubectl apply -f uygulama1.yaml` ile deklerative olarak oluşturduğunu deploymentı apply edebilirsiniz.

### ReplicaSet Objesi

![]({{ 'assets/img/deployment-high-level.png' | relative_url }})

Kubernetesde,  belirtilen templatedeki bilgilere göre x sayıda pod oluşturan ve sonrasında mevcut durum ile istenilen durum arasındaki farkı gözeten aslında deployment objesi değildir. ReplicaSet adında bir obje tipi mevcuttur ve bu işlemleri replicaSet objesi yapar. Bir deployment oluşturuduğumuz zaman deployment tanımında template altında belirttiğimiz özellikte podları deployment kendi oluşturmaz, deployment bu özelliklerde bir replicaSet objesi oluşturur ve podlar bu replicaSet objesi tarafından oluşturulur, aynı zamanda bu podların desired statede kalmasını sağlar.

***Bir replicaSet'in amacı, herhangi bir zamanda çalışan kararlı bir replika pod setini sürdürmektir. Bu nedenle, genellikle belirli sayıda özdeş podun kullanılabilirliğini garanti etmek için kullanılır.***

Özetle; biz bir deployment objesi oluşturduğumuz zaman bu deployment objesi kendi yönettiği bir replicaSet objesi oluşturur ve bu replicaSet objeside podları yaratır/yönetir. Eğer deployment tanımında bir değişiklik yaparsak; deployment, bu yeni tanımla yeni bir replicaSet objesi oluşturur.  İlk yaratılan replicaSet objesi yavaş yavaş kendi oluşturduğu podları silmeye başlar, yeni replicaSet ise yeni podları yaratır. Bu silme ve oluşturma işleminin hangi sırayla ne şekilde olacağını biz belirleyebiliriz.  Bu özellik bize herhangi bir kesinti olmadan uygulama güncelleme ve yeni versiyon geçiş imkanı sağlar.

Manuel olarak da bizler replicaSet oluşturabiliriz. Aşağıda bir replicaSet objesine ait tanım yer almakta. Tanım olarak deployment ile nerdeyse aynı.

![]({{ 'assets/img/replica.PNG' | relative_url }})

Fakat ReplicaSet; rollout, undo gibi özellikleri bize sunmadığı için replicaSet yaratmak yerine deployment yaratırız ve deployment bu replicaSeti yaratır, bizde bu işlemlerimizi gerçekleştirebiliyor oluruz.

Örneğin; replicaSet üzerinde bir değişiklik yaptık diyelim ve image kısmını değiştirdik. ReplicaSet, mevcut podları yeni imajla tekrar ayağa kaldırmaz yani bu tarz değişiklikleri mevcut podlara uygulamaz ancak yeni bir pod yaratacaksa bu podu yeni imajdan yaratır. İşte bu nedenlerle bizler direkt replicaSet yaratmıyor bunun yerine deployment kullanıyoruz.

### Rollout ve Rollback
Deployment yaml dosyalarında spec altında **strategy** anahtarıyla bizler, deploymentı güncellediğimiz zaman rollout işlemlerinin nasıl yapılacağını belirleriz. Kullanabileceğimiz iki tip rollout stratejisi mevcuttur.  Aşağıdaki yaml dosyasında bunlardan biri olan **recreate** set edilmiştir. Recreate şu anlama gelir: Bu deployment üzerinde bir değişiklik yapılırsa öncelikle mevcut tüm podları sil ve bu işlem tamamlandıktan sonra yeni podları oluştur. Bu yapıda kesinti olacaktır fakat major değişikliklerde bu starteji tercih sebebi olabilmektedir.

![]({{ 'assets/img/rollout.png' | relative_url }})

Eğer yaml dosyasında bir strateji belirtmezsek varsayılan olarak **RollingUpdate** Startejisi seçilir.

![]({{ 'assets/img/rollingupdate.PNG' | relative_url }})

RollingUpdate, recreate stratejisinin tam tersidir. Deploymentta bir değişiklik yaptığım zaman podlar üzerinde bu işlemi aşamalı olarak yapar. Bu aşamaların nasıl olacağını belirleyebileceğimiz iki opsiyon mevcuttur. Bu opsiyonlar yukarıda da görebileceğiniz üzere **maxUnavailable ve maxSurge** değerleridir. Bu strateji bizlere **zero downtime** geçiş yapma imkanı verir.

* **maxUnavailable**: Bir deploymentta değişiklik yaptığım zaman en fazla burada belirlediğim sayıdaki podu sil. Değişiklikleri uyglarken aynı anda en fazla x pod silebilir demektir aslında. Burada sayı yerine yüzdelik ifade de kullanılabilir.

* **maxSurge**: Geçiş sırasında toplam pod sayısının en fazla kaç olabileceğini belirler.  Örneğin desired state 10 pod olan bir ortamda geçiş sırasında 2 pod ayağa kalkacak ve toplamda 12 pod sistemde aynı anda çalışabilir. 

Örneğin aşağıdaki gibi deployment objemin image değerini değiştiriyor olayım.

`kubectl set image deployment uygulama1 nginx=httpd:alpine --record=true`

Buradaki record bize ne sağlıyor?  Deployment rollout işlemlerinde bir sıkıntı olduğu zaman daha önceki duruma geçebilmemiz mümkün.  **--record** ile ne zaman, neyi değiştiridik bu bilgiyi izleyebiliyoruz. Eğer bir değişiklikte --record parametresini kullanırsanız liste halinde history alabilmeniz mümkün.

`kubectl rollout history deployment uygulama1` --> rollout history komutu bizlere deployment üzerinde yapılan değişiklikleri gösterme imkanı sunuyor. Bu komut çıktısında uygulama1 deployment objesi içinde --record parametresi set edilerek yapılan tüm değişiklikler bizlere listenir.  

Eski bir versiyonu görmek istersem:

`kubectl rollout history deployment uygulama1 --revision=2` --> örneğin revision 2 numaralı değişikliği gösterir.

`kubectl rollout undo deployment uygulama1` dersem bir önceki versiyona/bir önceki değişikliğe geri dönebilirim. 

Eğer spesifik olarak dönmek istediğim bir revision varsa;

`kubectl rollout undo deployment uygulama1 --to-revision=3` --> örneğin 3 numaralı revision'a geri dönebilirim.

Özetle bu sayede bizler deployment üzerinde yaptığımız değişiklikleri kataloglama ve sonrasında da geri dönme imkanına kavuşuruz.

Son olarak işinize yarayacğını düşündüğüm bir kaç faydalı komutu da paylaşmak istiyorum.

`kubectl rollout status deployment uygulama1 -w` --> rollout status komutu ile bir deployment üzerinde yapılan değişiklikleri canlı olarak  adım adım izleyebiliriz.

`kubectl rollout pause deployment uygulama1` --> rollout pause komutu ile değişikliği direkt geri almaktansa değişikliği durdurabiliriz ki geçiş esnasında oluşan bir sıkıntının yani yeni versiyonun  bir soruna yol açması ihtimalinin analizini yapmak kolay olsun. Direkt geçişi iptal ettiğim eski versiyona döndüğüm durumda sıkıntının/sorunun kaynağının değişiklik kaynaklı mı olduğunu anlayamam. Sorunu çözdüğünüz durumda veya sorunun değişiklik kaynaklı olmadığını anladığınız durumda `kubectl rollout resume deployment uygulama1` komutu ile işleme devam edebilirsiniz, diyelim ve konuyu noktalayalım.  😀

Bir sonraki yazımı benim fikrimce en can alıcı (!) konu olan Kubernetes Ağ Altyapısı ve Servisler üzerine yazmayı planlıyorum. Hatta bunu publish edip peşine onu yazıyor olacağım, hızımı alamadım 😀 Görüşmek üzere..
