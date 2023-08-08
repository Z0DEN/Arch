# Установка Arch linux

## (После загрузки boot flash)

>## 1. Подключаемся к интернету

#### Если он проводной подключение происходит автоматически. Также можно самостоятельно сконфигурировать сетевое соединение.
## Подключаемся к точке доступа wifi:

---

#### Cмотрим название сетевого интерфейса: 
     
```
ip a
```
#### Например - `enp4s0`

#### Возможно wifi адаптер заблокирован в системе на программном уровне, разблокируем его:
```
rfkill unblock wifi
```

#### Затем включаем wifi адаптер командой

```
ip link set {ИМЯ_ИНТЕРФЕЙСА} up
```

#### Например: 

```
ip link set wlan0 up
```

#### Запускаем утилиту управления беспроводными сетями:

```
iwctl
```

#### Подключаемся к нашей точке доступа:

```
station {ИМЯ_ИНТЕРФЕЙСА} connect {ИМЯ_СЕТИ}
```

#### Затем вводим пароль точки доступа, если его нет приконнектится сразу

#### Выходим из утилиты:

```
exit
```

#### Проверяем наличие интернет соединения, пропингуем Гугл:

```
ping google.com
```

#### Иначе можно говорить о проблеме с днс серверами

#### Если все таки такое случилось их можно добавит самому отредактировав специальный конфиг:

```
nano /etc/resolv.conf
```

#### Добавить:

```
nameserver 8.8.8.8  
nameserver 8.8.4.4
```
>## 2. Разметка диска

#### Воспользуемся утилитой `fdisk`

```
fdisk {ПУТЬ_К_ДИСКУ}
```

#### Например:

```
fdisk /dev/sda
```

#### Узнаем есть ли на диске разделы:

```
p  
```

#### удалим имеющиеся разделы:

```
d
```

#### создадим новую таблицу разделов `GPT`:

```
g
```

#### Сохраним изменения:

````
w
````

#### Разобьём диск на разделы утилитой `cfdisk`:

```
cfdisk {ПУТЬ_К_ДИСКУ}
```

- ### Cоздадим раздел bios boot на 100 мегабайт:

#### Нажимаем `[NEW]`

#### Вводим `[100M]`

#### Выбираем `[TYPE]`:  BIOS boot

- ### Создадим раздел для ядер системы, размером от 300 до 500 мегабайт:

#### Нажимаем вниз `↓`

#### `[NEW]`

#### `[300M]`

#### `[TYPE]`: EFI System

#### Нажимаем вниз `↓`

- ### Создадим корневой раздел под операционную систему на оставшееся место:

#### `[NEW]`

#### `[enter]`

#### `[WRITE]`

#### `[yes]`

#### `[QUIT]`

#### Проверим разделы:
```
fdisk -l
```
### Форматируем все разделы **КРОМЕ BIOS boot**:

- #### 2-й раздел (EFI System) форматируем в `FAT32`:
```
mkfs.vfat {ПУТЬ_ДО_РАЗДЕЛА}
```
- #### 3-й раздел (Linux filesystem) форматируем в `btrfs`:
```
mkfs.btrfs -f 
```
>## 3. Монтируем разделы в `/mnt`

- ### Корневой: 
```
mount {ПУТЬ_ДО_РАЗДЕЛА} /mnt
```
- ### Загрузочный **`(ДЛЯ БИОСА)`**:

#### Сначала создадим каталог `/mnt/boot`
```
mkdir -p /mnt/boot
```
#### Примонтируем загрузочный раздел

```
mount /dev/sda2 /mnt/boot
```

- ### Загрузочный **`(ДЛЯ UEFI)`**:

#### Сначала создадим каталог `/mnt/boot/EFI`

```
mkdir -p /mnt/boot/EFI
```

#### Примонтируем загрузочный раздел

```
mount /dev/sda2 /mnt/boot/EFI
```

> ## 4. Устанавливаем базовую систему утилитой `pacstrap`


```
pacstrap -i /mnt base base-devel linux-zen linux-zen-headers linux-firmware dosfstools btrfs-progs intel-ucode iucode-tool nano
```

#### Создаём файл конфигурации файловых систем:

```
genfstab -U /mnt >> /mnt/etc/fstab
```

#### Проверяем конфиг:

```
cat /mnt/etc/fstab
```

#### Заходим в установленную систему:

```
arch-chroot /mnt
```

#### Сконфигурируем время и дату:

```
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
```

```
hwclock --systohc
```

>## 5.Русифицируем систему.

#### Раскомментируем строчку с нужным языком.


```
nano /etc/locale.gen
```

 Например
```
ru_RU.UTF-8 UTF-8   ----   398 строчка
en_US.UTF-8 UTF-8   ----   171 строчка
```
#### Чтобы найти нужную строчку:

**`CTRL`** + **`-`** → `398`  
**`CTRL`** + **`-`** → `171`

#### Сохраним файл конфига:

**`CTRL`** + **`O`**  
**`ENTER`**  
**`CTRL`** + **`X`**

#### Затем генерируем локализацию языков:

```
locale-gen
```

#### Открываем конфиг `locale.conf`:

```
nano /etc/locale.conf
```

#### Прописываем туда нашу локализацию:

```
LANG=ru_RU.UTF-8
```

#### Сохраним файл конфига:

**`CTRL`** + **`O`**  
**`ENTER`**  
**`CTRL`** + **`X`**

### Настраиваем язык консоли:

```
nano /etc/vconsole.conf
```

#### Прописываем:
```
KEYMAP=ru
FONT=cyr-sun16
```
### Зададим имя пк (не путать с доменным):

```
nano /etc/hostname
```

#### Прописываем имя

### Зададим ПК доменное имя

```
nano /etc/hosts
```

#### Прописываем:
```
127.0.0.1       localhost  
::1             localhost  
127.0.0.1       ДОМЕННОЕ_ИМЯ     ИМЯ_ПК
```

>## 6. Установка системы

#### Создаём образ ядра для оперативной памяти:

```
mkinitcpio -P
```

#### Устанавливаем пароль root:

```
passwd
```

#### Скачиваем загрузчик и сетевые утилиты:

```
pacman -S grub efibootmgr dhcpcd dhclient networkmanager
```

#### Устанавливаем загрузчик grub

```
grub-install {path}
```

### `path - ПУТЬ ДО ДИСКА, НЕ ДО РАЗДЕЛА`

#### Например:

```
grub-install /dev/sda
```

#### Для UEFI

```
grub-install --boot-directory=/boot/EFI
```

#### Конфигурируем загрузчик:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

>## 7. Выходим

```
exit
```

#### Размонтируем все командой

```
umount -R /mnt
```

#### Перезагружается и заходим в установленную систему

```
reboot
```
---
### На этом этапе необходимо вытащить установочный носитель.

### Заходим в систему под логином `"root"`.
---

#### Откроем в редакторе файл sudoers

```
nano /etc/sudoers
```

#### Раскомментируем 85 строчку - **`CTRL`** + **`-`** → `85`

`%wheel ALL=(ALL:ALL) ALL` 


#### Создадим нового пользователя

```
useradd -m -G wheel -s /bin/bash USER_NAME
```

`USER_NAME - имя пользователя со строчными буквами`

#### Задаём пароль учётной записи
```
passwd USER_NAME
```
#### Выходим:

```
exit
```

#### Заходим под созданным пользователем

#### Проверим root доступ:

```
sudo su
``` 

#### Запускаем сетевую службу командой

```
systemctl enable NetworkManager
```

#### Перезагружаем систему командой

```
reboot
```

>## 8. Настройка системы

### Подключение к интернету по проводу также происходит автоматически

### Подключение к точке доступа немного другое :
```
nmcli d wifi connect NAME
```
`NAME - имя точки доступа`

```
nmcli d wifi connect NAME password PASSWORD
```

`PASSWORD - пароль точки доступа если есть`


### Откроем в редакторе конфиг pacman

```
sudo nano /etc/pacman.conf
```

#### Раскомментируем две строчки - 93 и 94

**`CTRL`** + **`-`** → `93`  
**`CTRL`** + **`-`** → `94`

>## 9. Устанавливаем пакеты видеоускорения

### Для AMD

```
sudo pacman -Syu lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader
```

### Для INTEL
```
lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-    loader libva-media-driver xf86-video-intel
```
### Для NVIDIA
```
nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader lib32-opencl-nvidia opencl-nvidia libxnvctrl 
```

### Для INTEL + NVIDIA
```
nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader lib32-opencl-nvidia opencl-nvidia libxnvctrl lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader libva-intel-driver xf86-video-intel 
```

### Установим дополнение NetworkManager для графических оболочек:

```
sudo pacman -S network-manager-applet
```

### Перезагружаемся

```
sudo reboot
```

>## 10. Устанавливаем графическую оболочку:

### Обновим зеркала (может помочь если будет ошибка при установке графической оболочки):

```
sudo reflector --sort rate -l 5 --save /etc/pacman.d/mirrorlist
```

## KDE Plasma:
```
sudo pacman -S xorg xorg-server plasma plasma-wayland-session egl-wayland sddm sddm-kcm packagekit-qt5 kde-applications
```
### Включим графическую оболочку:
```
sudo systemctl enable sddm
```
## Gnome:
```
sudo pacman -S xorg xorg-server gnome gnome-extra gdm
```
### Включим графическую оболочку:
```
sudo systemctl enable gdm
```
## XFCE:
```
sudo pacman -S xorg xorg-server xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
```
### Включим графическую оболочку:
```
sudo systemctl enable lightdm
```
## Сinnamon:
```
sudo pacman -S xorg xorg-server cinnamon
```
### Включим графическую оболочку:
```
sudo systemctl enable gdm
```
## Deepin:
```
sudo pacman -S xorg xorg-server deepin deepin-extra lightdm lightdm-deepin-greeter
```
### Включим графическую оболочку:
```
sudo systemctl enable lightdm
```
## Enlightenment:
```
sudo pacman -S xorg xorg-server enlightenment lightdm lightdm-gtk-greeter
```
### Включим графическую оболочку:
```
sudo systemctl enable lightdm
```
## Mate:
```
sudo pacman -S xorg xorg-server mate mate-extra mate-panel mate-session-manager
```
### Включим графическую оболочку:
```
sudo systemctl enable mdm
```
## LXDE:
```
sudo pacman -S xorg xorg-server lxde-common lxsession openbox lxde lxdm
```
### Включим графическую оболочку:
```
sudo systemctl enable lxdm
```

# Перезагружаемся, и на этот раз уже в графическую оболочку.

```
reboot
```
