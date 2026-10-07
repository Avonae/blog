---
title: Фикс большой скорости тачпада на Linux Mint и Ubuntu
date: '2025-08-03'
translationKey: fix-high-touchpad-speed-on-linux-mint
aliases:
- /2025-08-03-Fix-high-touchpad-speed-on-linux-mint/
---

Месяца 3 назад я окончательно перебрался на Linux Mint и все в целом было окей, но была проблема с тачпадом. Он работал как-то слишком быстро.

В интернете уже [описаны решения](https://askubuntu.com/questions/1120045/touchpad-two-finger-scroll-too-fast#1132826), но оно мне до конца не помогло. Поэтому представляю вам свое.

## Настройка скорости тачпада

Первым делом ищем свой ID тачпада:

```bash
sudo xinput list
```

в списке будут все устройства, среди которых надо найти тачпад. У меня это 9.
![Вывод xinput list: тачпад ELAN06FA:00 с id=9](xinput-list-touchpad.png)
Затем смотрим, какое значение имеет скорость прокрутки. Чем меньше число - тем быстрее тачпад:

```bash
xinput list-props 'id of touchpad' | grep "Scrolling Distance"
```

Вы увидите что-то вроде:

```bash
Synaptics Scrolling Distance (351):  -88, 88
```

Результат состоит из 2 чисел -88 и 88. Их нужно поменять на что-то побольше. Мне нравится ставить в районе 350, но можно еще больше, вроде 400 или 500.

```bash
xinput set-prop 11 "Synaptics Scrolling Distance" -500 500
```

После этого проверьте, как работает тачпад. Должно стать лучше.
Поиграйтесь с числами и выберите желаемое значение. Его нужно записать в конфиге, иначе оно потеряется после перезагрузки.

## Правки конфига

Конфиг тачпада хранится в папке ```/usr/share/X11/xorg.conf.d```, найдите его:

```bash
ls /usr/share/X11/xorg.conf.d/
10-amdgpu.conf  10-quirks.conf  10-radeon.conf  40-libinput.conf  51-synaptics-quirks.conf  70-synaptics.conf  70-wacom.conf
```

Берем этот файл ```70-synaptics.conf``` и копируем в /etc:

```bash
sudo cp 70-synaptics.conf /etc/X11/xorg.conf.d/
```

открываем его и правим:

```bash
sudo vim /etc/X11/xorg.conf.d/70-synaptics.conf
```

Добавляем в него ваши 2 цифры скорости:

```bash
    Option "VertScrollDelta"  "-350"
    Option "HorizScrollDelta" "350"
```

все остельные настройки трогать не нужно. В результате получится такой конфиг:

```bash
Section "InputClass"
    Identifier "touchpad custom scroll"
    MatchIsTouchpad "on"
    Driver "synaptics"
    Option "VertScrollDelta"  "-350"
    Option "HorizScrollDelta" "350"
EndSection
```

Сохраните изменения и перезагрузитесь.
После перезагрузки ID устройства изменися, поэтому сначала узнаем его, а потом делаем проверку:

```bash
sudo xinput list
```

и затем с новым ID:

```bash
xinput list-props 11 | grep "Scrolling Distance"
```

должно выдавать нужные значения, у меня 350.
![Вывод xinput list-props 9: Synaptics Scrolling Distance -350, 350](xinput-scrolling-distance.png)

Вот и все - ID устройства будет меняться, но скролл будет работать как надо.
