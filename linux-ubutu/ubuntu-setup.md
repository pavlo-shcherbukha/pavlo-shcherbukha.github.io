---
layout: default
title: Настройка linux ubuntu 20.04
permalink: /ubuntu/

---

**v-1**
# Установка ubuntu - опис для початківця

<!-- TOC BEGIN -->
1. [Чому ubuntu](#p01) 
2. [Створення користувача  з правами SUDO](#p02)
3. [ Установка Ubuntu 20.04 GUI-графічного інтерфейсу](#p03)
4. [Установка RDP на UBUNTU (можливысть пыдключення по rdp з windows робочої станції)](#p04)
5. [Прокопування канал для віртуалки в ORACLE CLOUD](#p05)
6. [Установка DOCKER](#p06)
7. [Установка codepage для підримки кирилиці та  локальної timezone](#p07)
<!-- TOC END -->

## 1. <a name="p01">Чому ubuntu</a>

Linux-based  операційні системи зараз присутні у більшості популярних хмарах і на linux-подібних операційних системах побудовані контейнерні платформи.
Ubuntu  легка в установці. Має свій GUI інерфейс. Має широке ком'юніті та на її базі можна  підняти віртуалки  у більшості хмар.
Ну як мінімум віртуальні машини з ubuntu 20.04  присутні в  хмарах IBM, AWS, Oracle, DigitalOcean, AZURE  в різних конфігурація (по кількості процесорів та пам'яті). Ну а в AZURE  на ній навіть Kubernetes кластер можна підняти. Більш того, якщо дозволяють ресурси вашого комп'ютера, то на вашому компі чи laptop  можна поставити віртуалізацію від Microsoft  [Hyper-v](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)  та на неї поставити ту ж таки ubuntu 20.04. Вона займе менше ресурсів, ніж інші операційні системи. Системні вимоги до windows  для установки Hyper-v  показані на [pic-00](#pic-00).  

<kbd><img src="./linux-ubutu/doc/pic-00.png" /></kbd>
<p style="text-align: center;"><a name="pic-00">pic-00</a></p>

А на [pic-022](#pic-022)  показана уже запущена віртуалка на hyper-v з її конфігурацією. Більш того, віртуалка буде підключена до мережі internet і ви зможете спілкуватися і з інтернет і з вашою, так би мовити, host-машиною. Ну, і замість того, щоб інсталювати та деінсталювати різне програмне забезепечення для вивчення та проведеня тестів  на ваш ноутбук, то краще все поставити на віртуалку. Ну, як зламається - то створите нову віртуалку - та і все. З особистого досвіду, то на ubuntu-віртуалку ставив собі docker і в контейнерах підінімав для вивення fluent-d, elastic, kibana. Для windwos - користувача, спершу не звично. Але через деякий час звикаєш, і коли починаеш подорожувати по різних хмарах - то кругом все більш-менш однакове і знайоме.   

<kbd><img src="doc/pic-022.png" /></kbd>
<p style="text-align: center;"><a name="pic-022">pic-022</a></p>


 Більш того, більша частина описаного, підійде і для операційної системи на базі Raspberry PI: [Install Ubuntu
on a Raspberry Pi ](https://ubuntu.com/download/raspberry-pi).   
 Тому, як на мене, то це ідеальна операційна система для вивчення Linex. Тому, далі по тексту, опсані кроки по  налаштуанню OS UBUNTU  для роботи розробника.   

## 2. <a name="p02">Створення користувача  з правами SUDO</a>

Матеріал взято відсіля: [digitalocean create-a-new-sudo-enabled-user-on-ubuntu-20-04-quickstart](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu-20-04-quickstart)

- Підключитсяь по ssh

```bash
ssh root@your_server_ip_address
```
- Виконаи команду створення користувача

в даном випадку username=**psh**

```bash
sudo adduser  psh

```

кругом можна проклікати enter. Пароль в хмарах бажано придумувати складний - обижаються на прості (: . Ну, мають право.


- Додати створеного користувача в групу sudo

```bash
sudo usermod -aG sudo psh

```

- Перевіряємо результати роботи


    * протестувати можна, використовуючи коману  **su**  для перемикання но наовий account користувача:

```bash

su - psh

```
В резульаті буде 

```text
psh@instance-20220409-1955:~$
```

    * Спробуємо виконати команду, що вимагає привілегій sudo

Прочитаємо зміст каталога root, що доступний зазичай root клристувачу

    ```bash

    sudo ls -la /root

    ```

## 3. <a name="p03">Установка Ubuntu 20.04 GUI-графічного інтерфейсу<a>

Матеріал взятий відсіля: [Ubuntu 20.04 GUI installation](https://linuxconfig.org/ubuntu-20-04-gui-installation)


- Обновити пакети ОС та поставити менеджер пакетів **taskel**

```bash

$ sudo apt update
$ sudo apt install tasksel

```

Для перевірки установки виконаємо команду отримання списку пакетів

```bash
tasksel --list-tasks
```

- Установка **ubuntu Desctop**

```bash

sudo tasksel install ubuntu-desktop
```

- Перезавантаження системи

```bash
   reboot
```

- Установка завантаженя GUI за замовчуванням

```bash
$ sudo systemctl set-default graphical.target

```

## 4. <a name="p04">Установка RDP на UBUNTU (можливысть пыдключення по rdp з windows робочої станції)</a>


- Обновити пакети

```bash

sudo apt update

```


- Установка RDP сервера XRDP

Взято відсіль: https://www.youtube.com/watch?v=Moscv2moML8

```bash
  sudo apt install xrdp

```

- Зробити його автостартуючим

```bash
  sudo systemctl enable xrdp
```

- Перевірити роботу xrdp

```bash
sudo systemctl status xrdp
```
Повинно бути  щось отаке.

<kbd><img src="./ubuntu/doc/pic-01.png" /></kbd>
<p style="text-align: center;">pic-1</p>                        

Якщо виявлені помилки (не стартонув), значить скоріше всього не відкрито порт 3389 на сервері.

Для цього потрібно встановити пакет мережевних утиліт.

```bash
   sudo apt install net-tools

```

Перевіряємо дсотупність портів

```bash

netstat -nltp

```

Повинно бути  щось схоже як на pic-02.

<kbd><img src="doc/pic-02.png" /></kbd>
<p style="text-align: center;">pic-2</p>


Якщо порт 3389 не слухається, то потрібно його відкрити шляхом редагування файлу редактором nano

```bash
  sudo nano /etc/iptables/rules.v4

```

та додати рядок, що  на малюнку обведено червоною рамкою

<kbd><img src="doc/pic-03.png" /></kbd>
<p style="text-align: center;">pic-3</p>


про налаштування статичного iptables то читатати по лінку [How to configure iptables on Ubuntu](https://upcloud.com/community/tutorials/configure-iptables-ubuntu/)


В принципі на віртуалці все, але є нуюанс, якщо віртуалка піднята в хмарі то потрібно прокопати канал з вашого компа  в віртуальну приватну мережу. Показую на прикладі ORACLE CLOUD.

## 5. <a name="p05">Прокопування канал для віртуалки в ORACLE CLOUD</a>

Взято відсіль: https://www.youtube.com/watch?v=Moscv2moML8 , але в принципі, якщо ви створюєте віртуалку в хмарі, то вона створюється зразу у віртуальній приватній мережі і там потрібно буде вікрити доступ.


Заходимо на  перелік Virtual Cloud Networks https://cloud.oracle.com/networking/vcns

Заходимо в свою VCN **Virtual Cloud Network**

<kbd><img src="doc/pic-04.png" /></kbd>
<p style="text-align: center;">pic-4</p>


Та вікриваємо меню: "Security List". В нему внесенмо правило для мережі.


<kbd><img src="doc/pic-05.png" /></kbd>
<p style="text-align: center;">pic-5</p>


<kbd><img src="doc/pic-06.png" /></kbd>
<p style="text-align: center;">pic-6</p>


Заходимо в **"Security List"**, що створено за замовчуванням, та вносимо правило для доступу но порта RDP 3389


<kbd><img src="doc/pic-07.png" /></kbd>
<p style="text-align: center;">pic-7</p>


## Підключаємось по RDP  з windos 10

Запускаєм: %windir%\system32\mstsc.exe

або  просто **mstsc.exe**  та вносимо публічний IP вашої терміналки:

<kbd><img src="doc/pic-08.png" /></kbd>
<p style="text-align: center;">pic-8</p>


Ну, а далі підключаємося під root **ubunta**  або під користувачм, що ви створили **psh**

<kbd><img src="doc/pic-09.png" /></kbd>
<p style="text-align: center;">pic-9</p>


## 6. <a name="p06">Установка DOCKER</a>

Вілціяля:

- Update the apt package index and install packages to allow apt to use a repository over HTTPS:

 ```bash
 sudo apt-get update

 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

 ```

- Add Docker’s official GPG key:

```bash

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
- Use the following command to set up the stable repository. To add the nightly or test repository, add the word nightly or test (or both) after the word stable in the commands below.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null



```

- Install Docker Engine

```bash
sudo apt-get update

 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

- Verify that Docker Engine is installed correctly by running the hello-world image.

```bash
 sudo docker run hello-world

```

## 7. <a name="p07">Установка codepage для підримки кирилиці та  локальної timezone</a>

- отримати локаль

```bash
$locale -  всі локалі
$locale -a  Display  a  list  of  all  available   locales
$locale -a -v
$locale -a -v  -Display Detail info
locale: en_US           archive: /usr/lib/locale/locale-archive
-------------------------------------------------------------------------------
    title | English locale for the USA
   source | Free Software Foundation, Inc.
  address | http://www.gnu.org/software/libc/
    email | bug-glibc-locales@gnu.org
 language | American English
territory | United States
 revision | 1.0
     date | 2000-06-24
  codeset | UTF-8

locale: en_US.utf8      archive: /usr/lib/locale/locale-archive
-------------------------------------------------------------------------------
    title | English locale for the USA
   source | Free Software Foundation, Inc.
  address | http://www.gnu.org/software/libc/
    email | bug-glibc-locales@gnu.org
 language | American English
territory | United States
 revision | 1.0
     date | 2000-06-24
  codeset | UTF-8

locale: C.UTF-8         directory: /usr/lib/locale/C.UTF-8
-------------------------------------------------------------------------------
    title | C locale
    email | aurel32@debian.org
 language | C
 revision | 1.6
     date | 2016-08-08
  codeset | UTF-8



$locale -c charmap  Display  the  available charmap

```

Для настройки на ураїнську  установити пакет:

```bash
  sudo dpkg-reconfigure locales 

```
та вибрати локаль  **uk_UA.UTF-8.**

Установити часову зону:

``` bash
  timedatectl list-timezones  -  отримати список

  sudo timedatectl set-timezone  Europe/Kiev   установити часову зону киэва

  timedatectl - отримати поточну  часову зону
```
