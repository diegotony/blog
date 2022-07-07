+++
title = "Pipeline para deployar a Cloudfront"
date = "2022-07-06T16:51:04-05:00"
author = "diegotony"
cover = ""
tags = ["github-actions","aws","cloudfront"]
keywords = ["cloudfront", "github-actions"]
description = "AWS Cloudfront es un servicio para servir contenido estatico o tambien llamado CDN, que resulta muy util cuando queremos deployar una web page bajo demanda con un bajo costo.Comparto un pipeline para deployar a AWS Cloudfront Usando Github Actions"
showFullContent = false
readingTime = false
hideComments = false
draft: true
+++
AWS Cloudfront es un servicio para servir contenido estatico o tambien llamado CDN, que resulta muy util cuando se desea deployar una pagina web bajo demanda con un bajo costo. A continuacion les presento un pipeline para deployar a AWS Cloudfront Usando Github Actions.

Que debe hacer nuestro pipeline:

0. Configuraciones generales del job
1. Check de nuestro codigo
2. Build de nuestra aplicacion
3. Setup de las credenciales de AWS
4. Sync del contenido a nuestro bucket de S3
5. Crear una Invalidation
6. DONE

## Desarrollo
### 0. Configuraciones generales del job
En este paso se define que evento va hacer que el pipeline se **"encienda"**. En este caso cuando mandemos cambios directamente al branch **main**.
```yaml
name: Deploy to Cloudfront

on: 
  push:
    branches:
      - main
```
Despues se define el o los jobs que van correr, como por ejemplo:
```yaml
jobs:  # <= Dentro de este bloque se insertan los jobs que quiera dentro de este workflow                                     
  deploy: # <= Aqui se nombra el primer job
    name: aws-cloudfront # <= el nombre del job
    env: # <= Se inserta las variables de entorno que se van a ocupar de manera global, en este caso las credendiales de AWS
      BUILD_DIR: ${{ secrets.BUILD_DIR }}
      AWS_DISTRIBUTION_ID:  ${{ secrets.AWS_DISTRIBUTION_ID }}
      AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}

    runs-on: ubuntu-latest # <= En que sistema operativo se va a correr el job, en este ubuntu, que es ademas el mas barato 
    steps:
```
### 1. Check de nuestro codigo
Aqui lo que hace basicamente es descargar nuestro codigo al job
```yaml
    - name: Check out code  
      uses: actions/checkout@v3
```
### 2. Build de nuestra aplicacion
Para este pagina web se usa **NodeJS** como lenguaje de programacion, por lo que se tiene que instalar dentro del job, se lo hace de la siguiente manera.
```yaml
    - uses: actions/setup-node@v3
      with:
        node-version: 18
```

Se procede a hacer una instalacion de las dependencias.

```yaml

    - name: Install Dependencies
      run: |
        npm install
```
Se hace un build de la aplicacion, varia mucho del framework, lo que se usa para el ejemplo es **NextJS**

```yaml
    - name: Build NextJs App
      run: |
        npm run build --if-present 
```

### 1. Check de nuestro codigo
Aqui lo que hace basicamente es descargar nuestro codigo al job
```yaml
    - name: Check out code  
      uses: actions/checkout@v2
```
### 1. Check de nuestro codigo
Aqui lo que hace basicamente es descargar nuestro codigo al job
```yaml
    - name: Check out code  
      uses: actions/checkout@v2
```
### 1. Check de nuestro codigo
Aqui lo que hace basicamente es descargar nuestro codigo al job
```yaml
    - name: Check out code  
      uses: actions/checkout@v2
```
### 1. Check de nuestro codigo
Aqui lo que hace basicamente es descargar nuestro codigo al job
```yaml
    - name: Check out code  
      uses: actions/checkout@v2
```
