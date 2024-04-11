 # Установка компилятора в ubuntu
 
 * <b>Скачиваем и устанавливаем </b>
	
    * Скачиваем архив:  gcc-arm-none-eabi-4_6-2012q4-20121016.tar.bz2

		```https://launchpad.net/gcc-arm-embedded/+milestone/4.6-2012-q4-update```
		
    * Или с нашего сервера: 

        ```smb://10.32.32.7/install/BU_compiler_ATMEL```

    * Распаковываем tarball в целевую директорию:

        ```cd target_dir && tar xjf gcc-arm-none-eabi-4_6-2012q4-20121016.tar.bz2```		

    * Затем добавляем путь до arm-none-eabi-gcc(/target_dir/gcc-arm-none-eabi-4_6-2012q4/bin) в PATH(в файл):

        ```sudo nano /etc/environment```

        ```sudo reboot```

    * Для проверки пишем:

        ```arm-none-eabi-gcc --version```
