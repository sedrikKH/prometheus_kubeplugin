# prometheus_kubeplugin

## Завдання:

Доопрацюйте та виправте помилки скрипта kubeplugin
Зробіть рефаторинг коду скрипта
Протестуйте скрипт як плагін kubectl
Закомітьте скрипт у директорію scripts репозиторію
Текстовий вивід статистики з реального кластеру namespace kube-system у форматі: "Resource, Namespace, Name, CPU, Memory"
Додайте інструкцію з використання


Відповідь:

Доопрацювання та виправлення помилок скрипта kubeplugin

Ось список помилок, які були знайдені в скрипті kubeplugin:

Помилка синтаксису: змінна RESOURCE_TYPE не має ані значення за замовчуванням, ані способу його задання.
Помилка логіки: вивід скрипта не містить імені ресурсу, на якому було зібрано статистику використання ресурсів.

```
#!/bin/bash

# Define command-line arguments

RESOURCE_TYPE=$1
NAMESPACE=$2

# Retrieve resource usage statistics from Kubernetes
kubectl get $RESOURCE_TYPE -n $NAMESPACE | tail -n +2 | while read line
do
  # Extract resource name, CPU and memory usage from the output
  NAME=$(echo $line | awk '{print $1}')
  CPU=$(echo $line | awk '{print $2}')
  MEMORY=$(echo $line | awk '{print $3}')

  # Output the statistics to the console
  # "Resource, Namespace, Name, CPU, Memory"
  echo "$RESOURCE_TYPE, $NAMESPACE, $NAME, $CPU, $MEMORY"
done

```

Додамо перевірку введених аргументів скрипта:

```
# Define minimum required arguments
MIN_ARGS=2

# Check if enough arguments were provided
if [ $# -lt $MIN_ARGS ]; then
  echo "Error: Please provide both RESOURCE_TYPE and NAMESPACE arguments!"
  exit 1
fi
```

Протестуємо роботу скрипта

```
$ bash ./kubeplugin.sh pod argocd

pod, argocd, argocd-applicationset-controller-6f9d6cfd58-r5w7g, 1/1, Running
pod, argocd, argocd-redis-b5d6bf5f5-2d4r4, 1/1, Running
pod, argocd, argocd-notifications-controller-589b479947-tpfrb, 1/1, Running
pod, argocd, argocd-dex-server-6df5d4f8c4-zvddb, 1/1, Running
pod, argocd, argocd-application-controller-0, 1/1, Running
pod, argocd, argocd-repo-server-7b75b45897-z64ll, 1/1, Running
pod, argocd, argocd-server-7459448d56-hd5r7, 1/1, Running
```

Скопіюємо плагін 

```
sudo cp ./kubeplugin.sh /usr/local/bin/kubectl-kubeplugin
```

Перевіремо список плгінів 

```
$ kubectl plugin list
The following compatible plugins are available:

/home/sedrik/.krew/bin/kubectl-krew
/home/sedrik/.krew/bin/kubectl-ns
/usr/local/bin/kubectl-kubeplugin
```

Протестуємо плагін kubeplugin на локальному кластері k3d - argocd

Інформація про кластер

```
$ k3d cluster list
NAME   SERVERS   AGENTS   LOADBALANCER
argo   1/1       0/0      true


$ kubectl get po -n argocd
NAME                                                READY   STATUS    RESTARTS      AGE
argocd-applicationset-controller-6f9d6cfd58-r5w7g   1/1     Running   1 (52m ago)   4d17h
argocd-redis-b5d6bf5f5-2d4r4                        1/1     Running   1 (52m ago)   4d17h
argocd-notifications-controller-589b479947-tpfrb    1/1     Running   1 (52m ago)   4d17h
argocd-dex-server-6df5d4f8c4-zvddb                  1/1     Running   1 (52m ago)   4d17h
argocd-application-controller-0                     1/1     Running   1 (52m ago)   4d17h
argocd-repo-server-7b75b45897-z64ll                 1/1     Running   1 (52m ago)   4d17h
argocd-server-7459448d56-hd5r7                      1/1     Running   1 (52m ago)   4d17h
```

Тепер тестуємо плагін

```
$ kubectl kubeplugin pod argocd
pod, argocd, argocd-applicationset-controller-6f9d6cfd58-r5w7g, 1/1, Running
pod, argocd, argocd-redis-b5d6bf5f5-2d4r4, 1/1, Running
pod, argocd, argocd-notifications-controller-589b479947-tpfrb, 1/1, Running
pod, argocd, argocd-dex-server-6df5d4f8c4-zvddb, 1/1, Running
pod, argocd, argocd-application-controller-0, 1/1, Running
pod, argocd, argocd-repo-server-7b75b45897-z64ll, 1/1, Running
pod, argocd, argocd-server-7459448d56-hd5r7, 1/1, Running
```


