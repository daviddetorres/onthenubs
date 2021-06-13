# Capitulo #13 (03-6-2021)
[Escúchalo aquí](https://youtu.be/raukRcuDHTc).

Hoy rompemos contenedores.

## Rompemos un contenedor con una bomba logica:
Para reproducir la demo, ejecuta esto:
```
docker run -d --rm -p 80:80 --memory="200m" --cpus="0.5" nginx
docker container ls
docker exec -it <NOMBRE-CONTENEDOR> /bin/bash

# Dentro del contenedor:
:(){ :|:& };:
```

Más info:
* [Cgroups en el kernel de linux](https://man7.org/linux/man-pages/man7/cgroups.7.html)
* [Configurar los limites en contenedores docker](https://docs.docker.com/config/containers/resource_constraints/)
* [Entendiendo la bomba logica](https://oper.io/?p=understanding_the_bash_fork_bomb)

## Rompemos un contenedor borrando root
Para reproducir la demo, ejecuta esto:
```
docker run -d --rm -p 80:80 nginx
docker container ls
docker exec -it <NOMBRE-CONTENEDOR> /bin/bash
rm -R --no-preserve-root /
```

Más info:
* [Overlay FileSystem en el kernel de linux](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html)
* [Como usa docker el overlay FS con las imágenes](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay-driver-works)
