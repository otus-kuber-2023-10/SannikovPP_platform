создание кластера
sudo ./kind create cluster --config kind-config.yaml --name kind-6 --wait 240s

добавляем конфиг для kubectl
./kind get kubeconfig --name kind-6 > ~/.kube/config


Добавьте на DockerHub версию образа с новым тегом (v0.0.2, можно
просто перетегировать старый образ)

docker login
docker tag front_demo:latest sannikovpp/test:v0.0.2
docker push sannikovpp/test:v0.0.2

Обновите в манифесте версию образа

        image: docker.io/sannikovpp/test:v0.0.2

Примените новый манифест, параллельно запустите отслеживание
происходящего

проверим образ, указанный в ReplicaSet

kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
docker.io/sannikovpp/test:v0.0.2

образ из которого сейчас запущены pod, управляемые контроллером

kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
docker.io/sannikovpp/test:latest docker.io/sannikovpp/test:latest docker.io/sannikovpp/test:latest

удаляем поды
kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w

Поды создались заново
Проверяем

kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
docker.io/sannikovpp/test:v0.0.2

kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
docker.io/sannikovpp/test:v0.0.2 docker.io/sannikovpp/test:v0.0.2 docker.io/sannikovpp/test:v0.0.2

????????????????????????
почему обновление ReplicaSet не повлекло обновление запущенных pod

????????????????????????

собираем paymentservice

sudo docker build  --network=host -t paymentservice src/paymentservice/
docker tag paymentservice:latest sannikovpp/paymentservice:v0.0.1
docker push sannikovpp/paymentservice:v0.0.1

docker tag paymentservice:latest sannikovpp/paymentservice:v0.0.2
docker push sannikovpp/paymentservice:v0.0.2

kubectl apply -f paymentservice-replicaset.yaml| kubectl get pods -l app=paymentservice -w

paymentservice-9jhw9   1/1     Running             0          2m42s
paymentservice-5mgbq   1/1     Running             0          2m47s
paymentservice-tnrj4   1/1     Running             0          2m49s


kubectl get replicaset paymentservice -o=jsonpath='{.spec.template.spec.containers[0].image}'
docker.io/sannikovpp/paymentservice:v0.0.1

kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'
docker.io/sannikovpp/paymentservice:v0.0.1 docker.io/sannikovpp/paymentservice:v0.0.1 docker.io/sannikovpp/paymentservice:v0.0.1

kubectl apply -f paymentservice-deployment.yaml| kubectl get pods -l app=paymentservice -w

kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
paymentservice   3/3     3            3           26s

kubectl get rs
NAME             DESIRED   CURRENT   READY   AGE
frontend         3         3         3       12h
paymentservice   3         3         3       12m



Обновление Deployment

обновил наш Deployment на версию образа v0.0.2

Убедитесь что:
• Все новые pod развернуты из образа v0.0.2

kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'
docker.io/sannikovpp/paymentservice:v0.0.2 docker.io/sannikovpp/paymentservice:v0.0.2 docker.io/sannikovpp/paymentservice:v0.0.2

Создано два ReplicaSet:

kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
frontend                    3         3         3       12h
paymentservice-7d668c79c7   0         0         0       15m
paymentservice-855ffd9c9b   3         3         3       7m47s


kubectl rollout history deployment paymentservice

deployment.apps/paymentservice 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>


Deployment | Rollback

обновление по каким-то причинам произошло неудачно
и нам необходимо сделать откат.

kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l
app=paymentservice -w

2 сценария развертывания

Аналог blue-green

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 100%
      maxUnavailable: 0

Reverse Rolling Update

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1


PROBES


взял daemonset из https://github.com/bibinwilson/kubernetes-node-exporter/blob/main/daemonset.yaml

создал namespace monitoring

kubectl create namespace monitoring

Применил daemonset

kubectl apply -f node-exporter-daemonset.yaml

Поды поднялись только на worker нодах

Пробросил порт 

kubectl port-forward  node-exporter-4762l 9100:9100 -n monitoring


получил метрики 

curl localhost:9100/metrics

Пример 

# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.02

для запуска на всех нодах добавил спецификацию в демосет

      tolerations:
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule








