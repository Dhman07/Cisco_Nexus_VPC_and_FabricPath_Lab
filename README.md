# Cisco_Nexus-9k+7k_VPC_and_FabricPath_Lab
تطبيق عملي لبروتوكول vpc وربطه مع بروتوكول fapricpath في بيئه عمليه بأستخدام برنامج EVE-NG 

![image](https://github.com/user-attachments/assets/0dceb8c8-70b0-41f8-8b2c-1e2531987c29)

راح نطبق على اجهزه 9k+7k بعض الاوامر لتهيئة الاجهزة من الضروري القيام بها بعد تشغيل الاجهزه 


`continue with Power On Auto Provisioning (yes/skip/no)[no]: yes

هذا الامر لكي لا تضطر الى ادخال كلمة سر معقدة:

Do you want to enforce secure password standard (yes/no) [y]: no

Enter the password for "admin": admin

Confirm the password for "admin": admin

Would you like to enter the basic configuration dialog (yes/no): no`

بعدها قم بكتابه الأمر
```cisco
switch# dir
```
سوف تظهر لك مجموعة من المسارات فقط يهمك هذا المسار الذي يحدد ملف الاقلاع

```cisco
964875776    Jun 14 10:01:57 2018  nxos.7.0.3.I7.4.bin
```


اكتب هذه الاوامر لتحديد الملف الذي سوف يتم الاقلاع منه و من ثم احفظ التغيرات
```cisco
switch(config)# boot nxos bootflash: nxos.7.0.3.I7.4.bin
switch(config)# copy running-config startup-config
```
سوف يظهر لك هذا العداد الذي يوضح ان عملية النسخ اكتملت

[########################################] 100%


  وقبل الشروع في المهام قم بتفعيل المميزات على الجهزين من نوع NXOS-9k:

![image](https://github.com/user-attachments/assets/14206a54-234c-46ed-aaab-8e3ae572eccc)

#NXOS-1
```cisco
conf t
hostname NXOS-1
feature lacp
feature vpc
```
 #NXOS-2
 ```cisco
conf t
hostname NXOS-2
feature lacp
feature vpc
```

*** تنبية مهم جدا جدا ****

يتم تحديد نوع الاتصال بين أجهزة NXOS اما management او interface او vlan لتفعيل peer-keepalive وهي تختلف في التطبيق بشكل بسيط ولكن النتيجه نفسها ولكن المنافذ هذه تستخدم للاداره ولربط peer-keepalive فقط.



في المهمة الاولى: كيفية تفعيل peer-keepalive بين اجهزة NXOS بأستخدام management:

![image](https://github.com/user-attachments/assets/9ee52a0b-0547-4469-8090-bc12a1dd0d43)


#NXOS-1
```cisco
conf t
vpc domain 1
  role priority 10
  peer-keepalive destination 10.10.10.2 source 10.10.10.1 vrf management
interface mgmt0
  ip address 10.10.10.1/24
```
#NXOS-2
```cisco
conf t
vpc domain 1
  role priority 20
  peer-keepalive destination 10.10.10.1 source 10.10.10.2 vrf management
interface mgmt0
  ip address 10.10.10.2/24
```
كيفية تفعيل peer-keepalive بين اجهزة NXOS بأستخدام interface:

![image](https://github.com/user-attachments/assets/fd58bad4-f5b5-457a-9f38-35e5093d4f41)


#NXOS-1
```cisco
conf t
vrf context AAA
vpc domain 1
  role priority 10
  peer-keepalive destination 10.10.10.2 source 10.10.10.1 vrf AAA

interface Ethernet1/3
  no switchport
  vrf member AAA
  ip address 10.10.10.1/24
  no shutdown
```
#NXOS-2
```cisco
conf t
vrf context AAA
vpc domain 1
  role priority 20
  peer-keepalive destination 10.10.10.1 source 10.10.10.2 vrf AAA

interface Ethernet1/3
  no switchport
  vrf member AAA
  ip address 10.10.10.2/24
  no shutdown
```
كيفية تفعيل peer-keepalive بين اجهزة NXOS بأستخدام vlan:

#NXOS-1
```cisco
conf t
feature interface-vlan
vrf context BBB
vlan 10
vpc domain 1
  role priority 10
  peer-keepalive destination 10.10.10.2 source 10.10.10.1 vrf BBB

interface vlan 10
  vrf member BBB
  ip address 10.10.10.1/24
  no shutdown

interface Ethernet1/3
  switchport access vlan 10
```

#NXOS-2
```cisco
conf t
feature interface-vlan
vrf context BBB
vlan 10
vpc domain 1
  role priority 20
  peer-keepalive destination 10.10.10.1 source 10.10.10.2 vrf BBB

interface vlan 10
  vrf member BBB
  ip address 10.10.10.2/24
  no shutdown

interface Ethernet1/3
  switchport access vlan 10
```
 الاوامر السابقه هي كفيلة بتفعيل peer-keepalive. لتأكد من التفعيل:
 ```cisco
do sh vpc
```

![image](https://github.com/user-attachments/assets/87090064-c58b-4078-b72e-b8c13b73a376)


المهمة الثانية:  كيفية تفعيل peer-link status

![image](https://github.com/user-attachments/assets/0cc62235-4eb2-4c31-9901-bf1dab1c77f5)

#NXOS-1
 ```cisco
interface e1/1-2
  switchport mode trunk
  channel-group 1 mode active
  no shutdown
int po1
  vpc peer-link
  no shutdown

vlan 10
```
#NXOS-2
 ```cisco
interface e1/1-2
  switchport mode trunk
  channel-group 1 mode active
  no shutdown
int po1
  vpc peer-link
  no shutdown

vlan 10
```
هذه الاوامر كفيلة بتفعيل peer-link status
امر vlan ليس له علاقة في تشغيل peer-link status سوف يعمل بدونه ولكن في حالة طلب منك العمل على فيلان رقم 10 او vlan معين حسب المطلوب وفي حالتنا هذه نحن نحتاج vlan عشان بعدها بنفعل بروتوكول fabricpath.


![image](https://github.com/user-attachments/assets/28b6e05a-d4b7-44d1-8a8c-5dce4caeeda9)


المهمة الثالثة: كيفية تفعيل channel-group بين السويتشات و NXOS-K9 يجب الانتباه الى ارقام المنافذ المربوطه من اجهزة NXOS الى Switch :



![image](https://github.com/user-attachments/assets/fa4875ee-ec2a-4f72-add4-26dc0118f44b)


#NXOS-1
 ```cisco
interface Ethernet1/6
  switchport mode trunk
  channel-group 2 mode active
  no shutdown

interface Ethernet1/7
  switchport mode trunk
  channel-group 3 mode active
  no shutdown

int po2
  vpc 2
  no shutdown

int po3
  vpc 3
  no shutdown
```
#NXOS-2
 ```cisco
interface Ethernet1/6
  switchport mode trunk
  channel-group 3 mode active
  no shutdown

interface Ethernet1/7
  switchport mode trunk
  channel-group 2 mode active
  no shutdown

int po2
  vpc 2
  no shutdown

int po3
  vpc 3
  no shutdown
```
#SW1
 ```cisco
interface range Ethernet0/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 2 mode passive
 no sh
```
#SW2
 ```cisco
interface range Ethernet0/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 3 mode passive
 no sh
```

 الاوامر السابقة كفيلة بتفعيل channel-group بين السويتشات و NXOS K9 
 

  ```cisco
do sh vpc
```
 ![image](https://github.com/user-attachments/assets/01e8a881-487c-4a2e-9150-12f95d9135a8)

سنقوم الأن بوضع اعدادات على السوتش المربوطه مع PC وايضا استخدام vlan10 :

#SW1
 ```cisco
vlan 10
interface Ethernet0/2
 switchport access vlan 10
no sh
```
#SW2
 ```cisco
vlan 10
interface Ethernet0/2
 switchport access vlan 10
no sh
```
#PC-5
 ```cisco
ip 192.168.1.1/24
save
```
#PC-6
 ```cisco
ip 192.168.1.2/24
save
```
 
لتاكد من ان جميع الاعدادت صحيحة:
الاوامرالسابقه كفيلة بأن يكون هناك ping بين اجهزة PCs



![image](https://github.com/user-attachments/assets/de7c051c-6dea-412d-a4a2-3f7c271bc6a7)

#حفظ الاعدادات على NXOS 1 and 2 /// SW 1 and 2

#9k
```cisco
copy running-config startup-config
 ```
#Sw
```cisco
wr
 ```
# من هنا يبداء اعداد titanium - 7k

 ![image](https://github.com/user-attachments/assets/7e1c0a7b-daab-44fe-ad4f-5723b5b1c862)
 

 
#الأن سنقوم بربط fabricpath:

اوامر يجب تنفيذها على جميع سويتشات 7k يفضل التسلسل التالي:

#7K 

```cisco
license grace-period
install feature-set fabricpath
feature-set fabricpath
vlan 1-200
  mode fabricpath
  exit
```
اوامر يجب تنفيذه على جميع سويتشات 7k مع تغير الرقم على كل سويتش كالتالي:

```cisco
fabricpath switch-id 1 

fabricpath domain default
  root-priority 254
```
يجب ان يكون لكل سويتش switch-id مختلف مثلا الثاني يكون switch-id 2 و الثالث يكون switch-id 3 وهكذا.

يجب ان يكون لكل سويتش root-priority مختلف مثلا اعلى اولويه يكون root-priority 254 و الذي يلية في الالولوية يكون root-priority 253 والذي يلية يكون root-priority 252 وهكذا.

في المرحلة التالية تطبيق هذه الاوامر على جميع سويتشات k7:
```cisco
interface Ethernet2/7
  switchport
  switchport access vlan 10
  no shutdown

interface Ethernet2/1-6
  switchport
  switchport mode fabricpath
  no shutdown
```

في اخر مرحلة تطبيق trank على interface الذي يربط بين SW2  و 7k والموجود حسب الهيكل وهذا يسمح لربط 7k مع 9k: 


#7K
```cisco
interface Ethernet2/8
  switchport
  switchport mode trunk
  no shutdown
```

  #Sw2
```cisco
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no sh
 ```
ومن ثم اعداد اجهزة PCs

#PC-5
```cisco
ip 192.168.1.3/24
save
```
#PC-6
```cisco
ip 192.168.1.4/24
save
```
#PC-7
```cisco
ip 192.168.1.5/24
save
```
PC-8
```cisco
ip 192.168.1.6/24
save
```
 لا تنسا تحفظ جميع الاعدادات على 7k و Sw2:
#7k
```cisco
copy running-config startup-config
 ```
#Sw2
```cisco
wr
 ```
وفي الختام اتمنى أن أكون قد قدمت الفائده المرجوه. شكرا لكم "_"
