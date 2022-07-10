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
draft = false
+++
AWS Cloudfront es un servicio para servir contenido estatico o tambien llamado CDN, que resulta muy util cuando se desea deployar una pagina web bajo demanda con un bajo costo. A continuacion les presento un pipeline para deployar a AWS Cloudfront Usando Github Actions.

Que debe hacer nuestro pipeline:

0. Configuraciones generales del job
1. Check de nuestro codigo
2. Build de nuestra aplicacion
3. Setup del Role de AWS
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
| Algo importante es que a veces sabe dar error el el build cuando se genera los proyectos por medio de CLI's del framework, como por ejemplo angular-cli o react-cli, por lo que se necesita instalar este CLI antes de hacer el build de manera global, como por ejemplo `npm install -g @angular/cli` 
### 3. Setup del Role de AWS

Se hace la configuracion del Role de AWS para desplegar el proyecto por medio de GitHub Actions, a mi parecer es la manera mas segura, [aqui](https://github.com/aws-actions/configure-aws-credentials#assuming-a-role) te dejo como configurar el Role o en tal caso que quieras hacerlo con crendenciales presiona [aqui](https://github.com/aws-actions/configure-aws-credentials).
```yaml
    - name: Configure AWS Role
      uses: aws-actions/configure-aws-credentials@v1
      with:
        role-to-assume: ${{ secrets.AWS_ACCESS_ROLE }}
        role-session-name: github-action-job
        role-duration-seconds: 900
        aws-region: us-east-1
```
Algo importante a mencionar es que si usaste la configuracion con el Role, debes agregar el ARN del Roles en los SECRETS de GitHub Actions, dentro del step debes de poner este bloque.
```yaml
    permissions:
      id-token: write
      contents: read
```

Finalmente se veria asi
```yaml
    - name: Configure AWS Role
      uses: aws-actions/configure-aws-credentials@v1
      permissions:
        id-token: write
        contents: read
      with:
        role-to-assume: ${{ secrets.AWS_ACCESS_ROLE }}
        role-session-name: github-action-job
        role-duration-seconds: 900
        aws-region: us-east-1
```

### 4. Sync del contenido a nuestro bucket de S3

Ahora se sube el build dentro del bucket de S3, para eso se necesita el URL del Bucket, por lo que el url se veria mas o menos asi `s3://my-source-bucket`, algo mas para agregar es que en la etapa de build, no necesariamente siempre va a generar el el diretorio `build/` como resultado del el build, en alguno casos tambien se same llamar `dist/` o `out/`, por lo que te recomiendo correr en local el comando para que sepas que directorio vas a subir. Te recomiendo poner el nombre de bucket en los SECRETS de GitHub Actions.  
```yaml
    - name: Sync S3 Bucket
      run: |
        echo "Save in S3 Bucket"
        aws s3 sync public $AWS_S3_BUCKET
```
### 5. Crear una Invalidation
Y por ultimo se procede a invalidar la version subida en **Cloudfront**, *Por que es necesario esto?*  Ya que cada vez que se despliega una version nueva de la pagina web, Cloudfront lo guarda en cache dentro de su servicio, para optimizar la cargar del sitio, por lo que se necesita decirle que tome la nueva version en el bucket, por lo que se invalida la que previamente fue subida, y se valida la actual, para esto se necesita el ID del Cloudfront.
```yaml
    - name: Deploy Invalidation 
      run: |
        echo "Deploy to Cloudfront"
        aws cloudfront create-invalidation --paths /* --distribution-id $AWS_DISTRIBUTION_ID
```

En caso que estes familiarizado con los Reusable Jobs en Github puedes hecharle un ojo a este [job](https://github.com/diegotony/gh-pipelines/blob/main/.github/workflows/aws-cloudfront.yml).