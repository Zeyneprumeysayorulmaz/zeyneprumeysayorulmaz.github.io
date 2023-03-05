---
title: Kubernetes Notlarım VII- Resource Limits
---

> Merhabalar Kubernetes yazı serisine hız kesmeden devam ediyorum. Bu yazımda resource limits ve environment variable konularına değineceğim. Keyifli okumalar..

## Resource Limits
Containerlar aksi belirtilmedikçe varsayılan olarak üstünde çalıştıkları hostun cpu ve memory kaynaklarına sınırsız şekilde erişebilirler. Bu durumda özellikle kubernetes gibi yüzlerce containerların çalıştığı dağıtık yapılarda bir sorun oluşturur.
Eğer bir pod üzerinde çalıştığı hostun tüm kaynaklarını tüketiyorsa aynı host üzerinde çalışan diğer podlar için sıkıntılı bir durum teşkil eder, diğer podlar işlerini düzgün yapamaz hale gelir. Bu nedenle podların içinde çalışan containerlar için cpu ve memory limitlerini kısıtlamamız gerekir. Pod tanımlarında kullanılan request ve limit anahtarları bizlere bu imkanı verir.  

* **CPU**

Kubernetes üzerinde cpu kısıtlamaları;

Her worker node'un belirli cpu kaynakları mevcuttur. Bu kaynaklar cloud üstünde ya da sanal olarak çalışan sistemlerde **vCPU** olarak, baremetal sunucularda ise **hyperthread** olarak hesaplanır. 

Örneğin aşağıdaki gibi bir tanımda, bu podun çalıştığı node üzerindeki corelardan sadece 1 tanesini kullanabileceği anlamına gelir.
![]({{ 'assets/img/resourcecpu.PNG' | relative_url }})

Cpu tanımlarında kesirli değerler kullanabiliriz. 0,5 CPU talep eden bir container, 1CPU talep eden bir container'ın yarısı kadar cpu alır demektir. Mili anlamında m son eki kullanılabilir. 1m'den daha ince hassasiyetin tanımlanmasına izin verilmez.

CPU her zaman mutlak bir miktar olarak talep edilir,  göreli bir miktar olarak talep edilmez; 0,1 tek çekirdekli, çift çekirdekli veya 48 çekirdekli bir makinede de aynı miktarda CPU'dur.

1 CPU Core = cpu: "1" = cpu: "1000" = cpu: "1000m"

1 CPU Core!un %10'u = cpu: "0,1" = cpu: "100" = cpu: "100m"

* **Memory**

Kubernetes üzerinde memory kısıtlamaları;

Containerların kullanacağı max memory miktarını byte cinsinden tanımlayabilirsiniz.  Clusterda mevcut durumda kullanılabilir bellek varsa, bir container belirlenen bellek isteğini aşabilir. Ancak bir container bellek sınırından fazlasını kullanmasına izin verilmez. Container limitlerin üstünde bellek tüketmeye  devam ederse, container sonlandırılır ve kubelet onu yeniden başlatır.

![]({{ 'assets/img/resourcelimits.PNG' | relative_url }})

**Requests altında tanımladığımız değerler;**

Kubernetes bu podu oluşturmaya başladığı zaman scheduler bu podun oluşturulacağı bir node seçecektir. Bu seçme aşamasında kubernetes'e diyoruz ki bu podu en az  request altında verdiğim değerler kadar kaynak olan bir node üzerinde schedule et. Kısacası bu podun oluşturulabilmesi için minimum ne kadar boş kaynağın olması gerektiğini belirtiyoruz.

**Limits altında tanımladığımız değerler ise;**

Bu container'ın en fazla kullanabileceği sistem kaynağını belirtir. Eğer container limit altında verilen  cpu değerinden fazlasına erişmeye çalışırsa mümkün olmayacaktır yani erişemeyecektir.

Fakat memory limitinde durum biraz farklıdır, memory allocation CPU gibi çalışmaz. Container limit altında verilen memory değerinden daha fazlasını sistemden talep edebilir ve sistem bunu varsayılan olarak engelleme şansına sahip değildir. Container limit memory değerine geldiğinde ve daha fazlasını allocate edebileceği bir mekanizma olmadığından bu duruma gelindiğinde pod "**OOMKilled"** yani **"Out of Memory Killed"** durumuna geçerek restart edilir. Bu nedenle eğer limit altında verilen memoryden fazlası container tarafından talep edilirse pod restart edilecektir. Tüm bunlar düşünüldüğünde limit kısıtları uygulamanın davranışına uygun olarak belirlenmelidir. 

![]({{ 'assets/img/oomkilled.gif' | relative_url }})


## Environment Variables
Örneğin bir web uygulamamız var ve bu web uygulaması bir db'ye bağlanarak verilerini burada saklıyor olsun. Bu uygulamanın bağlandığı veritabanı sunucu adresi ve kullanıcı bilgileri gibi bilgileri uygulama içine hard-coded olarak gömdük diyelim. Daha sonra bu uygulamayı container imajı haline getirdik. Bu senaryoda bir kaç sıkıntılı durum var. Veritabanı bağlantı bilgilerini container imajı içerisinde hard-coded olarak yazdık, bu güvenlik anlamında best practice değildir.  Bir diğer sıkıntı da bu imajdan container oluşturmak istediğim zaman bu container her defasında aynı veritabanına aynı user ve şifre ile bağlanacak, bu senaryoda veritabanı adresim test ve prod ortamlarda ayrı olabilir veya bu user ve şifre bilgilerini zaman içersinde güncellemiş olabiliriz.

İşte bu tarz ortama göre değişebilen veya hassas bilgileri container içerisine gömmek sağlıklı bir yol değil.  Bunun yerine uygulamalarımızda bunları çalıştırdıkları sistemden okuyacakları şekilde bir değişken olarak tanımlarız yani environment variable olarak tanımlarız.

Örneğin yukarıdaki veritabanı senaryosunu baz alacak olursak. Bu veritabanı bilgilerini değişkenlerin değerlerine bakacak şekilde ayarlarız.

Ne zaman bu imajdan bir container oluşturmak istersek o an hangi değerler atanması gerekiyorsa bu değerleri containerda environment variable olarak tanımlarız.

Environment variables containers: altında env: anahtarıyla tanımlanır.  Env altında list şeklinde tüm environmentlar tanımlanır.
![]({{ 'assets/img/env.PNG' | relative_url }})


Kısaca resource limits ve environment variable konularına değinmeye çalıştım Umarım faydalı olmuştur. 😊

Bir sonraki yazımda görüşmek üzere 👋
