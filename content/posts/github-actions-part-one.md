+++
title = "Github Actions - Actions Favoritos Parte 1"
date = "2022-07-06T16:51:04-05:00"
author = "diegotony"
cover = ""
tags = ["github-actions"]
keywords = ["upload-artifact", "download-artifact"]
description = "GitHub Actions es un PaaS, perteneciente a Github, es una plataforma que nos permite automatizar algunas procesos para nuestro codigo"
showFullContent = false
readingTime = false
hideComments = false
+++
GitHub Actions es un PaaS, perteneciente a Github, es una plataforma que nos permite automatizar algunas procesos para nuestro codigo, como por ejemplo:

- Pruebas Unitarias (test)
- Construcion (build)
- Despliegue (deploy)
- Documentacion (docs)
- Enviar un mensaje a Slack (webhook)

Existen algunas cosas mas, pero esto es para lo que se ocupa actualemente. A continuacion te presento los actions:

## upload-artifact
[Repositorio](https://github.com/actions/upload-artifact)

Si has ocupado GitLab CI sabes existe una manera de pasar artefactos de un job a otro, este action es la traduccion de eso, digamos que se requiere que pases el build de tu aplicacion a otro job por lo que este accion te facilita eso, **OJO** , este solo sube, necesitas de otro accion para descargarlo, mas adelante te lo presentare. Aqui te presento como se veria en un job:

```yaml
  - name: Create a file
    run: echo "I won't live long" > my_file.txt

  - name: Upload Artifact
    uses: actions/upload-artifact@v3
    with:
      name: my-artifact
      path: my_file.txt
      retention-days: 5 
```
Algunas consideraciones antes de usar este job:

- Puedes subir tanto carpetas como archivos
- Puedes establecer un tiempo de existencia para el artefacto este disponible
- En caso que lo quieras usar en otro job, debes de referirte a el por el nombre que pusiste en el parametro de **name** como en este caso es `my_file.txt`.


## download-artifact
[Repositorio](https://github.com/actions/download-artifact)

Bueno como ya te lo mencione en el action anterior, este job nos ayuda a recuperar los artefactos que subimos con el action anterior, es muy sencillo se lo hace de la siguiente manera: 

```yaml
- uses: actions/checkout@v2

- uses: actions/download-artifact@v3
  with:
    name: my-artifact
    
```
No hay consideraciones para este job
