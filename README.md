# Сервис на systemctl

1. Создаём сервис в папке с системными сервисами с помощью команды:
    `sudo nano /etc/systemd/system/my-service.service`
    У пользовательского сервиса должно быть расширение .service. Команда `sudo` даёт пользователю права супер-пользователя, `nano` – открывает текстовый редактор nano на Ubuntu.
2. В открывшемся текстовом редакторе пишем: 

    [Unit]
   - Description= your service description # Описание вашего сервиса
   - After=network.target # Для запуска после установки соединения
    [Service]
   - WorkingDirectory=/path/to/your/project # Путь к папке вашего проекта
   - ExecStart=/usr/bin/dotnet run # Команда, запускающая проект
   - User=*Your_username* # Username пользователя, который запускает сервис
   - Restart=always # Перезапуск сервиса при сбое
   - RestartSec=5 # Задержка при перезапуске в секундах
    [Install]
   - WantedBy=multi-user.target # Запуск в режиме многопользовательской системы
  
3.  Перезагружаем systemd-сервисы командой:
	`sudo systemctl daemon-reload`
4.  Включаем сервис командой:
	`sudo systemctl enable my-service.service`
5.  Для проверки работы сервиса используем команды:
	   - для проверки статуса: `sudo systemctl status my-service.service`
	   - для просмотра логов: `sudo journalctl -u my-service.service` 
  
   
   
   