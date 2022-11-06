---
title: Kubernetes Notlarım III- Kubernetes Objeleri | Label | Selector | Namespace
---

Merhabalar, Kubernetes Notlarım yazı serisine devam ediyoruzzz. Serinin üçüncü  yazısı için keyifli okumalar 🥳

### Label ve Selector
##### Etiket (label) ve etiket seçme (selector)
Etiketler/tagler kubernetes'de her türlü objeye atayabildiğimiz anahtar değer eşleniklerdir. Oluşturduğumuz objeye bizlerin daha iyi anlamdırabileceği gruplama yaparken kullanabileceğimiz bilgiler eklemiş oluyoruz.  Obje oluştururken veya oluşturulduktan sonra etiket tanımı yapabiliriz. Bir kaç yazım kuralı var, yine kubernetes sokümanlarından bakılabilir ezbere ne gerek :)

![]({{ 'assets/img/abstraction-labels.png' | relative_url }})

***Önemli:*** Kubernetes'de objeler arası bağlantı etiketler aracılığıyla gerçekleştirilir. Örneğin service ve deployent objeleri hangi podların onlara ait olduğunu ve hangi podlarla ilişki kuracaklarını etiketler sayesinde belirlerler.
