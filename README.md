# Домашнее задание к занятию Troubleshooting - Морозов Александр

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

### Ответ
1. При установле приложения выдает ошибку в отсутствии Namespase: Web и Data
![alt text](https://github.com/Mars12121/kuber-homeworks_3.5/blob/main/img/1.png)

2. Скопируем себе манифест и добавим в него добавление Namspase [Deployment.yml](https://github.com/Mars12121/kuber-homeworks_3.5/blob/main/k8s/deployment.yml) 
```
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: web
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: data
---
```

3. Запускаем полученный Deployment и проверяем состояние. Как видим приложения запустились
![alt text](https://github.com/Mars12121/kuber-homeworks_3.5/blob/main/img/2.png)

4. Проверяем лог приложений. Приложение auth-db запустилось и работает. А вот приложение web-consumer запустилось, но не может подключиться по Curl к auth-db.
![alt text](https://github.com/Mars12121/kuber-homeworks_3.5/blob/main/img/3.png)

5. Проверим доступ из пода по доменному имени и по IP. Как видим из приложения web-consumer доступ по IP есть, а по доменному имени не резолвится. Соответственно проблема в DNS. Или сервис web-consumer ничего не знает про сервис auth-db.
![alt text](https://github.com/Mars12121/kuber-homeworks_3.5/blob/main/img/4.png)

6. В Deployment был указано обращение к сервис auth-db без указания Namespace. Соответственно при обращении по доменному имени auth-db, сервис web-consumer искал данный сервис в своем Namespace и не мог его найти. Исправляем в command Deployment обращение к auth-db с указанием Namespace
```
  spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db.data; sleep 5; done
```

7. Запускаем и проверяем лог. В логе приложения web-consumer видим, что сервис auth-db стал доступен по доменному имени. Измение непосредственно в Deploymen позволит в будующем при обновлении приложения или его перезапуске не допустить данной ошибки. Еси бы мы внесли изменения непосредственно в поде, то после перезапуска ошибка повторилась.
![alt text](https://github.com/Mars12121/kuber-homeworks_3.5/blob/main/img/5.png)

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.