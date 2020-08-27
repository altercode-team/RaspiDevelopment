# Настройка RO-RW в Raspbian Buster

## Скрипт

Для быстрой настройки можно воспользоваться скриптом "setup.sh" по ссылке https://github.com/Turonk/raspberry_read_only . Скрипт необходимо запускать под root-правами. 

## Подробнее
* <b>Необходимо обновить список пакетов:</b>

		sudo apt update

* <b>Устанавливаем BusyBox и NTP</b>
		
		apt install -y busybox-syslogd ntp
		
* <b>Удяляем лишнии утилиты (Ключ -y означает на все запросы консоли отвечать "yes", Ключ -purge - удаление всех данных связанных с приложением):</b>

		apt remove -y --purge anacron logrotate dphys-swapfile rsyslog
		
* <b>Изменяем параметры загрузки:</b>

	* <i>Сначала делаем backup cmdline.txt</i>
			
			cp /boot/cmdline.txt /boot/cmdline.txt.backup
		
	* <i>Получаем в переменную uuid универсальный уникальный индентификатор раздела, в котором находится etc/fstab (| - перенаправление вывода одной команды на вход другой)</i>
			
			uuid=`grep '/ ' /etc/fstab | awk -F'[=]' '{print $2}' | awk '{print $1}'`
			
* <b>Перемещаем resolv.conf в tmpfs(Во временную файловую систему)</b>

	* <i>Сначала перемещаем /etc/resolv.conf в /tmp/dhcpcd.resolv.conf</i>
	
			mv /etc/resolv.conf /tmp/dhcpcd.resolv.conf
			
	* <i>Затем создаем ярлык в /etc/resolv.conf на файл /tmp/dhcpcd.resolv.conf</i>
	
			ln -s /tmp/dhcpcd.resolv.conf /etc/resolv.conf

* <b>Перемещаем pid- и дргуих файлов в tmpfs(Во временную файловую систему)</b>
	
	* <i>Сначала удаляем /var/lib/systemd/random-seed, затем, если удаление прошло успешно создаем символическую ссылку /var/lib/systemd/random-seed на файл /tmp/random-seed (&& - Используется для выполнения операции, если предыдущая завершилась успешно. \ - Когда обратный слэш указывается последним во вводимой строке, оболочка интерпретирует его как знак продолжения строки. В этом случае она удаляет следующий далее знак новой строки, не интерпретируя его как разделитель аргументов, как будто его вообще не было:)</i>
			
			rm /var/lib/systemd/random-seed && \
			ln -s /tmp/random-seed /var/lib/systemd/random-seed
			
* <b>Редактируем файлы</b>			

	* <i>Делаем backup файла /lib/systemd/system/systemd-random-seed.service</i>

			cp /lib/systemd/system/systemd-random-seed.service /lib/systemd/system/systemd-random-seed.service.backup
			
	* <i>Заменяем содержимое файла /lib/systemd/system/systemd-random-seed.service (cat << EOF ... EOF - позволяет вводить в файл множество строк)</i>
			
			cat > /lib/systemd/system/systemd-random-seed.service << EOF			
			[Unit]
			Description=Load/Save Random Seed
			Documentation=man:systemd-random-seed.service(8) man:random(4)
			DefaultDependencies=no
			RequiresMountsFor=/var/lib/systemd/random-seed
			Conflicts=shutdown.target
			After=systemd-remount-fs.service
			Before=sysinit.target shutdown.target
			ConditionVirtualization=!container
			[Service]
			Type=oneshot
			RemainAfterExit=yes
			ExecStartPre=/bin/echo '' >/tmp/random-seed
			ExecStart=/lib/systemd/systemd-random-seed load
			ExecStop=/lib/systemd/systemd-random-seed save
			TimeoutSec=30s
			EOF
			
	* <i>Делаем backup файла /etc/cron.hourly/fake-hwclock</i>

			cp /etc/cron.hourly/fake-hwclock /etc/cron.hourly/fake-hwclock.backup
			
	* <i>Заменяем содержимое файла /etc/cron.hourly/fake-hwclock (cat << EOF ... EOF - позволяет вводить в файл множество строк)</i>		
			
			cat > /etc/cron.hourly/fake-hwclock << EOF
			# Простой скрипт - периодически сохраняет текущие часы, для предотвращения 
			# сброса времени при сбое питания.			 
			if (command -v fake-hwclock >/dev/null 2>&1) ; then
			  ro=$(mount | sed -n -e "s/^\/dev\/.* on \/ .*(\(r[w|o]\).*/\1/p")
			  if [ "$ro" = "ro" ]; then
				mount -o remount,rw /
			  fi
			  fake-hwclock save
			  if [ "$ro" = "ro" ]; then
				mount -o remount,ro /
			  fi
			fi
			EOF
			
	* <i>Меняем содержимое файла /etc/ntp.conf</i>

			sed -i.bak '/driftfile/c\driftfile /tmp\/ntp.drift' /etc/ntp.conf
			
* <b>Настройка tmpfs(временная файловая система) для рабочего стола</b>

	* <i>Создание(-fs означает, что если ссылка уже существует, то "принудительно" обновит ссылку) символической ссылки /home/pi/.Xauthority на файл /tmp/.Xauthority</i>
	
			ln -fs /tmp/.Xauthority /home/pi/.Xauthority
		
	* <i>Создание(-fs означает, что если ссылка уже существует, то "принудительно" обновит ссылку) символической ссылки /home/pi/.xsession-errors на файл /tmp/.xsession-errors</i>
	
			ln -fs /tmp/.xsession-errors /home/pi/.xsession-errors
		
* <b>Настройка файловой системы на read-only в fstab</b>
	
	* <i>Если в файле /etc/fstab /boot и ext4 не в ro, то устанавливаем. И конец файла /etc/fstab добавить папки (/tmp, /var/log, /var/tmp, /var/lib/dhcpcd5) в tmpfs</i>
	
			#Если количество "ro" в файле /etc/fstab равно нулю то:
			if [ 0 -eq $( grep -c ',ro' /etc/fstab ) ]; then
				sed -i.bak "/boot/ s/defaults/defaults,ro/g" /etc/fstab
				sed -i "/ext4/ s/defaults/defaults,ro/g" /etc/fstab

				echo "
				tmpfs           /tmp             tmpfs   nosuid,nodev         0       0
				tmpfs           /var/log         tmpfs   nosuid,nodev         0       0
				tmpfs           /var/tmp         tmpfs   nosuid,nodev         0       0
				tmpfs           /var/lib/dhcpcd5 tmpfs   nosuid,nodev         0       0
				" >> /etc/fstab
			fi
			
* <b>Модифицируем bashrc</b>
	
	* <i>Если в /etc/bash.bashrc нет строки "mount -o remount" то данные из ./bash.bashrc.addon записать в конец /etc/bash.bashrc</i>
	
			if [ 0 -eq $( grep -c 'mount -o remount' /etc/bash.bashrc ) ]; then
				cat ./bash.bashrc.addon >> /etc/bash.bashrc
			fi
			
* <b>Добавляем /etc/bash.bash_logout</b>
	
	* <i>Создаем bash.bash_logout, если он есть, то обновит файл до текущего времени</i>
	
			touch /etc/bash.bash_logout

	
	* <i>Если в /etc/bash.bash_logout нет строки "mount -o remount" то данные из ./bash.bash_logout.addon записать в конец /etc/bash.bash_logout</i>
	
			if [ 0 -eq $( grep -c 'mount -o remount' /etc/bash.bashrc ) ]; then
				cat ./bash.bashrc.addon >> /etc/bash.bashrc
			fi
			
* <b>Найстройка ядра на автоматическую перезагрузку при ?ошибке?(Configuring kernel to auto reboot on ?panic?.)</b>		

	* <i>Устанавливаем значение в kernel.panic и записываем в файл /etc/sysctl.d/01-panic.conf</i>
	
			echo "kernel.panic = 10" > /etc/sysctl.d/01-panic.conf

* <b>Отключение apt-daily и apt-daily-upgrade service</b>		
			
			#apt-daily.timer – ежедневный таймер службы для скачивания новых пакетов
			#apt-daily-upgrade.timer – ежедневный таймер службы для обновления и очистки пакетов
			#apt-daily.service – непосредственно запускает скачивание новых пакетов (команда #/usr/lib/apt/apt.systemd.daily update), вызывается таймером 
			#apt-daily-upgrade.service – непосредственно запускает установку новых пакетов и очистку кэша (команда #/usr/lib/apt/apt.systemd.daily install), вызывается таймером
			
			systemctl disable apt-daily.service
			systemctl disable apt-daily.timer
			systemctl disable apt-daily-upgrade.service
			systemctl disable apt-daily-upgrade.timer
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
	