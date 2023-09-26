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


## Paso 3

Ahora vamos a crear la base postgres, esta se define en los archivos bajo la carpeta `postgres`.

El archivo `postgres-secrets.yaml` contiene la clave usada en la base de datos, que se encuentra codificada en base64, que es la forma que tiene K8s para codificar los secrets. El valor que está almacenado se puede cambiar, siempre que lo codifiques en base64.

El archivo `postgres-service` define el servicio postgres, que será usado posteriormente por migrations y los servicios. Acá se define el port que usará el servicio, entre otros parámetros.

Por último el archivo `postgres-statefullset.yaml` crea un `StatefulSet` que es la forma de crear objetos persistentes, en particular, acá configuramos los volúmenes persistentes donde quedará la base de datos.

Aplicamos estos objetos con el siguiente comando:

```
kubectl apply -f postgres
```

## Paso 4

Vamos a ejecutar la migración de datos con flyway usando un Job.

El job ya se encuentra configurado en el archivo `migrations/migration-job.yaml`.

Ese archivo contiene un objeto `config-map` :

```
apiVersion: v1
data:
  V1__Create_users_table.sql: |
    CREATE TABLE users(
      id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
      name VARCHAR(100) NOT NULL,
      email VARCHAR(255) NOT NULL UNIQUE,
      password VARCHAR(255) NOT NULL,
      birthday DATE
    );
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: migration-config
```

Fíjate como hemos "copiado" el contenido del original de migración flyway dentro de la sección `data:` de este archivo.

Aplicaremos la migración ejecutando el siguiente comando:

```
kubectl apply -f migration
```


Puedes verificarlo "entrando" a la base postgres. En el dashbaord selecciona `Pods` en el extremo derecho del Pods `postgres-0` aparecen tres puntos verticales, este es un menú, ahi selecciona la opción `Exec`, esto abre un shell en el pod y puedes ingresar a la base de datos haciendo:

```
psql -U postgres
```

Entrando en la base de datos puedes realizar consultas para ver los datos que poblamos con la migración.

Puedes hacer lo mismo en la linea de comandos del siguiente modo:

```
kubectl exec --stdin --tty  postgres-0 -- /bin/bash
root@posrgres-0:/# psql -U postgres
```
