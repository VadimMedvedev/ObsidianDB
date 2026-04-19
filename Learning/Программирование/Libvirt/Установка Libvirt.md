
libvirt, (вспомогательные утилиты:) virt-install, ebtables, dnsmasq, dmidecode, virt-viewer.

Ставим libvirt в автозагрузку:
``` bash
systemctl start libvirtd
systemctl enable libvirtd
```

Для того чтобы пользоваться сервисом libvirt юзер linux должен быть добавлен в группу этого приложения:
``` bash
usermod -aG libvirt vadim
```
