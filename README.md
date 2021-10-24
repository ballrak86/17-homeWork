## Описание файлов в директории
logFileFull.log - полный лог выполнения  

Vagrant_folder - все что понадобится для поднятия VM и краткое описание файлов в ней  
Vagrantfile - вагрант файл  

## Теоритическая часть
```
**Сеть office1**
- 192.168.2.0/26 - dev 
Network:   192.168.2.0/26
Broadcast: 192.168.2.63
HostMin:   192.168.2.1
HostMax:   192.168.2.62
Hosts: 62
- 192.168.2.64/26 - test servers 
Network:   192.168.2.64/26
Broadcast: 192.168.2.127
HostMin:   192.168.2.65
HostMax:   192.168.2.126
Hosts: 62
- 192.168.2.128/26 - managers
Network:   192.168.2.128/26
Broadcast: 192.168.2.191
HostMin:   192.168.2.129
HostMax:   192.168.2.190
Hosts: 62
- 192.168.2.192/26 - office hardware
Network:   192.168.2.192/26
Broadcast: 192.168.2.255
HostMin:   192.168.2.193
HostMax:   192.168.2.254
Hosts: 62

**Сеть office2**
- 192.168.1.0/25 - dev 
Network:   192.168.1.0/25
Broadcast: 192.168.1.127
HostMin:   192.168.1.1
HostMax:   192.168.1.126
Hosts: 126
- 192.168.1.128/26 - test servers
Network:   192.168.1.128/26
Broadcast: 192.168.1.191
HostMin:   192.168.1.129
HostMax:   192.168.1.190
Hosts: 62
- 192.168.1.192/26 - office hardware 
Network:   192.168.1.192/26
Broadcast: 192.168.1.255
HostMin:   192.168.1.193
HostMax:   192.168.1.254
Hosts: 62

**Сеть central**
- 192.168.0.0/28 - directors 
Network:   192.168.0.0/28
Broadcast: 192.168.0.15
HostMin:   192.168.0.1
HostMax:   192.168.0.14
Hosts: 14       

- 192.168.0.32/28 - office hardware
Network:   192.168.0.32/28
Broadcast: 192.168.0.47
HostMin:   192.168.0.33
HostMax:   192.168.0.46
Hosts: 14 

- 192.168.0.64/26 - wifi
Network:   192.168.0.64/26
Broadcast: 192.168.0.127
HostMin:   192.168.0.65
HostMax:   192.168.0.126
Hosts: 62

**Свободные подсети**

- 192.168.0.16/28 - Свободна
- Broadcast: 192.168.0.31

- 192.168.0.48/28 - Свободна
- Broadcast: 192.168.0.63

- 192.168.0.128/25 - Свободна
- Broadcast: 192.168.0.255
```
## Практическая часть  
## Описание как запустить виртуальную машину (кратко)
Поднять ВМ и подключиться к ВМ office1Server
```
vagrant up
vagrant ssh office1Server
```
Выполнить команды ниже чтобы проверить что другие сервера сети доступны, а так же доступны внешние ресурсы и маршрут проходит через inetRouter  
192.168.1.130 - office2Server  
192.168.0.2 - centralServer
```
ping -c 1 192.168.1.130
ping -c 1 192.168.0.2
ping -c 1 google.com
tracepath google.com
```

# VagrantFile только важное
Все router настроенны по аналогии. Настроены интерфейсы выбраны адреса из диапазона и выполнены изменения в конфигурации.  
net.ipv4.conf.all.forwarding - Включение форвардинга пакетов. Иными словами, необходимо разрешить ядру операционной системы осуществлять проброс трафика с одного интерфейса на другой.  
net.ipv4.conf.all.proxy_arp - Включает проксирование arp-запросов для заданного интерфейса. ARP-прокси позволяет маршрутизатору отвечать на ARP запросы в одну сеть, в то время как запрашиваемый хост находится в другой сети.  
Отключаем присвоение шлюза на интерфейсе eth0 по dhcp.  
Настраиваем шлюз на нужном интерфейсе и перезагружаем ВМ чтобы все настройки вступили в силу.
```
when "centralRouter"
  box.vm.provision "shell", run: "always", inline: <<-SHELL
    echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.d/99-sysctl.conf
    echo "net.ipv4.conf.all.proxy_arp=1" >> /etc/sysctl.d/99-sysctl.conf
    echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
    echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
    reboot
    SHELL
```
добавляем маршруты inetRouter чтобы он знал куда отправлять пакеты в локальную сеть  
192.168.255.2 - centralRouter
```
ip r add 192.168.0.0/24 via 192.168.255.2
ip r add 192.168.1.0/24 via 192.168.255.2
ip r add 192.168.2.0/24 via 192.168.255.2
```

📚Домашнее задание/проектная работа разработано(-на) для курса ["Administrator Linux. Professional"](https://otus.ru/lessons/linux-professional/)