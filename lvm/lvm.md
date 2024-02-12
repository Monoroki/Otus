#  Сборка LVM

### Уменьшить том под / до 8G


```bash

pvcreate /dev/sdb # Создаем том

vgcreate vg_root /dev/sdb # Создаем группу томов

lvcreate -n lv_root -l +100%FREE /dev/vg_root # Создаем раздел 

mkfs.xfs /dev/vg_root/lv_root # Создаем ФС 

mount /dev/vg_root/lv_root /mnt # Монтируем

yum install xfsdump

xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt # Копируем корень на новый раздел

for i in /proc/ /sys/ /dev/ /run/ /boot/; \
 do mount --bind $i /mnt/$i; done # прокидываем ссылки на новый раздел

chroot /mnt/ # переходим на новый корень

grub2-mkconfig -o /boot/grub2/grub.cfg # Настраиваем grub

cd /boot ; for i in `ls initramfs-*img`; \
do dracut -v $i `echo $i|sed "s/initramfs-//g; \
> s/.img//g"` --force; done # Обновим образ initrd

nano /boot/grub2/grub.cfg # Правим конфиг

lvremove /dev/VolGroup00/LogVol00 # Удаляем старый логический раздел

lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00 # Создаем новый

mkfs.xfs /dev/VolGroup00/LogVol00 # Создаем ФС

mount /dev/VolGroup00/LogVol00 /mnt # Монтируем

xfsdump -J - /dev/vg_root/lv_root | \ 
 xfsrestore -J - /mnt # Переносим обратно корень, далее в обратном порядке правим загрузчик и загружаемся уже с нового раздела.

```

### Выделить том под /var в зеркало

```bash
pvcreate /dev/sdc /dev/sdd

``` 
