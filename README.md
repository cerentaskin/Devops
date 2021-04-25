# DevOps
## Case 1
RedHat dağıtımlarından Centos 7 minimal iso dosyası indirilir ve sanallaştırma yazılımları üzerinden kurulum yapılır. Kurulum sırasında “network” ayarlarının yapılandırılması önemlidir.  Bu senaryoda VMWare üzerinden kurulum yapılmıştır. 
#### 1.1
Güncellemeler için: 
`yum update –y`
#### 1.2
Kişisel kullanıcı oluşturmak için (bu senaryoda ceren.taskin) şeklinde olacak:
`adduser ceren.taskin`  
Bu kullanıcının parolasını değiştirmek için ise: 
`passwd ceren.taskin`

Kullanıcımızı yetkili gruba atayabilmek için: 
`usermod -a -G wheel ceren.taskin`
Buradaki Wheel grubundaysanız sudo komutuyla işlem yapabilirsiniz. Sudo çalıştırıldığında, çalıştıran kişi anlık olarak uid’si 0 olan kullanıcı olur. (özel gruba dahil) 
/etc/shadow kullanıcı bilgilerinin tutulduğu yerdir. Aşağıdaki komutun çıktısında kullanıcı ile ilgili bilgilere ulaşılabilir. Burada kullanıcının parolası “şifrelenmiş” bir şekilde tutulmaktadır. Aşağıdaki satırdan şifrelenme metodu, parolanın son değiştirilme tarihi (1 Ocak 1970’den itibaren geçen gün sayısından hesaplanır) vb. bilgilere ulaşabiliriz. 
`cat /etc/shadow | grep ceren.taskin` 
```

[ceren.taskin@localhost ~]$ sudo cat /etc/shadow | grep ceren.taskin
[sudo] password for ceren.taskin:
ceren.taskin:$6$Tg7HO/Z4$wd4WVfB28Y3KDFC4G41Es3s8TgeDuh4xwlzXXy5l0AbKiesKVJmxif/BvCdeQXmAHNPZZI2xSKgnf.p9rALOa0:18741:0:99999:7:::
```

#### 1.3
VMWare ayarlarından disk eklendikten sonra `lsblk` komutuyla diskin geldiği görülebilir: (sdb)
```

[ceren.taskin@localhost ~]$ sudo lsblk
[sudo] password for ceren.taskin:
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda 8:0 0 20G 0 disk
├─sda1 8:1 0 1G 0 part /boot
└─sda2 8:2 0 19G 0 part
├─centos-root 253:0 0 17G 0 lvm /
└─centos-swap 253:1 0 2G 0 lvm [SWAP]
sdb 8:16 0 10G 0 disk
sr0 11:0 1 973M 0 rom
```

 `sudo mkfs.ext4 /dev/sdb` komutuyla disk formatlanır. Sonrasında /bootcamp dizini oluşturulur ve disk buraya bağlanır. 

 ```
 sudo mkdir /bootcamp  

 sudo mount /dev/sdb /bootcamp 
 ```
 ```
[ceren.taskin@localhost ~]$ sudo lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda 8:0 0 20G 0 disk
├─sda1 8:1 0 1G 0 part /boot
└─sda2 8:2 0 19G 0 part
├─centos-root 253:0 0 17G 0 lvm /
└─centos-swap 253:1 0 2G 0 lvm [SWAP]
sdb 8:16 0 10G 0 disk /bootcamp
sr0 11:0 1 973M 0 rom
```

#### 1.4
/opt altında bootcamp dizini `mkdir` komutu ile oluşturulduktan sonra içinde bootcamp.txt dosyası oluşturulup vim ile düzenlenmiştir. 
 ```
sudo touch bootcamp.txt
sudo vim bootcamp.txt
cat bootcamp.txt
```
 ```

[ceren.taskin@localhost bootcamp]$ sudo touch bootcamp.txt
[sudo] password for ceren.taskin:
[ceren.taskin@localhost bootcamp]$ sudo vim bootcamp.txt
[ceren.taskin@localhost bootcamp]$ cat bootcamp.txt
Merhaba Trendyol
```


#### 1.5 
 ```
sudo mv $(sudo find / -iname bootcamp.txt -type f) /bootcamp
cat /bootcamp/bootcamp.txt
 ```
```

[ceren.taskin@localhost bootcamp]$ sudo mv $(sudo find / -iname bootcamp.txt -type f) /bootcamp
[ceren.taskin@localhost bootcamp]$ cat /bootcamp/bootcamp.txt
Merhaba Trendyol
```

