# Laboratorio Kubernetes

Este laboratorio está basado en los ejemplos previos, en particular la tarea 4.


## Requisitos

Para ejecutar este laboratorio debes tener instalado `mini-kube`.

Encuentra las instrucciones para instalarlo acá: https://minikube.sigs.k8s.io/docs/start/

Además debes tener una cuenta en dockerhub: https://hub.docker.com

## Imagenes

Este laboratorio asume que has creado 3 imágenes en docker-hub:

- labcna-chat-frontend: con la imagen del frontend
- labcna-users-svc: con la imagen de la api para gestionar usuarios
- labcna-ws-server: con la imagen del servidor web socket

Si no las has creado en la tarea 4, puede usar las imágenes que el profesor disponga en clases.

## Paso 1

Vamos a iniciar minikube

```
minikube start
```

Abre otro terminal y ejecuta el dashboard:

```
minikube dashboard
```

Este comando abrirá tu navegador y te mostrará el dashboard de k8s.


## Paso 2

Revisa el contenido de la carpeta `base`.

Encontrarás dos archivos, el primero es `chat-app-namespace.yaml` que contiene lo siguiente:

```
apiVersion: v1
kind: Namespace
metadata:
  name: chat-app
  labels:
    name: chat-app
```

Esto define el namespace `chat-app` que es donde residirán todos los objetos que crearemos.

El otro archivo es `chat-app-storage-class.yaml`, que define un storage class que será usado para crear un volumen persistente para nuestra base de datos posteriormente.

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: k8s.io/minikube-hostpath
parameters:
  type: pd-ssd
```

Luego para crear estos objetos debes ejecutar:

```
kubectl apply -f base
```

Entra al dashboard y selecciona el namespace `chat-app` en el combobox. Haz click en `Storage Classes` (debajo de `Config and Storage`) y verifica que aparece `fast` y `standard`.

También puedes obtener esta información en la consola del siguiente modo:

```
kubectl get storageclass --namespace=chat-app
```

Si no quieres tener que escribir el namespace en cada comando `kubectl` puedes cambiar el contexto así:

```
kubectl config set-context --current --namespace=chat-app
```
