## LABORATORIO GIT ACTIONS ##
En este laboratorio voy a generar workflows de Git Actions para demostrar como usar Git para Ci/CD  
Parto del proyecto Node.js de la carpeta hangman-front  
Los workflows deben colocarse en nuestro proyecto en la carpeta /.github/workflows
  
### EJERCICIO 1: WORKFLOW CI ###
Para este ejercicio voy a crear el fichero .github/workflows/ci.yaml:  
```
name: Integracion continua

#definicion de los eventos que lanzarán la acción
on:
  # al colocar una version(commit) en la rama main
  push:
    branches: [ main ]
    # obligo a que solo se lance la acción si hay cambios en la carpeta hangman-front
    paths: [ "hangman-front/**" ]

  # al hacer una PullRequest sobre la rama main
  pull_request: 
    branches: [ main ]
    paths: [ "hangman-front/**" ]

# Trabajos a realizar en el workflow
jobs:
  # trabajo build
  build:
    # usando una máquina virtual ubuntu
    runs-on: ubuntu-latest
    
    # pasos que se realizarán
    steps:
      # llevar el repositorio a la máquina virtual usando una
      # action de github llamada checkout
      - name: Checkout #nombre que le ponemos al paso
        uses: actions/checkout@v6  # acción que usamos

      # establecer la versión de node.js a la version 18 usando la
      # action de github setup-node
      - name: Set up node.js version
        uses: actions/setup-node@v6
        with:   #parametros de la acción
          node-version: 18
      
      # construir el proyecto de node.js que hay en la carpeta hangman-front
      - name: Build
        working-directory: ./hangman-front  #directorio de trabajo
        run: | # Ejecutamos unas instrucciones en la máquina virtual. La | indica varias lineas
          npm ci
          npm run build
  
  # segundo trabajo: testear el programa
  test:
    runs-on: ubuntu-latest 
    # indica que es necesario que se haya ejecutado correctamente el trabajo build
    needs: build
    
    steps:
      - name: Checkout
        uses: actions/checkout@v6
      - name: Set up node.js version
        uses: actions/setup-node@v6
        with:
          node-version: 18
      - name: Test
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run test
```

Para activar este workflow en el repositorio debo seguir estos pasos:  
1. Crear otra rama, commit en esta rama y subirlo a github.
Crearé el archivo ci.yaml en una rama llamada "rama-ci". Además, incluyo en el commit algún archivo de la carpeta hangman-front que haya modificado y subo este commit al repositorio.
<img width="877" height="448" alt="ejercicio 1-commit" src="https://github.com/user-attachments/assets/53a27359-6ea1-4e8d-be21-e407f5f07fcc" />
  
2. Realizar la PullRequest
Desde github, pestaña PullRequest, activamos la PullRequest de la rama "rama-ci" a main. Esto Activará el workflow
<img width="962" height="714" alt="ejercicio 1-iniciar pullrequest" src="https://github.com/user-attachments/assets/2c6fe0fd-313f-4e9a-a146-1de8f60c8582" />  
  
Al pulsar para crear el PullRequest nos aparecerá una nueva ventana en la que, si todo va bien, podremos observar la ejecución del workflow
<img width="917" height="703" alt="ejercicio 1-ejecución de workflow" src="https://github.com/user-attachments/assets/023bc1dc-5b79-40fd-abc0-341860d54258" />  
  
Pasado un tiempo, finalizará la ejecución del workflow
<img width="909" height="714" alt="ejercicio 1-ejecución de workflow-error" src="https://github.com/user-attachments/assets/f971b2c3-5f24-493f-af8e-35bc51755a90" />  
  
Como se observa hay un error en la ejecución de los Test, (no es parte de este laboratorio) el depurar el código.
Podemos acceder a la salida del workflow pulsando directamente en el job correspondiente.
<img width="1540" height="707" alt="ejercicio 1-detalles ejecución workflow" src="https://github.com/user-attachments/assets/aa33cb5d-1b5a-4d3f-815a-daabc6af76b0" />
  
3. Consulta de la PullRequest
Podemos dejar abierta la PullRequest. Mientras esté abierta cualquier commit en la misma rama, si se cumplen las condiciones, hará que se ejecute el workflow.
Se podrá consultar la Pullrequest en la pestaña PullRequest de github.
<img width="613" height="342" alt="ejercicio 1-pullrequest" src="https://github.com/user-attachments/assets/d7c7162d-66f8-4090-9618-479efe4ec3df" />
   
Y acceder a la propia PullRequest pulsando en ella.

5. Cerrar la PullRequest  
Una vez estemos en la PullRequest, hacemos Merge, cerrándola y borrando la rama.
<img width="970" height="608" alt="ejercicio 1-merge" src="https://github.com/user-attachments/assets/00518313-6b64-45f0-b81e-a39e20b5ca2c" />
Además, borramos la rama en el git local y actualizamos la rama main.  
<img width="694" height="491" alt="ejercicio 1-fin git local" src="https://github.com/user-attachments/assets/c024d02a-ca6a-4b57-9d9f-8a40379ccc36" />

### EJERCICIO 2: WORKFLOW CD ###
Este ejercicio me permitirá publicar la imagen Docker que se genera con el código en el registro de GITHUB.  
Tengo que tener en cuenta varias cosas:  
1. El repositorio en el que se almacenará la imagen es ghcr.io  
2. Debo tener un Token de GitHub para que se pueda publicar en mi espacio de GitHub.  
Para ello, debo acceder a mis Settings de GitHub, y en el apartado Developer Settings/Personal access Tokens definir un token.
<img width="794" height="478" alt="ejercicio2-token" src="https://github.com/user-attachments/assets/8a4f981f-3683-45fe-9ce9-3c7ae6884e34" />

El siguiente paso es definir el archivo de workflow. Yo he definido .github/workflows/cd.yaml  
```
# defino variables que usaré después
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
    
# defino los trabajos a realizar
jobs:
  # se usará una máquina virtual de ubuntu para el despliegue
  construir-publicar-imagen:
    runs-on: ubuntu-latest

    # defino los permisos necesarios. Es importante que para packages sea write (escritura). Si nó no 
    # podremos colocar la imagen que creamos
    permissions:
      contents: read
      packages: write
      attestations: write
      id-token: write

    #definimos los pasos
    steps:
      # subo el repositorio a la máquina virtual
      - name: Checkout
        uses: actions/checkout@v6

      # me conecto al repositorio de GitHub. Usaré el usuario de Github con el que estoy
      # conectado y el token que he definido en Github en setting como contraseña. 
      - name: Docker login
        uses: docker/login-action@v4
        with:
          # repositorio
          registry: ${{ env.REGISTRY }}
          # usuario, el de Github
          username: ${{ github.actor }}
          # contraseña, el token que tengo definido
          password: ${{ secrets.GITHUB_TOKEN }}

      # obtengo metadatos necesarios en pasos posteriores
      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v6
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      # configuro docker para que permita versiones en varios SO
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      # construyo y publico la imagen docker que se genera para el proyecto hangman-front
      - name: Build and push Docker image
        uses: docker/build-push-action@v7
        with:
          # trabajaré sobre la carpeta hangman-from
          context: ./hangman-front
          # publico la imagen
          push: true
          # defino etiquetas
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

Una vez definido el workflow debemos subirlo a nuestro repositorio. Además, vamos a ejecutarlo y ver los resultados. Para todo este proceso se siguen los siguientes pasos:  

1. Subir al repositorio el archivo de workflow  
Desde mi máquina local, genero un commit y realizo un push sobre Github.
<img width="646" height="352" alt="ejercicio2-push" src="https://github.com/user-attachments/assets/4f5c2db3-ee26-4778-827b-043accfc294b" />

2. PullRequest sobre la rama principal  
Genero el PullRequest de la rama 'rama-cd' a la rama main. Cierro la PoolRequest para que aparezca el workflow en Actions. Al no haber realizado ningún cambio en archivos de hangman-front no se lanzará el workflow del ejercicio 1. 
<img width="604" height="691" alt="ejercicio2-pullrequest" src="https://github.com/user-attachments/assets/782b4a7c-7e92-408c-b58c-e3969b5d607e" />

3. Verifico que ya tengo el workflow Despliegue continuo disponible.  
En la pestaña Actions puedo comprobarlo.  
<img width="795" height="515" alt="ejercicio2-accion" src="https://github.com/user-attachments/assets/38469cde-02f2-421e-92e7-b042482fc509" />  

4. Ejecutar el workflow Despliegue Continuo  
En Actions, selecciono el workflow y pulso en el botón Run workflow. Puedo indicar la rama que voy a usar.  
<img width="797" height="477" alt="ejercicio2-ejecutar accion" src="https://github.com/user-attachments/assets/d3b826d5-bf0a-4b48-ba89-764c3b6e4ea4" />

Una vez lanzada podremos ver el estado de ejecución.  
<img width="754" height="502" alt="ejercicio2-ejecutar correcta" src="https://github.com/user-attachments/assets/52dced4b-0a98-434e-b75c-55d9d51082ce" />  

Y los detalles.  
<img width="779" height="685" alt="ejercicio2-ejecutar correcta detalles" src="https://github.com/user-attachments/assets/9378d975-9e82-4023-a146-b0cf3e9ecc01" />  

5. Borrar rama en Github y actualizar rama main local
Borro la rama "rama-cd" y en mi equipo realizo un pull sobre la rama main










 



  

 





