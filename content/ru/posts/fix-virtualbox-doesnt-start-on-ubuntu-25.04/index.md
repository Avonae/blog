---
title: Virtualbox не запускается на Ubuntu 25.04
date: '2025-11-23'
translationKey: fixing-virtualbox-start-on-ubuntu-25.04
aliases:
- /2025-11-23-Fix-Virtualbox-doesnt-start-on-ubuntu-25.04/
---

По какой-то глупой случайности я обновил свою десктопную LTS убунту на последнюю версию - 25.04. И тут началось… Половина приложений просто перестала работать, так как десктопная среда переехала на Wayland с Gnome и в довершение - перестал работать Virtualbox. Я стал разбираться.

Оказалось, что начиная с [версии 6.12 ядра](https://www.virtualbox.org/ticket/22248#comment:1), KVM в убунте включен по умолчанию и мешает запуску виртуалбокса. Поэтому, если он вам не нужен - отключаем.


Отключить проще всего через параметр grub. Открываем его конфиг:

```bash
sudo nano /etc/default/grub
```

В строке `GRUB_CMDLINE_LINUX` добавьте `kvm.enable_virt_at_load=0`

![Параметр kvm.enable_virt_at_load=0 в GRUB_CMDLINE_LINUX](kvm-ubuntu-25.04.png)

Сохраните изменения и дальше:

```bash
sudo update-grub
```

И перезагрузитесь, например `sudo reboot`

Ну и всё. KVM не будет загружаться и мешать виртуалбоксу.

А если хотите использовать виртуалбокс через KVM бекенд, посмотрите [этот репозиторий](https://github.com/cyberus-technology/virtualbox-kvm). Честно, я не пробовал — но вдруг вам пригодится.