# Lab_04
# Настройка IPv6-адресов на сетевых устройствах

## Топология

<img width="952" height="128" alt="image" src="https://github.com/user-attachments/assets/c9d162eb-27a5-4ed7-be5a-e475478041dc" />

## Таблица адресации 

| Устройство | Интерфейс | IPv6-адрес         | Link local | Длина префикса | Шлюз по умолчанию |
|------------|-----------|--------------------|------------|----------------|-------------------|
| R1         | G0/0/0    | 2001:db8:acad:a::1 | fe80::1    | 64             | ---               |
| R1         | G0/0/1    | 2001:db8:acad:1::1 | fe80::1    | 64             | ---               |
| S1         | VLAN 1    | 2001:db8:acad:1::b | fe80::b    | 64             | ---               |
| PC-A       | NIC       | 2001:db8:acad:1::3 | SLAAC      | 64             | fe80::1           |
| PC-B       | NIC       | 2001:db8:acad:a::3 | SLAAC      | 64             | fe80::1           |


## Часть 1. Настройка топологии и базовых параметров

### 1.1 Настройка маршрутизатора R1

Router> enable  
Router# configure terminal  
Router(config)# hostname R1  
R1(config)# no ip domain-lookup  
R1(config)# enable secret class  
R1(config)# line console 0  
R1(config-line)# password cisco  
R1(config-line)# login  
R1(config-line)# exit  
R1(config)# line vty 0 4  
R1(config-line)# password cisco  
R1(config-line)# login  
R1(config-line)# exit  
R1(config)# service password-encryption  
R1(config)# banner motd ^CGO OUT NOW!^C  
R1(config)# exit  
R1# write memory  
#### Включение шаблона SDM для IPv6 
S1# configure terminal  
S1(config)# sdm prefer dual-ipv4-and-ipv6 default  
Changes to the running SDM preferences have been stored in NVRAM.  
Please reload the device for the new SDM preferences to take effect.  
S1(config)# end  
S1# reload  

### 1.2 Настройка коммутатора S1  
  
Switch> enable  
Switch# configure terminal  
Switch(config)# hostname S1  
S1(config)# no ip domain-lookup  
S1(config)# enable secret class  
S1(config)# line console 0  
S1(config-line)# password cisco  
S1(config-line)# login  
S1(config-line)# exit  
S1(config)# line vty 0 15  
S1(config-line)# password cisco  
S1(config-line)# login  
S1(config-line)# exit  
S1(config)# service password-encryption  
S1(config)# banner motd ^CGO OUT NOW!^C  
S1(config)# exit  
S1# write memory  

### Ручная настройка IPv6-адресов

#### 2.1 Назначение глобальных IPv6-адресов интерфейсам R1
R1> enable  
R1# configure terminal  
R1(config)# interface g0/0/0  
R1(config-if)# ipv6 address 2001:db8:acad:a::1/64  
R1(config-if)# no shutdown  
R1(config-if)# exit  
R1(config)# interface g0/0/1  
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64  
R1(config-if)# no shutdown  
R1(config-if)# exit  
R1(config)# end  
  
#### Проверка
R1# show ipv6 interface brief  
GigabitEthernet0/0/0   [up/up]  
    FE80::5054:FF:FEXX:XXXX  
    2001:DB8:ACAD:A::1  
GigabitEthernet0/0/1   [up/up]  
    FE80::5054:FF:FEXX:XXXX  
    2001:DB8:ACAD:1::1  

### 2.2 Назначение локальных адресов канала
R1# configure terminal  
R1(config)# interface g0/0/0  
R1(config-if)# ipv6 address fe80::1 link-local  
R1(config-if)# exit  
R1(config)# interface g0/0/1  
R1(config-if)# ipv6 address fe80::1 link-local  
R1(config-if)# end  

#### Проверка
R1# show ipv6 interface g0/0/0 | include link-local  
    Link-local address: fe80::1  
R1# show ipv6 interface g0/0/1 | include link-local  
    Link-local address: fe80::1  

### 2.3 Активация IPv6-маршрутизации на R1
R1# configure terminal  
R1(config)# ipv6 unicast-routing  
R1(config)# end  

#### Проверка групп многоадресной рассылки на интерфейсе G0/0  
R1# show ipv6 interface g0/0/0 | include multicast  
    Joined group address(es): FF02::1, FF02::2, FF02::1:FF00:1  

### 2.4 Настройка интерфейса управления S1 
S1> enable  
S1# configure terminal  
S1(config)# interface vlan1  
S1(config-if)# ipv6 address 2001:db8:acad:1::b/64  
S1(config-if)# ipv6 address fe80::b link-local  
S1(config-if)# no shutdown  
S1(config-if)# end  

#### Проверка  
S1# show ipv6 interface vlan1  
Vlan1 is up, line protocol is up  
  IPv6 is enabled, link-local address is fe80::b  
  Global unicast address(es):  
    2001:DB8:ACAD:1::B, subnet is 2001:DB8:ACAD:1::/64  
  Joined group address(es):  
    FF02::1  
    FF02::2  

### Настройка статистических адресов на ПК
#### PC-A  
IPv6-адрес: 2001:db8:acad:1::3  
  
Длина префикса: 64  
  
Шлюз по умолчанию: fe80::1  

#### PC-B  
IPv6-адрес: 2001:db8:acad:a::3  
  
Длина префикса: 64  
  
Шлюз по умолчанию: fe80::1  

#### Проверка на PC-B до включения IPv6-маршрутизации на R1  
Через командную строку:  
  
C:\> ipconfig  
  
Адаптер Ethernet:  
   IPv6-адрес канала... : fe80::XXXX:XXXX:XXXX:XXXX%13  
   IPv4-адрес........... : 0.0.0.0  
   Маска подсети........ : 0.0.0.0  
   Основной шлюз........ :  
   
   FF02::1:FF00:B  

  Видим только локальный адрес, т.к. мы не активировали юникаст-роутинг на R1  

#### Проверка после включения ipv6 unicast-routing на R1  
  
C:\> ipconfig  
  
Адаптер Ethernet:  
   IPv6-адрес......... : 2001:db8:acad:a:1c5d:8e2a:3f4b:9c7a (предпочтительный)  
   IPv6-адрес канала... : fe80::1c5d:8e2a:3f4b:9c7a%13  
   Основной шлюз...... : fe80::1  
   
R1 начал отправлять Router Advertisement с префиксом 2001:db8:acad:a::/64, и PC-B применил SLAAC.  
  

### 3. Проверка сквозного подключения  
#### 3.1 PC-A  
Работаем через cmd:

C:\> ping fe80::1  
Ответ от fe80::1: время=1 мс  
Ответ от fe80::1: время=1 мс  
Ответ от fe80::1: время=1 мс  
Ответ от fe80::1: время=1 мс  
  
C:\> ping 2001:db8:acad:1::b  
Ответ от 2001:db8:acad:1::b: время=1 мс  
Ответ от 2001:db8:acad:1::b: время=1 мс  
  
C:\> tracert 2001:db8:acad:a::3  
  
  1    <1 мс    <1 мс    <1 мс  2001:db8:acad:1::1  
  2     1 мс     1 мс     1 мс  2001:db8:acad:a::3  

#### 3.2  PC-B

Работаем через cmd:

C:\> ping 2001:db8:acad:1::3  
Ответ от 2001:db8:acad:1::3: время=1 мс  
  
C:\> ping fe80::1  
Ответ от fe80::1: время=1 мс  

### Ответы на вопросы

Мы можем задавать одинаковые локальные адреса на интефейсах, т.к. они уникальны только в пределах одной сети. Интерфейсы G 0/0/0 и G 0/0/1 находятся в разеных широковещательных доменах. Конфликта не возникает.  
Идентификатором подсети в IPv6-адресе 2001:db8:acad::aaaa:1234/64 являются первые 64 бита, т.к. адрес 128 битный.  
Первые 64 бита - идентификатор подсети (сетевая часть) 2001:db8:acad:: , вторые 64- хостовая часть.  
