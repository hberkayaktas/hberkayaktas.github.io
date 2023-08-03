---
layout: post
title:  "Kubeadm ile Kubernetes'in Ubuntu 22.04'e Kurulumu"
author: hberkayaktas
categories: [ tutorial ]
---

Kubeadm ile Kubernetes Kurulumu

# Kubeadm ile Kubernetes'in Ubuntu 22.04'e Kurulumu

Önkoşullar
Bu kılavuzda, bir ana düğüm ve iki çalışan düğüm kullanıyoruz. Aşağıda, her düğümdeki sistem gereksinimleri verilmiştir,

- Minimum kurulum Ubuntu 22.04
- En az 2GB RAM veya daha fazlası
- Minimum 2 CPU çekirdeği / veya 2 vCPU
- /var veya üzerinde 20 GB boş disk alanı
- Yönetici haklarına sahip Sudo kullanıcısı
- Her düğümde İnternet bağlantısı

Laboratuvar Kurulumu

- Master Node: 192.168.1.173 – k8smaster.example.net
- İlk Worker Node: 192.168.1.174 – k8sworker1.example.net
- İkinci Worker Node: 192.168.1.175 – k8sworker2.example.net

## 1) Her Düğümde ana bilgisayar adını ayarlayın

Ana düğümde oturum açın ve hostnamectl komutunu kullanarak ana bilgisayar adını ayarlayın,

```bash
$ sudo hostnamectl set-hostname "k8smaster.example.net"
$ exec bash
```

Çalışan düğümlerde çalıştırın

```bash
# 1st worker node
$ sudo hostnamectl set-hostname "k8sworker1.example.net"   

# 2nd worker node
$ sudo hostnamectl set-hostname "k8sworker2.example.net"   
$ exec bash
```

Her düğümde /etc/hosts dosyasına aşağıdaki domainleri tanımlayın.

```yaml
192.168.1.173 k8smaster.example.net k8smaster
192.168.1.174 k8sworker1.example.net k8sworker1
192.168.1.175 k8sworker2.example.net k8sworker2
```

## 2) Swap'i Kapatma & Kernel Parametrelerini Ekleme

Takas işlemini devre dışı bırakmak için swapoff ve sed komutunun altında yürütün. Tüm düğümlerde aşağıdaki komutları çalıştırdığınızdan emin olun.

```bash
$ sudo swapoff -a
$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Aşağıdaki çekirdek modüllerini tüm düğümlere yükleyin,

```bash
$ sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

Kubernet'ler için aşağıdaki Çekirdek parametrelerini ayarlayın, tee komutunun altında çalıştırın

```bash
$ sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF 
```

Yukarıdaki değişiklikleri yeniden yükleyin, çalıştırın

```bash
$ sudo sysctl --system
```

## 3) Containerd Runtime'ı Kurun

Bu kılavuzda, Kubernetes kümemiz için containerd runtime kullanıyoruz. Bu nedenle, containerd'yi kurmak için önce bağımlılıklarını kurun.

```bash
$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

Docker deposunu etkinleştir

```bash
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Şimdi, containerd'yi yüklemek için aşağıdaki apt komutunu çalıştırın.

```bash
$ sudo apt update
$ sudo apt install -y containerd.io
```

Containerd'yi, systemd'yi cgroup olarak kullanmaya başlayacak şekilde yapılandırın.

```bash
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Containerd hizmetini yeniden başlatın ve etkinleştirin

```bash
$ sudo systemctl restart containerd
$ sudo systemctl enable containerd
```

## 4) Kubernetes'ler için Apt Deposu Ekleyin

Kubernetes paketi, varsayılan Ubuntu 22.04 paket havuzlarında mevcut değildir. Bu yüzden kubernet depoları eklememiz, aşağıdaki komutu çalıştırmamız gerekiyor,

```bash
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg
$ sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

`Not:` Bu kılavuzu yazarken, Xenial en son Kubernetes deposudur ancak depo Ubuntu 22.04 (Jammy Jellyfish) için kullanılabilir olduğunda, xenial kelimesini 'apt-add-repository' komutunda 'jammy' ile değiştirmeniz gerekir.

## 5) Kubectl, Kubeadm ve Kubelet'i kurun

Depoları ekledikten sonra tüm düğümlere kubectl, kubelet ve Kubeadm yardımcı programı gibi Kubernetes bileşenlerini kurun. Aşağıdaki komut dizisini yürütün,

```bash
$ sudo apt update
$ sudo apt install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

## 6) Kubernetes Kümesini Kubeadm ile Başlatın (Master için)

Artık hepimiz Kubernetes kümesini başlatmaya hazırız. Aşağıdaki Kubeadm komutunu yalnızca ana düğümde çalıştırın.

```bash
$ sudo kubeadm init --control-plane-endpoint=k8smaster.example.net
```

Yukarıdaki komutun çıktısı,

![kubeadm-init]({{ site.baseurl }}/assets/post_images/Kubeadm-initialize-kubernetes-ubuntu-22-04-768x422.jpeg)

Başlatma tamamlandıktan sonra, çalışan düğümlerin kümeye nasıl katılacağına ilişkin talimatları içeren bir mesaj göreceksiniz. İleride başvurmak için kubeadm birleştirme komutunu not edin.(kubeadm join ....)

Bu nedenle, küme ile etkileşime başlamak için ana düğümde aşağıdaki komutları çalıştırın,

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

ardından, küme ve düğüm durumunu görüntülemek için aşağıdaki kubectl komutlarını çalıştırmayı deneyin

```bash
$ kubectl cluster-info
$ kubectl get nodes
```

Çıktı,

![kubeadm-cluster-info]({{ site.baseurl }}/assets/post_images/Initial-k8s-cluster-information-768x123.jpeg)

## 7) Worker Nodeları Clustere Ekleme

Her çalışan düğümde, 6. adımda ana düğümü başlattıktan sonra daha önce not aldığınız kubeadm birleştirme komutunu kullanın. Bunun gibi bir şey görünmelidir:

```bash
$ sudo kubeadm join k8smaster.example.net:6443 --token vt4ua6.wcma2y8pl4menxh2 \
   --discovery-token-ca-cert-hash sha256:0494aa7fc6ced8f8e7b20137ec0c5d2699dc5f8e616656932ff9173c94962a36
```

Her iki çalışan düğümden çıktı,

![Woker1-Join]({{ site.baseurl }}/assets/post_images/Woker1-Join-kubernetes-Cluster-768x262.jpeg)

![Woker2-Join]({{ site.baseurl }}/assets/post_images/Woker2-Join-kubernetes-Cluster-768x261.jpeg)

Çalışan düğümlerden alınan yukarıdaki çıktı, her iki düğümün de kümeye katıldığını doğrular. Kubectl komutunu kullanarak ana düğümden düğümlerin durumunu kontrol edin,

```bash
$ kubectl get nodes
```

![Node-Status]({{ site.baseurl }}/assets/post_images/Node-Status-K8s-Before-CNI-768x137.jpeg)

Gördüğümüz gibi, düğümlerin durumu ' Hazır Değil ', yani onu aktif hale getirmek için. CNI (Container Network Interface) veya Calico, Flannel ve Weave-net gibi ağ eklentileri kurmalıyız.

## 8) Calico Ağ Eklentisini Kurun

Kümedeki bölmeler arasında iletişimi etkinleştirmek için bir ağ eklentisi gerekir. Ana düğümden Calico ağ eklentisini yüklemek için aşağıdaki kubectl komutunu çalıştırın,

```bash
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

Yukarıdaki komutların çıktısı aşağıdaki gibi görünecektir,

![Install-Calico]({{ site.baseurl }}/assets/post_images/Install-Calico-Network-Add-on-k8s-768x477.jpeg)

Kube-system namespacedeki podların durumunu kontrol edelim,

```bash
$ kubectl get pods -n kube-system
```

Çıktı

![kubesys]({{ site.baseurl }}/assets/post_images/Kube-System-Pods-after-calico-installation-768x278.jpeg)

Mükemmel, düğümlerin durumunu da kontrol edin.

```bash
$ kubectl get nodes
```

![node-status]({{ site.baseurl }}/assets/post_images/Nodes-Status-after-Calico-Network-Add-on-768x143.jpeg)

Harika, yukarıdaki resim düğümlerin aktif düğüm olduğunu onaylıyor. Artık Kubernetes clusterımızın çalıştığını söyleyebiliriz. Başarı ile kubernetes cluster kurulumunu kudeadm ile gerçekleştirmiş olduk.

Bu bir çeviri yazıdır  kaynakça da çeviri kaynağı verilmiştir

kaynak:

[https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/](https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/)
