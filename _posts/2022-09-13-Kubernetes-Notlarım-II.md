---
title: Kubernetes Notlarım II- Kubernetes Objeleri
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

Kubernetesde obje tanımlarının yaml formatında yapılmasına izin verilir. Bizler objeleri yaml dosyaları şeklinde tanımlar ve k8s'e deklare ederiz. Her objenin tanımı farklıdır ama temelde benzer olan br formatı takip eder.
