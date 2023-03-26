# DeployAspNetCoreGuide

клоним репу \
apt install git \
cd /home \
git clone https://github.com/Excalib88/DeployGuide.git \

ставим .net core sdk \
wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0 \

sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-6.0 \

собираем/запускаем приложение \
cd DeployGuide/DeployGuide \
dotnet run \

установка nginx \
sudo apt update \
sudo apt install nginx \

открытие портов \ 
sudo ufw status \
sudo ufw allow 'Nginx Full' \
sudo ufw allow 22 \
sudo ufw allow 80 \
sudo ufw allow 443 \
sudo ufw enable \
sudo ufw status \

настройка демона \
sudo nano /etc/systemd/system/kestrel-deploy-guide.service \

правим пути и энвы по необходимости и вставляем в созданный файл \
[Unit]
Description=Example .NET Web API App running on Linux

[Service]
WorkingDirectory=/home/DeployGuide/DeployGuide
ExecStart=/usr/bin/dotnet /home/DeployGuide/DeployGuide/bin/Debug/net6.0/DeployGuide.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
#envs
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target

конец файла
продолжаем настройку демона
sudo systemctl enable kestrel-deploy-guide.service \
sudo systemctl start kestrel-deploy-guide.service \
sudo systemctl status kestrel-deploy-guide.service \

настройки nginx \
sudo nano /etc/nginx/sites-available/excalib.ru \

конфиг нжинкса проверь порт приложения в демоне \

server {
    listen        80;
    server_name   excalib.ru;
    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
#конец файла

sudo ln -s /etc/nginx/sites-available/excalib.ru /etc/nginx/sites-enabled/
sudo nano /etc/nginx/nginx.conf

#раскоментировать в http блоке
#server_names_hash_bucket_size 64;

#если после выполенения этой команды ошибок не получили то всё отлично проверяем (вводим ip в браузере и любуемся нашим сайтом)
sudo systemctl restart nginx

#настроил днсы(не забыл создать днс хост на стороне провайдера вдс)

#настраиваем https с сертом letsencrypt certbot

sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
#если нжинкс слушает 80ый порт то надо вырубить нжинкс
sudo systemctl stop nginx 

sudo certbot --nginx -d excalib.ru
sudo certbot renew --dry-run
