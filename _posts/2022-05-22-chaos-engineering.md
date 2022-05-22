---
title: Chaos Engineering-LitmusChaos
---

Merhabalar, bu yazımda Chaos Engineering konusunu ele aldım.  Beraberinde açık kaynak bir Chaos Engineering platformu olan Litmus'u ve kubernetes ortamınıza nasıl dağıtabileceğinizi anlatmaya çalıştım. Keyifli okumalar..

![]({{ 'assets/img/chaos1.jpeg' | relative_url }})

Mikroservis mimarileri sağladığı bir çok fayda ile hayatımıza yerini aldı. Günümüzde ise yaygınlaşmaya devam ediyor. Beraberinde ise birtakım problemler ve soru işaretleri ile geliyor. Uygulama servis sayılarının artması ve altyapının tüm bu servislerin birbiri ile konuştuğu dağıtık yapılara dönmesi servislerin yönetimini zorlaştırmakta. Aynı zamanda uygulamaya dair hata ayıklama durumlarını da zorlaştırmaktadır. Takdir edersiniz ki production ortamında karşımıza çıkan olası hatalar ve beraberinde servis kesintisi istenmeyen bir durum.  Canlı ortamda bu durumların yaşanması can sıkıcı aynı zamanda müdahalesi güç olabiliyor.

Tüm bu ihtimaller göz önünde bulundurarak ve bu ihtimalleri uygulayarak bir çalışma yapabilirisiniz. Böylece etkilerini ve alınabilecek aksiyonları tespit etmiş olursunuz. Sistemlerinizin ne kadar dağıtık veya karmaşık olduğu fark etmeksizin, oluşabilecek problemlere karşı sistemlerinizin dayanıklılığını ve verebileceği tepkiyi görebilirisiniz. 
Kaos Mühendisliğinde amaç uygulamalar üzerinde kontrollü deneyler gerçekleştirerek bu deneylere karşı sistemin tepkisini ve ayakta kalabilme durumunu ölçmektir. Burada bir kaos yaratabilir sonrasında bu kaosa karşı oluşan durumları, sistemin davranışlarını inceleyebilirsiniz.

![]({{ 'assets/img/chaos2.jpg' | relative_url }})

---

Deneylerin planlı ve düzenli yapılması önemlidir. Deneyi planlayarak başlamalısınız. Chaos Engineering için kullanılan araçları inceleyecek olursanız uygulayabileceğiniz birçok hipotez ile karşılaşacaksınız. Oluşabilecek kaoslar için senaryolar oluşturulmuş ve uygulayabilmeniz için hazırdır. Ek olarak sizde bir hipotez oluşturabilirsiniz. Sonrasında bu kaos senaryosunu uygulayabilirsiniz. Oluşturduğunuz kaosa karşı sisteminiz nasıl davranıyor ve ayakta kalabiliyor mu gibi soruların cevaplarını toplayabilirsiniz. Deney sonuçlarının analizi de oldukça önemlidir. Bu sonuçlara göre sistemin dayanıklılığı için alınabilecek aksiyonları ve geliştirmeleri yapmalısınız.

---

**Özetle;** bu deneyleri ve ölçümleri yapmak canlı ortamda çalışan uygulamalarınızın kaotik durumların içerisinde bulunabileceğini öngörerek çalışmalar yürütmek sunduğunuz hizmetlerin kesintisiz ve tutarlı olarak sağlanabilmesi noktasında kıymetlidir. Netflix bu çalışmaları 2010 yılında başlatmış. Sahip oldukları micro servislerini AWS'e taşırken oluşabilecek kaotik durumların testini gerçekleştirmek istemiş ve bu sayede oluşabilecek durumlara olabildiğince hazırlıklı olmayı hedeflemişler. Bu çalışmalar için ise Chaos Monkey aracını geliştirmişler. Kaynak kodları ise açık durumda github da yer almakta.  [Bu](http://https://github.com/Netflix/chaosmonkey) adresten inceleyebilirsiniz. Netflix'in yüzlerce micro servisi barındırdığı canlı sistemlerine tamamen rastgele kaos oluşturan bir bot ekibi bulunuyor. Böylece bu durumlara karşı hazırlıklı oluyorlar ki ne kadar başarılı olduklarını da görüyoruz.

![]({{ 'assets/img/chaos3.png' | relative_url }})

[CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io/) adresine göz atacak olursanız Chaos Engineering için kullanılan araçları görebilirsiniz. Burada Chaos Mesh, Litmus, ChaosToolkit, Gremlin gibi araçlar karşımıza çıkıyor. Bu yazımda Litmus'a değinecek ve kubernetes ortamına nasıl dağıtılacağını ele alacağım.
Litmus, Kubernetes için cloud native bir chaos engineering aracıdır. Kubernetes tabanlı sistemlerde kaos testlerini oluşturmak ve çalıştırmak için tamamen açık kaynaklı bir üründür. 

Litmus 4 bileşenden oluşuyor.

![]({{ 'assets/img/chaos4.png' | relative_url }})

---

Mevcutta 3 API sağlamakta; **ChaosEngine, ChaosExperiment ve ChaosResult.**

Bir uygulama için yapacağınız bir chaos testi olduğunu varsayalım. Burada bu uygulama için **ChaosEngine CR** oluşturulup bir appLabel ile etiketlenir. Sonrasında bu CR, uygulayacağınız senaryolara göre bir veya daha fazla ChaosExperiment'ı bağlar. 

**ChaosExperiment CR** ise kaosu uygulayacağınız uygulamadaki senaryonun ayrıntılarını, parametrelerini taşımaktadır.
ChaosResult CR, senaryo çalıştırıldıktan sonra operatör tarafından oluşturulur. Her ChaosEngine için bir ChaosResult CR yer almaktadır. Chaos Experiment'ın anlamlandırılması noktasında fayda sağlamaktadır. 

**Chaos Operator** ise Chaos CR'ların yaşam döngüsünü yönetmektedir. Chaos operatörü [OperatorHub.io](https://operatorhub.io/operator/litmuschaos) 'da mevcuttur.

Chaos deneyleri aslında **chaos kütüphaneleri veya chaos yürütücüleri** tarafından yapılmaktadır. Örneğin Litmus projesi "LitmusLib" adında bir chaos kütüphanesi oluşturur. LitmusLib, senaryoların örneğin bir podun nasıl öldürüleceğinin, CPU kaynak kullanımının, bellek kullanımının nasıl yapılacağının gibi bir çok senaryonun farkındadır. Litmus eklenti frameworkü, diğer chaos projelerinin chaos uygulayabilmesi için Litmus'u kullanmasına olanak tanımaktadır. Örneğin pod-delete senaryosu için Pumba veya PowerfulSeal (örnek olarak) kullanılarak Litmus frameworkü aracılığıyla bir chaos grafiği oluşturulabilmektedir.

**Chaos Hub**, kullanılabilir tüm chaos deneylerini barındırır. Chaos grafikleri [hub.litmuschaos.io](https://hub.litmuschaos.io/) adresinde bulunmaktadır. Hub'ın amacı kişilerin CI pipelinelarındaki uygulamalarını test etmek için kullandıkları deneyleri paylaşmak ve yeniden kullanılabilirliği sağlamaktır.

---

Litmus' u Kubernetes ortamınıza dağıtmak isterseniz 2 farklı metotla bunu yapabilirisiniz. 

1- Helm Kullanarak

2- Kubectl ile litmus yaml dosyasını uygulayarak

Öncesinde Kubernetes versiyonunun 1.17 veya üstü olmasına dikkat etmelisiniz. Bir de önerilen 20 GB 'lık bir Persistent Volume (PV)' ye sahip olmalısınız. 

Bu yazımda Helm kullarak kurulumu ele alacağım.

İlk adımda litmus helm repository'i ekleyelim ve repo listi kontorol edelim.

```
helm repo add litmuschaos https://litmuschaos.github.io/litmus-helm/
```
```
helm repo list
```
Daha sonra Litmus' u kurmak istediğimiz namespace'i oluşturalım. Burada litmus adında bir namespace oluşturuyoruz.
```
kubectl create ns litmus
```
Son adımda litmus namespace'i için ChaosCenter'ı kurabiliriz.
```
helm install chaos litmuschaos/litmus --namespace=litmus
```
Çıktı olarak Litmus'u yüklediğinize dair bir teşekkür aldıysanız başarılı bir şekilde kurulumu tamamlamışsınızdır.😀

Litmus Chaos Center'a erişmek ve oturum açmak için kurulumu yaptığınız namespace de oluşan servisleri listeleyebilir ve litmus-frontend-service için kullanılan portu öğrenebilirsiniz.
```
kubectl get svc -n litmus
```
litmus-frontend-service den gerekli bilgileri aldıktan sonra "node_ip:port" şeklinde Chaos Center'a erişebilirsiniz.
```
http://node_ip:port
```
Varsayılan credential bilgileri aşağıdaki gibidir.
	
```
Username: admin
Password: litmus
```
![]({{ 'assets/img/chaos5.png' | relative_url }})
	
Varsayılan olarak owner izinlerine sahip bir projeyle atanırsınız.
	
![]({{ 'assets/img/chaos6.png' | relative_url }})
	
Kurulumu sağlıklı yaptığınızdan emin olmak için podları listeleyebilir ve statülerini kontrol edebilirsiniz. Mongo, frontend, server, auth, workflow gibi dağıtıma ait bir çok podu görebiliyor olmalısınız.
	
```
kubectl get pods -n litmus
```
Şimdi uygulamalarınız için kaotik durumlar oluşturabilir ve çıktıları analiz edebilirsiniz. 😀

![]({{ 'assets/img/chaos7.jpeg' | relative_url }})

	
	
	
	
	
	
---
	
Referanslar:
	
[Litmus Github](https://github.com/litmuschaos/litmus)
	
[Cloud native chaos engineering](https://www.cncf.io/blog/2019/11/06/cloud-native-chaos-engineering-enhancing-kubernetes-application-resiliency/)
	
[PRINCIPLES OF CHAOS ENGINEERING](https://principlesofchaos.org/)
	
[Netflix Kaos Mühendisliği](https://continium.io/en/blog/netflix-kaos-muhendisligi-corona-virusu-ve-benzeri-felaketlere-hazirlikli-olmak/)
