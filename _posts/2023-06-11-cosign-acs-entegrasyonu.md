---
title: Openshift Platformu üzerine ACS (Advanced Cluster Security) Kurulumu ve Cosign
  ile Imaj İmzalama
---

Merhabalar, bu yazımda Openshift platformunda ACS ürününü nasıl kurabileceğinizden ve ACS ürününün yetkinliklerinden bahsedeceğim. Beraberinde Cosign ile nasıl imaj imzalanır ve bu imzalama ACS ile nasıl yönetilir değinmek istiyorum.

![]({{ 'assets/img/rhacsfk.png' | relative_url }})

ACS'i Openshift Platformunuza kurmak için karşınıza iki seçenek çıkıyor
İlki ACS Operatörünü manuel olarak kurabilirsiniz.
İkinci bir seçenek ise helm paketlerini kullanarak deploy edebilirsiniz.
Operatörden nasıl kuracağımıza bakalım.
Operator Hub alanından ACS operatörünü bulabilirsiniz.
