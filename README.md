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

##Case 2 
Kullanılan makine özellikleri şu şekildedir: 
  Rol           |  OS    | IP             | 
  :-----------: | -----: | -----:         |
  Host          | centos | 192.168.79.128 |
  Ansible       | centos | 192.168.79.129 |
```
sudo yum install epel-release
sudo yum install ansible
``` 
vim /etc/ansible/hosts dosyasına aşağıdaki satırları ekliyoruz. 
``` 
[bootcamp]
192.168.79.128 ansible_ssh_user=root ansible_ssh_pass=root 
```
Doğrulama için ise:  

`ansible --list-hosts all`
``` 
 [root@ansible ~]# ansible --list-hosts all
  hosts (1):
    ansible_host=192.168.79.128  
``` 
hostumuza ulaşmayı deneyelim:
```
[root@localhost ~]# ansible -m ping all
192.168.79.128 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

Docker kurulumu için adımlar: 
`ansible-galaxy install geerlingguy.docker`
![image](https://user-images.githubusercontent.com/33395649/115997718-e6530f00-a5ec-11eb-9c2b-183e860b4c7f.png)

Syntax check kullanarak hata var mı diye kontrol ediyoruz. Bu komut, eğer playbook yazımı ise sadece ismini döndürür: `ansible-playbook sample.yml - - syntax – check `

Docker kurulumu için playbook çalıştırılması: 

`ansible-playbook docker.yml`

Wordpress için playbook çalıştırılması: 

`ansible-playbook main.yml` 

Hostumuzda gerekli kontrolleri sağlayalım: 

```

[ceren.taskin@localhost ~]$ sudo docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED        STATUS       PORTS                                   NAMES
d48e47190bb7   mysql:5.7   "docker-entrypoint.s…"   25 hours ago   Up 2 hours   3306/tcp, 33060/tcp                     wordpress_db_1
21c14cf45386   wordpress   "docker-entrypoint.s…"   25 hours ago   Up 2 hours   0.0.0.0:8080->80/tcp, :::8080->80/tcp   wordpress_wordpress_1
[ceren.taskin@localhost ~]$ sudo docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        5.7       87eca374c0ed   5 days ago   447MB
wordpress    latest    c01290f258b3   9 days ago   550MB

```
Containerların ayakta olduğunu görmekteyiz. Wordpress sayfasına erişmeye çalışalım. 

http://192.168.79.128:8080  

![image](https://user-images.githubusercontent.com/33395649/115997910-8dd04180-a5ed-11eb-8ef9-6e63dcab5f13.png)

Nginx: 
```

ansible-galaxy install geerlingguy.nginx
ansible-playbook nginx.yml --syntax –check 
ansible-playbook nginx.yml 
```
Nginx'in konf dosyasını düzenleyeceğiz. 

`sudo vim /etc/nginx/conf.d/nginx.conf`

```
server {
    listen      80;

    location / {
        proxy_pass http://192.168.79.128:8080;
    }

}
```

Nginx'in durumunu kontrol edelim. 

`systemctl status nginx`

```
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset                                                                                                                                   : disabled)
   Active: active (running) since Sun 2021-04-25 17:47:32 EEST; 6s ago
     Docs: http://nginx.org/en/docs/
  Process: 1891 ExecStop=/bin/sh -c /bin/kill -s TERM $(/bin/cat /var/run/nginx.                                                                                                                                   pid) (code=exited, status=0/SUCCESS)
  Process: 3794 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited,                                                                                                                                    status=0/SUCCESS)
 Main PID: 3795 (nginx)
    Tasks: 2
   Memory: 1.5M
   CGroup: /system.slice/nginx.service
           ├─3795 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.c...
           └─3796 nginx: worker process

Apr 25 17:47:32 localhost.localdomain systemd[1]: Starting nginx - high perfo...
Apr 25 17:47:32 localhost.localdomain systemd[1]: Can't open PID file /var/ru...
Apr 25 17:47:32 localhost.localdomain systemd[1]: Started nginx - high perfor...
Hint: Some lines were ellipsized, use -l to show in full.
```

