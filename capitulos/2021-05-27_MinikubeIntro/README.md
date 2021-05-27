# 2021-05-27 - Introducción a MiniKube

En este episodio hemos hecho una introducción a MiniKube, una versión ligera de Kubernetes, ideal para aprender y hacer pruebas en local.

En concreto, hemos convertido una aplicación PHP en un contenedor, la hemos desplegado en Docker, y luego en MiniKube.


## ¿Qué son los contenedores?

Paquetes que encapsulan los archivos y recursos de una aplicación, junto con las instrucciones para ejecutarla.

Los contenedores tienen dos principales ventajas:

- Si funciona en local, funciona en producción. Ya que la configuración para ejecutar es la misma.
- Los programas están aislados dentro de su contenedor. Si hackean un contenedor, no tienen acceso inmediato a todo el servidor.

## Diferencias entre contenedores y máquinas virtuales.

Las máquinas virtuales también comparten estas ventajas, sin embargo ejecutan un sistema operativo completo, mientras que el contenedor se ejecuta normalmente en el mismo sistema huesped.

Una máquina virtual necesita más recursos.

Por otro lado, al contener un sistema operativo completo tienen una carga extra de mantenimiento. No solo hay que actualizar y asegurar la aplicación, si no todos los binarios que se incluyen en su sistema operativo.


## Imágenes vs. contenedores en ejecución

Primero "cocinamos" la imágen de un contenedor.

Esa imágen la podemos ejecutar varias veces, teniendo varias instancias del mismo programa en ejecución.


## Requisitos

Para poder seguir los pasos de la demo, tienes que:

1. Activar las instrucciones de virtualización en tu ordenador.
2. Instalar [Docker for Desktop](https://www.docker.com/products/docker-desktop).
3. Instalar [Minikube](https://minikube.sigs.k8s.io/docs/start/).


## Creando una imagen de contenedor

En `php-container` está todo lo necesario para construir el contenedor que hemos usado en el capítulo de hoy.

```
$ cd php-container
```

Allí encontrarás los siguientes archivos:
- **index.php**: El programa php.
- **Dockerfile**: Las instrucciones para crear y ejecutar el contenedor.
- **000-default.conf**: Configuración específica de apache.
- **start-apache.sh**: Un script que iniciará nuestra aplicación dentro de Apache.

Con `docker build` creamos la imagen del contenedor:

```
docker build -t my-php-app .
```

Donde `-t` indica una etiqueta que identificará al contenedor.

Normalmente los contenedores se identifican con el hash sha256 de su contenido. Es un número largo y difícil de memorizar, por lo que las etiquetas hacen esta tecnología un poco más apta para humanos.

Con `docker run` podemos ejecutar nuestro contenedor:

```
docker run -p80:80 -it --rm --name my-running-app my-php-app
```

La opción `-p80:80` mapea el puerto 80 del contenedor con el de nuestra máquina local. Accediendo a localhost:80 accederemos a la app del contenedor.

Con `-it` iniciamoss el proceso en modo de consola interactivo. No es necesario, creo, pero en el ejemplo que he usado lo ponía. Funciona, así que no he probado a quitarlo :).

El flag `--rm` (cleanup) borrará todos los recursos asociados al contenedor cuando lo paremos. Siendo un contenedor de pruebas, nos evita hacer limpieza entre prueba y prueba.

Con `--name` indicamos cómo queremos identificar al nuevo contenedor en ejecución. Como podemos ejecutar la misma imagen varias veces a la vez, podemos tener varias versiones numeradas "-01" para difefenciarlas.

Por último `my-php-app` es el nombre de la imagen a ejecutar.


## Los problemas de usar Docker "a pelo"

Con las instrucciones anteriores ya lo tenemos. Accedemos a `localhost:80` y vemos nuestra aplicación ejecutándose.

Pero esto no funcionará bien para una aplicación compleja.

Una aplicación compleja puede estar formada por cientos de contenedores, que tendremos que coordinar a través de varios servidores. Para complicar más las cosas, cada contenedor tiene su propia dirección IP.

¿Y si un contenedor falla? ¿Tenemos que apagarlo y encenderlo manualmente?

Para eso tenemos ¡Kubernetes! (y otros **orquestradores de contenedores**).

A Kubernetes le damos nuestro estado deseado: "Quiero 3 servidores web, un balanceador de carga, dos bases de datos…". Y el orquestrador se encarga de repartir la carga entre los servidores (nodos) disponibles, y de que siempre haya en ejecución lo que tú quieres.

Por ejemplo, si un nodo falla y sus contenedores ya no están disponibles, Kubernetes inicia nuevos contenedores en el resto de nodos para compensar la carga desaparecida.


## Desplegando nuestro contenedor en Kubernetes.

Antes de continuar asegúrate de que el contenedor de Docker ya no se está ejecutando.

En el episodio usamos VirtualBox como "driver" de minikube. O en otras palabras, minikube creará una máquina virtual con Linux en VirtualBox. En esa máquina virtual instalará Kubernetes, y sobre esa máquina virtual ejecutará los contenedores.

De esta forma evitamos instalar cosas raras en nuestro ordenador principal, y podemos hacer pruebas más rápido.

En un entorno de producción lo más normal es que instalásemos Kubernetes sin una máquina virtual.


### Iniciando el cluster Kubernetes

Para iniciar el cluster de Kubernetes hacemoss:

```
minikube config set driver virtualbox
minikube start
```

Es posible que minikube se queje de que las instrucciones de virtualización no están activadas. Si estás seguro de que están activadas, puede que se trate de un bug, prueba usando la opción `--no-vtx-check` así: `minikube start --no-vtx-check`.


### Usando un repositorio local

Lo normal es que el ordenador en el que se desarrolla un contenedor, no sea el mismo en el que se ejecuta. Un desarrollador programa en su ordenador, pero luego se usa un servidor para desplegar en producción.

Por eso, lo normal es que las imágenes de contenedores se guarden en repositorios, accesibles tanto al equipo de desarrollo como al servidor de producción.

Cuando pidamos a Kubernetes que despliege nuestra imagen, Kubernetes la buscará en uno de esos repositorios.

Así que deberíamos publicar la imagen que hemos creado.

Peeeero, aquí estamos haciendo una prueba rápida. Así que vamos a engañar a Kubernetes para que cargue la imagen desde nuestro propio ordenador.

Para eso ejecutamos:
```
# En terminales unix
eval $(minikube docker-env)

# En windows PowerShell
minikube docker-env | Invoke-Expression
```

Y volvemos a construir la imagen:
```
docker build -t my-php-app .
```


## Desplegando nuestra imágen

Si ejecutamos:

```
minikube dashboard
```

Se abrirá en un navegador un escritorio de Minikube, donde podemos investigar qué está desplegado. Úsalo para ver cómo afecta al clúster los siguientes comandos.

Podemos desplegar nuestra imágen con `kubectl run`:

```
kubectl run test-php --image=my-php-app --image-pull-policy=Never --port=80
```

Donde:

- `test-php` es el nombre del ojeto que se cree en Kubernetes.
- `--image` nos sirve para señalar la imagen a desplegar.
- `--image-pull-policy=Never` sirve para que descargue la imagen del repositorio local y no la busque en remoto.
- `--port=80` hará que el puerto `80` que usa nuestra aplicación esté disponible en el contenedor.

Verás que se ha creado un `Pod` llamado `test-php`.

Para hacer accesible nuestra aplicación fuera del cluster, tenemos que exponer el puerto `80` al exterior:

```
kubectl expose pod test-php --type=NodePort --port=80
```

Verás que se ha creado un `Service` llamado `test-php`.

Para probar la aplicación, pon en un navegador la url que devuelve este comando:
```
minikube service test-php --url
```

## Otros enlaces de interés:

* [https://www.docker.com/](https://www.docker.com/)
* [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
* [https://semaphoreci.com/community/tutorials/dockerizing-a-php-application](https://semaphoreci.com/community/tutorials/dockerizing-a-php-application)
* [https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube](https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube)
