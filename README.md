# SannikovPP_platform
SannikovPP Platform repository

Запустил minikub на windows с драйвером virtualbox
Создал долкер образ 
minikube ssh
создал Dockerfile согласно рекомендации
создал nginx.conf согласно рекомендации
собрал имедж
docker build -t nginx_local -f ./doker_nginx .
Dockerfile и nginx.conf положил в директорию web

создал манифест web-pod.yaml
запустил контейнер
посмотрел манифест запущенного пода и его описание

добавил в манифест web-pod.yaml init контейнер
перезапустил под с новым манифестом
наблюдал с помощью рекомендованных команд старт пода
Сначала стартовал init потом сам под

Сделал проксирование порта 2-мя способами через kubectl и приложение kubernetes-port-forwarder
открылась ссылка http://localhost:8000/index.html
положил index.html в директорию web

Собрал докер образ и положил его на dokerhub docker.io/sannikovpp/test:latest

запустил контейнер командой kubectl 
Под в состоянии error так как для его работы не хватает переменных окружения env о чем говорит лог пода.

panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set 

создал манифест frontend-pod-healthy.yaml
Запустился pod frontend-8456646998-rmlwv
