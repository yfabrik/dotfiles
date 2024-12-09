# creer iscsi disk
## ( sur serveur )
`apt install targetcli` le service c'est target
`sudo targetcli`
### créer disk 
crée un fichier /iscsi_disk.img de 10G:
`/backstores/fileio create iscsi_disk /iscsi_disk.img 10G`

crée un iscsi target:
`/iscsi create iqn.2024-11.com.example:target1`

lier le fichier/disk et le iscsi target
`/iscsi/iqn.2024-11.com.example:target1/tpg1/luns create /backstores/fileio/iscsi_disk`

set portal pour que le disk puisse etre vu/monté
`/iscsi/iqn.2024-11.com.example:target1/tpg1/portals create`

add les pc client dans acls quand ils essayent de se connecter
`journalctl|grep iqn` pour les trouver
`/iscsi/iqn.2024-11.com.example:target1/tpg1/acls create iqn.client:initiator1`
# install sur iscsi
boot live install
`apt install open-iscsi`
trouver le disk:
`sudo iscsiadm -m discovery -t sendtargets -p 192.168.1.55`
connect disk:
`sudo iscsiadm -m node --targetname iqn.2010-04.org.ipxe.dolphin:storage -p 192.168.1.55 --login


# post install
le grub est sans doute foiré

## (sur serveur)

connecter le disk iscsi

mount it et chroot inside
```
sudo mount /dev/sdb2 /mnt
sudo mount /dev/sdb1 /mnt/boot/efi
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run
sudo chroot /mnt
```

edit /etc/fstab parce que la partition efi à pas la bonne ID
```
blkid ## pour obtenir l'id 
```

setup iscsi
```
apt install open-iscsi

echo ISCSI_AUTO=true >/etc/iscsi/iscsi.initramfs
```
refaire le grub parce que le grub il est vide
```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --boot-directory=/boot --recheck 
update-grub

update-initramfs -u
```

unmount
```
sudo umount /mnt/run
sudo umount /mnt/sys
sudo umount /mnt/proc
sudo umount /mnt/dev
sudo umount /mnt/boot/efi
sudo umount /mnt

```

### testé mais pas utile apparement

```
systemctl enable iscsid

UUID=12345678-1234-1234-1234-123456789012 /     ext4   defaults,_netdev,x-systemd.requires=iscsid.service   0      1
```

# ipxe setting
dans le menu ipxe
```
:sanbootiscsi
set initiator-iqn iqn.2024-12.fabrik.ipxe:ipxe

sanboot --filename \EFI\ubuntu\grubx64.efi iscsi:192.168.1.55:::0:iqn.2024-11.lan.fabrik.iscsi:windows
```

