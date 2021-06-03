# Настройка BlueZ(Bluetooth)
 
## Скачать исходный код
 
* Получите ссылку на последнюю версию BlueZ с http://www.bluez.org/download/ и загрузите ее:

			wget http://www.kernel.org/pub/linux/bluetooth/bluez-5.43.tar.xz
			
* Затем распакуйте его с помощью

			tar xvf bluez-5.43.tar.xz			
			
* Сменить каталог:

			cd bluez-5.43 /
		
* Затем скопируйте необходимые пакеты:

			sudo apt-get update 
			sudo apt-get install -y libusb-dev libdbus-1-dev libglib2.0-dev libudev-dev libical-dev libreadline-dev
			
* Затем соберите BlueZ со стандартной конфигурацией:

			./configure
			
	Это должно создать конфигурацию без ошибок. Если есть ошибки, вероятно, что-то пошло не так с указанными выше зависимостями и установкой библиотеки.
<br>
<br>

* Начнем компиляцию стандартной make:

			make

* Это займет около 20 минут. Затем установите его с помощью

			sudo make install
			
## Запуск / остановка / включение / отключение службы

* Поскольку системная служба на диске изменилась с нашей сборкой, перезагрузите службу с помощью

			sudo systemctl daemon-reload
			
* Используйте следующее, чтобы запустить или остановить службу:

			sudo systemctl start bluetooth 
			sudo systemctl stop bluetooth
			
* Для получения статуса услуги используется следующее:

			systemctl status bluetooth
			
* Для автоматического включения службы при запуске системы используйте

			sudo systemctl enable bluetooth

* Это можно снова отключить с помощью

			sudo systemctl disable bluetooth