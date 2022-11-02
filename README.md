# Ubuntu 22.04 Sunucuya Nasıl "Static IP" Verilir ?

## Mevcut ip'mizi ve interface'imizi görüntülemek için ip a komutunu çalıştırıyoruz.

<pre><code>
haeshin@worker-ubuntu-2204-k8s:/etc/netplan$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:7c:46:1f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.26/24 metric 100 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 84618sec preferred_lft 84618sec
    inet6 fe80::a00:27ff:fe7c:461f/64 scope link 
       valid_lft forever preferred_lft foreve
</pre></code>

## Static IP için düzenleme /etc/netplan altındaki config dosyasında yapılacak.Buraya gidip dosyaları listeleyelim.Değiştirilmesi gereken config dosyamız >>> 00-installer-config.yaml

<pre><code>
haeshin@worker-ubuntu-2204-k8s:~$ cd /etc/netplan
haeshin@worker-ubuntu-2204-k8s:/etc/netplan$ ls
00-installer-config.yaml
</pre></code>

## Şimdi düzenleyelim.

<pre><code>
haeshin@worker-ubuntu-2204-k8s:/etc/netplan$ sudo vi 00-installer-config.yaml 
</pre></code>

## Dosyanın içeriğini kendi ortamınıza uygun biçimde düzenlemeniz gerekiyor.

- enp0s3 >>> interface adı
- addresses >>> sunucuya vermek istediğiniz ip
- nameservers >>> ulaşılabilir olmalı aşağıdaki örnekte google sunucularının ip'leri verildi
- routes >>> gateway 

<pre><code>
network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            addresses:
                - 192.168.1.26/24
            nameservers:
                addresses: [8.8.8.8, 8.8.4.4]
            routes:
                - to: default
                  via: 192.168.1.1
 </pre></code>
                  
## Dosyayı düzenledikten sonra aşağıdaki komut ile değişiklikleri uyguluyoruz.

<pre><code>
sudo netplan apply 
</pre></code>
 
## NOT: Gateway ip'nizi aşağıdaki komutla görebilirsiniz.

<pre><code>
haeshin@master-ubuntu-2204-k8s:/etc/netplan$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 enp0s3
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 enp0s3
192.168.1.1     0.0.0.0         255.255.255.255 UH    100    0        0 enp0s3
</pre></code>

### Bu komutu çalıştırabilmek için net-tools paketlerinin kurulu olması gerekiyor. Aşağıdaki komut ile paketleri kurabilirsiniz.

<pre><code>
sudo apt install net-tools
</pre></code>
