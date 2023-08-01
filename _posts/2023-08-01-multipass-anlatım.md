---
layout: post
title:  "Multipass Nedir?"
author: hberkayaktas
categories: [ Sanallaştırma, tutorial, Mulltipass ]
image: https://miro.medium.com/v2/resize:fit:716/0*oUkaLxHNJfXxIBVp.png
---

Multipass bir sanallaştıma yazılımıdır. Canonical tarafından geliştirillen yazılım  
Ubuntu imajlarını shell based bir  formda çalıştırmamıza olanak tanır.

## Multipass ile Ubuntu Sanal Makine Yönetimi

**Multipass**, Ubuntu'nun hafif sanal makine yönetim aracıdır. Bu araç, Linux işletim sistemi üzerinde sanal makine örnekleri oluşturmanıza ve yönetmenize olanak tanır. Multipass, hızlı, kolay ve kullanıcı dostu bir arayüze sahiptir ve Ubuntu tabanlı dağıtımlar için harika bir araçtır. Bu makalede, Multipass kullanarak nasıl Ubuntu sanal makine örnekleri oluşturacağınızı, yöneteceğinizi ve silip kaldıracağınızı öğreneceksiniz.

## Multipass'ı Kurma

Multipass'ı kullanmaya başlamak için öncelikle Linux işletim sisteminize Multipass'ı kurmanız gerekmektedir. Bu adım, bir terminal uygulamasını açarak ve aşağıdaki komutu çalıştırarak kolayca yapılabilir:

```
snap install multipass
```

## Multipass ile Ubuntu Sanal Makineleri Oluşturma

Multipass, farklı özelliklere sahip Ubuntu sanal makine örnekleri oluşturmanıza izin verir. Örneklerinizi oluşturmadan önce kullanabileceğiniz imajları görmek için aşağıdaki komutu kullanabilirsiniz:

```
multipass find
```

Örneğin, "focal" adlı bir imaj kullanarak iki ayrı sanal makine oluşturmak istediğinizi düşünelim. Bunları "master" ve "node1" olarak adlandırmak istiyorsunuz. İşte bunu yapmanın iki farklı yöntemi:

1. İlk yöntem, varsayılan imajı kullanarak sanal makine örneklerini oluşturmaktır:

```
multipass launch --name master -c 2 -m 4G -d 10G 
multipass launch --name node1 -c 2 -m 2G -d 10G 
```

2. İkinci yöntem ise, belirli bir imajı ("focal" imajı) kullanarak sanal makine örneklerini oluşturmaktır:

```
multipass launch focal --name master -c 2 -m 4G -d 10G 
multipass launch focal --name node1 -c 2 -m 2G -d 10G 
```

## Multipass ile Sanal Makineye Erişim

Oluşturduğunuz sanal makinelere erişmek için şu komutu kullanabilirsiniz:

```
multipass shell master
```

Bu komut, "master" adlı sanal makineye bir kabuk (shell) oturumu açacaktır. Böylece, sanal makineyi yönetmek için terminale doğrudan erişebilirsiniz. İşiniz bittiğinde, sanal makineden çıkmak için "exit" komutunu kullanabilirsiniz.

## Sanal Makineleri Silme

Artık ihtiyacınız olmayan sanal makine örneklerini silebilirsiniz. Bunun için aşağıdaki komutu kullanabilirsiniz:

```
multipass delete master
```

## Kullanılmayan Sanal Makineleri Temizleme

Multipass, silinen ancak tamamen kaldırılmamış sanal makine örnekleriyle dolabilir. Kullanılmayan örnekleri temizlemek için aşağıdaki komutu kullanabilirsiniz:

```
multipass purge
```

## Sanal Makinelere Paket Güncelleme

Sanal makinelerinizi güncellemek için, aşağıdaki komutu kullanabilirsiniz:

```
multipass exec web-server -- sudo apt update
```

Bu komut, "web-server" adlı sanal makinede paket listesini güncelleyecektir.

## Multipass ile Ağ Yönetimi

Multipass, sanal makinelerinizi farklı ağlarla ilişkilendirmenize izin verir. Sistemdeki mevcut ağları görmek için şu komutu kullanabilirsiniz:

```
multipass networks
```

Belirli bir ağ ile bir sanal makine örneği oluşturmak için ise aşağıdaki komutu kullanabilirsiniz:

```
multipass launch --name master -c 2 -m 4G -d 10G --network Ethernet
```

## Ubuntu Kullanıcısına Şifre Atama

Eğer oluşturduğunuz Ubuntu sanal makinelerinde "ubuntu" kullanıcısına şifre atamak istiyorsanız, aşağıdaki komutu kullanabilirsiniz:

```
sudo passwd ubuntu
```

Bu komutla "ubuntu" kullanıcısının şifresini değiştirebilirsiniz.

---

Bu makalede, Multipass ile Ubuntu sanal makine yönetiminin temel adımlarını öğrendiniz. Artık Multipass'ı kullanarak Ubuntu sanal makinelerinizi oluşturabilir, yönetebilir, silip kaldırabilir ve ağlarla ilişkilendirebilirsiniz. Bir sonraki yazıda bulşuncaya dek kendinize dikkat edin. 