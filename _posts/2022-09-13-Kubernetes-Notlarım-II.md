---
title: Kubernetes Notlarım II- Kubernetes Objeleri | Pod
---

Merhabalar, daha önceki yazımda bir seri olarak Kubernetes notlarımı paylaşacağımdan bahsetmiştim. Serinin ikinci yazısı için keyifli okumalar diliyorum.
Serinin ilk yazısına [buradan](https://zeyneprumeysayorulmaz.github.io/k8s-notlar%C4%B1m/) ulaşabilirsiniz.

![]({{ 'assets/img/k8sobje.png' | relative_url }})
Kubernetes objeleri/kaynakları/bileşenleri, clusterın durumunu temsil eden Kubernetes sistemindeki kalıcı varlıklardır. Bu objeleri, veri serileştirme dili YAML ile tanımlayabilir ve oluşturulmadan önce doğrulandıkları api sunucusuna gönderebiliriz.  Basitçe ifade edecek olursak, Kubernetes üzerinde deploy ettiğimiz, çalıştırdığımız, koşturduğumuz yapılara obje deriz.
Bir obje oluşturarak, aslında cluster'a iş yükünün nasıl görünmesini istediğimizi söyleriz.  Kubernetes üzerinde her şey bir api objesidir ve bizler onun üzerinde koşturmak istediğimiz iş yüklerini objeler ile oluştururuz.

K8s üzerinde oluşturduğumuz en temel obje **pod**dur.  Pod, en küçük ve temel birimdir. Pod içerisinde bir veya daha fazla container barındırabilir fakat çoğu zaman bir pod içerisinde bir container çalışır.  Her podun benzersiz bir "uid" değeri ve yine benzersiz bir ip adresi vardır. 

Teorik kısımları geçelm ve birlikte bir pod oluşturalım. Pod oluşturmak için **kubectl run** komutunu kullanabilir ve yeni bir obje oluşturabiliriz. Aşağıdaki komut ile mypod adında bir pod oluşturuyor ve içerisinde çalıştırılacak container'ın bir  nginx  isimli imajdan oluşturulmasını söylüyorum. Son olarak da eğer podun içersindeki container kapanırsa yeniden çalıştırılmaması için **--restart=Never** parametresini kullanıyorum. Podu oluşturduktan sonra peşinden clusterda çalışan podları listelemek için **kubectl get pods** komutunu çalıştırıyorum.  
`kubectl run mypod --image=nginx --restart=Never` 
![]({{ 'assets/img/podcreate.PNG' | relative_url }})

**-o wide** ile geniş bir çıktı alabilirsiniz. Burada status kısmı podun statüsünü belirtiyor. Pod statülerine daha önceki yazımda değinmiştim. [Bu adresten](http://https://zeyneprumeysayorulmaz.github.io/Kubernetes-Pod-Pending/) ulaşabilirsiniz. Bu çıktıda önemli olan bir diğer ayrıntı Ready sütunudur. Burada gördüğümüz **1/1**, podda çalışması istenilen container sayısı 1 ve bu 1 container da şu anda çalışıyor ve hazır durumda anlamına gelmektedir.
![]({{ 'assets/img/owide.PNG' | relative_url }})

Herhangi bir objeye ait daha detaylı bilgi edinmek istersek **kubectl describe** komutunu kullanabiliriz. Hangi obje tipini ayrıntılı görmek istiyorsak komuta o obje tpini ve ismini vermeliyiz.

`kubectl describe pods mypod`

Container loglarını görmek isterseniz aşağıdaki gibi `kubectl logs mypod` komutunu çalıştırabilirsiniz. **kubectl logs -f mypod** ile canlı logları da izlemeniz mümkün.
![]({{ 'assets/img/logs.PNG' | relative_url }})

Podun içerisinde çalışan containere bağlanmak ve komut çalıştırmak isteyebilirsiniz, bunun için **kubectl exec** komutunu kullanabilirsiniz.

`kubectl exec mypod -- <çalıştırılmak istenen komut>`
![]({{ 'assets/img/komut.PNG' | relative_url }})

Poda bağlanmak isterseniz `kubectl exec -it mypod -- /bin/sh` komutunu kullanmalısınız.
![]({{ 'assets/img/execit.PNG' | relative_url }})

`kubectl delete pods mypod` komutu ile de oluşturduğunuz pod objesini silebilirsiniz.

Fakat pratikte biz objeleri direkt kubectl komutları ile oluşturmayız. Bu imperative yöntemdir.  Bir obje oluştururken o objede olmasını istediğimzi özellikleri belirttiğimiz bir dosya oluşturur ve k8s'e oluşturduğumuz dosyadaki özelliklere göre bir obje istediğimizi söyleyerek deklarasyonda bulunuruz. Bu declarative yöntemdir.

Kubernetesde obje tanımlarının yaml formatında yapılmasına izin verilir. Bizler objeleri yaml dosyaları şeklinde tanımlar ve k8s'e deklare ederiz. Her objenin tanımı farklıdır ama temelde benzer olan bir formatı takip eder.

Örneğin pod objesinin tanımlandığı aşağıdaki gibi bir yaml dosyası varsa bunu kuebctl apply komutu ile kubernetes'e deklare edebiliriz.

![]({{ 'assets/img/podyaml.PNG' | relative_url }})

`kubectl apply -f pod.yaml`

![]({{ 'assets/img/podapply.PNG' | relative_url }})

Kendiniz bir pod objesi oluşturabilirsiniz. Burada **apiVersion**,  **kind**, **metadata** anahtarlarının her k8s obje tanımında bulunması gerekir. **kind**, objenin ne olduğunu söylediğimiz anahtardır, yani objenin tipini burada söyleriz. **apiVersion** ise oluşturmak istediğimiz obje tipinin k8s apisinde hangi endpoint üstünde tanımlandığını belirttiğimiz anahtardır. Her objenin sunulduğu bir api vardır, obje tipine göre burada anahtara gerekli olan api bilgisini girmek gerekiyor. K8s dokümantasyonlarından veya kubectl aracı ile buradaki obje tipinin sunulduğu api bilgisini öğrenebiliriz. Aşağıdaki gibi `kubectl explain <obje-adı>` komutu ile bunu öğrenebilmemiz mümkün. Ezbere gerke yok, açık kaynak istediğimizde erişebiliriz :) 

![]({{ 'assets/img/explainpods.PNG' | relative_url }})

**metadata** ise obje ile ilgili uniq bilgilerin tanımlandığı yerdir. Objeye özel, objeyi betimleyen bilgiler burada tanımlanır. **spec** kısmı her obje tipine göre değişkenlik gösterir.  Buradaki tanımlanacak bilgileri de dokümantasyonlardan elde edebiliriz.

`kubectl describe pod <pod-name>` komutu ile oluşturduğumuz pod objesi ile ilgili detaylı bilgi edebiliriz. Events kısmı bir podun nasıl oluşturduğunu anlamak noktasında kıymetli, o yüzden biraz değinelim istiyorum.

![]({{ 'assets/img/poddescribe.PNG' | relative_url }})

### Pod yaşam döngüsü
Pod oluşturmak için ya imperative yöntemle kubectl aracını kullanarak bir pod oluşturuyoruz ya da bir yaml dosyasında istediğimiz poda dair detayları vererek yaml dosyasını kubectl apply komutu ile  k8s api'ine deklare ederek bir pod oluşturuyoruz. K8s API Server bu yaml dosyasını alır, bizim belirttiğimiz pod özellikleri ile varsayılan set edilmiş ayarları birleştirir, poda uniq ID atar ve tüm bunları etcd'ye kaydeder. Bu noktadan itibaren pod oluşturulmuştur ve yaşam döngüsüne başlar.

| State | Açıklama |
| -------- | -------- | 
| Pending   |  Sistemde bir pod oluşturuldu. Podla ilgili tüm gerekli tanımlamalar yapıldı ve veritabanına kaydedildi.  Ama pod henüz herhangi bir node üzerinde oluşturulmadı. Eğer pod uzun bir süre bu aşamada bekliyorsa scheduler uygun bir node bulamamış demektir. Bunun bir çok nedeni olabilir. Scheduler, etcd'yi sürekli gözler ve yeni yaratılan ataması yapılmamış pod varsa çalışmaya başlar.  Podun çalışabieceği en uygun node'a atamayı yapar ve bu aşamadan sonra pod creating duruma geçer. |
| Creating     |  kubelet, etcd'yi sürekli gözleyerek bulunduğu node'a atanmış br podları tespit eder. Yeni oluşturulmuş bir pod varsa hemen işlemlere başlar. Pod tanımında belirtilen container imajlarını sisteme indirmeye başlar, eğer bir şekilde indirme yapılamazsa pod, errImagePull ve ardından ImagePullBackOff aşamasına geçer. ImagePullBackOff, node'un imajı repositoryden çekemediği anlamına gelir. Bunun bir kaç nedeni olabilir. Imaj düzgün çekilirse pod diğer statüye geçer. |
|  Running    |  kubelet, container engine ile haberleşir ve ilgili containerların oluşturulmasını sağlar.  Containerlar çalışmaya başladığı an pod running duruma geçer.  Artık pod oluşturulmuş olur. Podun altında çalışan containerlar çalışmaya devam ettikçe pod running durumda kalmaya devam eder.  |
| Succeeded     |   Eğer containerların hepsi hata vermeden doğal olarak kapanırsa ve restart policy, never ya da on-failru olarak set edildiyse pod succeeded/completed statüsüne döner ve pod yaşam döngüsünü başarılı bir şekilde tamamlar.  | 
| Failed     | Yine restart policy, never veya on-failru olarak işaretlendiyse  ve containerlardan biri hata verip kapanırsa   pod failed statüsüne geçer.  | 

Fakat eğer restart policy, always olarak ayarlandıysa ve containerlar hata kaynaklı veya hata olmaksızın kapansa bile restart olur ve pod, hiç bir zaman completed veya failed state'e geçmez.  Bunun yerine podun içerisindeki container yeniden başlatılır ve running state'de devam eder. Fakat bu yeniden başlatma belirli aralıkta sık oluyorsa bir hata olduğuna kanaat getirir ve podu CrashLoopBackOff statüsüne koyar.

| CrashLoopBackOff     | Bir pod oluştu ama içerisindeki container veya containerlar belirli aralıklarla kapanıp duruyor demektir.      | 


***Temel Kural***
Container imajlarında, container oluşturulup çalıştırıldığı zaman balatılması belirlenen bir uygulama bulunur. Bu uygulama çalıştığı sürece container da çalışır durumdadır. Fakat uygulama çalışmayı bıraktığı zaman container da durdurulur. Uygulamanın çalışmayı bırakması da 3 şekilde olur.
* Uygulama görevlerini tamamlayıp otomatik şekilde hata vermeden kapanır.
* Kullanıcı ya da sistem uygulamayı kapanma sinyali gönderir ve hata vermeden kapanır.
* Ya da bir şekilde hata verir ve hata kodu oluşturarak kapanır.

Yukarıdakiler aklımızın bir köşesinde kalsın :) konuyu bağlayalım. Podun içerisinde çalışan containerlara podun içinde bir **restart policy** tanımlanır. Restart policy 3 değer alabilir.

**Always:** Default değerdir. Aksi belirtilmedikçe bu ayarlanır. Podun içindeki container hata vereverek veya vermeyerek hangi durumda durdurulursa durdurulsun o containerı yeniden başlat demektir.

**On-failru:** Sadece hata alıp kapanırsa yeniden başlatılır.

**Never:** Hiç bir zaman yeniden başlatılmaz.
