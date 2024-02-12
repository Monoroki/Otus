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
vgcreate vg_var /dev/sdc /dev/sdd
lvcreate -L 950M -m1 -n lv_var vg_var # Создаем зеркало

mkfs.ext4 /dev/vg_var/lv_var # Создаем ФС и перемещаем туда /var
mount /dev/vg_var/lv_var /mnt
cp -aR /var/* /mnt/

umount /mnt
mount /dev/vg_var/lv_var /var # Монтируем новый var в каталог /var

echo "`blkid | grep var: | awk '{print $2}'` \
 /var ext4 defaults 0 0" >> /etc/fstab #Правим fstab для автоматического монтирования /var

``` 

### Выделить том под /home
 
По той же системе что делали с /var

### Сапшоты

```bash
touch /home/file{1..20} # Генерируем файлы

lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home # Снимаем снапшот

rm -f /home/file{11..20} # Удаляем часть файлов

umount /home
lvconvert --merge /dev/VolGroup00/home_snap
mount /home # Восстанавливаем удаленное из снапшота
```
###  Результат

![Скриншот01](https://github.com/Monoroki/Otus/blob/main/image/lvm.png)

![Скриншот01](https://github.com/Monoroki/Otus/blob/main/image/lvm2.png)
