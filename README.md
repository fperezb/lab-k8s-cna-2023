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
