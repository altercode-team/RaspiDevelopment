# Настройка WriteFolder и USB

## WriteFolder

* <b>Флешку с образом системы необходимо подсоединить к виртуальной машине с Linux</b>

* <b>Устанавливаем и запускаем программу GParted</b>
	
		sudo apt-get install gparted
		sudo gparted
		
* <b>С помощью програмыы GParted создаем новый раздел</b>

	* Демонтируем rootfs раздел. Для этого нажимаем на него правой кнопкой мыши, затем выбираем  "Unmount" и оставляем для него 10Гб
	
	* Выбираем в меню "Partition -> New" и создаем новый раздел
	
* <b>Переставляем флешку в raspberry 4 и добавляем в ytc/fstab строку:</b>

		PARTUUID=76661929-03  /home/pi/writefolder       ext4    defaults,noatime     0       2
		
* <b>Для переноса ПО Запускаем старый аппарат и с него запускаем скрипт:</b>

	* sourceDir - директория, которую необходимо перенести
	* deployDir - директория, в которую необходимо перенести
	* ip - указываем адрес нового аппарата
	* rsync - команда, которая выполняет синхронизация папки со старого аппарата в новый (копирует) 

			sourceDir=/home/pi/writefolder/
			ip=10.32.32.132 
			deployDir=/home/pi/writefolder/
			rsync -rt --recursive -O $sourceDir pi@$ip:$deployDir
	
		
## USB

* <b>Для монтирования флешки в /home/pi/writefolder/usb в файл ytc/fstab нужно записать строку:</b>

		/dev/sda1 /home/pi/writefolder/usb/mntusb	 auto 	 rw,nofail,uid=pi,gid=pi,noatime 0       0
		
