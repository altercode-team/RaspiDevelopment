# Настройка экрана и правка стилей

## Настройка экрана

* <b>Поворот экрана для Raspberry Pi 4 (Так не сохраняется)</b>

	Из-за нового видеодрайвера, используемого Raspberry Pi 4, вы не можете повернуть экран, используя старый метод /boot/config.txt.

	Если вы хотите повернуть экран на Pi 4, вам следует использовать инструмент настройки экрана, так как его гораздо проще использовать.

	Вместо этого вам нужно будет использовать команду xrandr.

	В Терминале Raspberry Pi выполните одну из следующих команд.

	Если вы хотите, чтобы поворот повлиял на второй слот HDMI, попробуйте использовать HDMI-2 вместо HDMI-1 в приведенных ниже командах.
		
			DISPLAY =: 0 xrandr –output HDMI-1 –rotate normal
			
	Эта команда восстанавливает нормальное вращение.
		
			DISPLAY =: 0 xrandr –output HDMI-1 – повернуть влево
			
	Приведенная выше команда поворачивает выход монитора влево. Это эквивалент поворота на 90 градусов.
		
			DISPLAY =: 0 xrandr –output HDMI-1 – повернуть вправо
			
	Эта команда поворачивает экран вправо. Эта команда похожа на поворот экрана на 270 градусов.
		
			DISPLAY =: 0 xrandr –output HDMI-1 –rotate инвертированный
			
	Эта последняя команда перевернет экран. Это похоже на поворот экрана на 180 градусов.	
<br>
<br>

* <b>Рабочий вариант повора экрана</b> 
	
			sudo arandr

* <b> Для поворота тачпада</b>			
	
	В файл /usr/share/X11/xorg.conf.d/40-libinput.conf
	
			Section "InputClass"
				Identifier "libinput touchpad catchall"
				MatchIsTouchpad "on"
				Option "CalibrationMatrix" "1 0 0 0 1 0 0 0 1"
				MatchDevicePath "/dev/input/event*"
				Driver "libinput"
			EndSection

			Section "InputClass"
					Identifier "libinput touchscreen catchall"
					MatchIsTouchscreen "on"
					MatchDevicePath "/dev/input/event*"
					Driver "libinput"
					Option "TransformationMatrix" "1 0 0 0 1 0 0 0 1"
					Option "SwapAxes" "true"
					Option "InvertX" "true"
					Option "InvertY" "true"
			EndSection
	
	Менять необходимо обе строки: (Нормальная ориентация)
	
			Option "CalibrationMatrix" "1 0 0 0 1 0 0 0 1"
			Option "TransformationMatrix" "1 0 0 0 1 0 0 0 1"

* <b>В файл boot/config.txt добавить:</b>
			
			max_usb_current=1
			hdmi_group=2
			hdmi_mode=87
			hdmi_cvt 1024 600 60 6 0 0 0
			
* <b>В командную строку ввести:
		
			sudo apt-get install xserver-xorg-input-evdev
			sudo mv /usr/share/X11/xorg.conf.d/40-libinput.conf /usr/share/X11/xorg.conf.d/40-libinput.conf_bak
			
			
## Правка стилей 

* <b>Добавить в файл Widget.qss стиль: </b>
		
			QStatusBar::item { border: 0px solid black }; 