# Hap-Keep-Docker
Отказоустойчивый кластер с балансировкой нагрузки Docker, HAProxy, Keepalived, nginx, Load Balance

Кластер на основе Docker.<br>
На базе образов morningspace/lab-web и morningspace/lab-lb. <br>

Два веб-сервера web1/web2 - контейнеры (morningspace/lab-web), на основе Nginx в качестве веб-сервера. Http/https включены.<br>
Перед серверами две балансировщика mylb1(главный узел) и mylb2(резервный узел) - контейнеры (morningspace/lab-lb)<br>
И haproxy, и keepalived установлены. haproxy подключается к двум веб-серверам. <br>
keepalived проверяет есть ли доступ к haproxy. <br>

Создание образов и запуск веб-сервера:<br>

docker build -f docker/web/Dockerfile -t morningspace/lab-web .<br>
docker build -f docker/lb/Dockerfile -t morningspace/lab-lb .<br>

docker run -d --name myweb1 --hostname myweb1 --net=lab -p 18080:80 -p 18443:443 morningspace/lab-web<br>
docker run -d --name myweb2 --hostname myweb2 --net=lab -p 19080:80 -p 19443:443 morningspace/lab-web<br>

Запуск балансировщиков:<br>
docker run -it --name mylb1 --hostname mylb1 --net=lab -p 28080:8080 -p 28443:8443 -p 28090:8090 --sysctl net.ipv4.ip_nonlocal_bind=1 --privileged morningspace/lab-lb<br>
docker run -it --name mylb2 --hostname mylb2 --net=lab -p 29080:8080 -p 29443:8443 -p 29090:8090 --sysctl net.ipv4.ip_nonlocal_bind=1 --privileged morningspace/lab-lb<br>

Важно скопировать файл конфигурации в /etc/haproxy:<br>
haproxy-ssl-termination.conf haproxy.cfg<br>

Виртуальный IP адрес: 172.18.0.6 (в keepalived-master.confи keepalived-backup.conf)<br>

Запуск keepalived на мастере и резерве:<br>
keepalived --dump-conf --log-console --log-detail --log-facility 7 --vrrp -f /etc/keepalived/keepalived-master.conf<br><br>
keepalived --dump-conf --log-console --log-detail --log-facility 7 --vrrp -f /etc/keepalived/keepalived-backup.conf<br>
