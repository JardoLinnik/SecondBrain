Подключение к терминалу пода  
kubectl exec -it <имя-пода> -- /bin/bash  
  
Просмотр логов пода  
kubectl logs <имя-пода>  
  
Изменение количества подов  
kubectl scale deployment <имя-развертывания> --replicas=5  
  
Деплой нового сервиса на основе готового конфига  
kubectl apply -f ./k8s-configs/  
  
Список подов  
kubectl get pods  
  
Информация о поде  
kubectl describe pod <имя-пода>  
  
Удалить под  
kubectl delete pod <имя-пода>  
  
Создать(удалить) пространство имён  
kubectl create(delete) namespace <имя-неймспейса>  
  
Снести проект  
kubectl delete all -l app=<имя-приложения>