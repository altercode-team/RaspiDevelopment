# Настройка экрана и правка стилей

## Настройка экрана

* В файл boot/config.txt добавить:
			
			max_usb_current=1
			hdmi_group=2
			hdmi_mode=87
			hdmi_cvt 1024 600 60 6 0 0 0
			
* В командную строку ввести:
		
			sudo apt-get install xserver-xorg-input-evdev
			sudo mv /usr/share/X11/xorg.conf.d/40-libinput.conf /usr/share/X11/xorg.conf.d/40-libinput.conf_bak
			
			
## Правка стилей 

* Добавить в файл Widget.qss стиль: 
		
			QStatusBar::item { border: 0px solid black }; 