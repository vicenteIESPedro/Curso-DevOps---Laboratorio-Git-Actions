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
Como se observa hay un error en la ejecucuón de los Test, (no es parte de este laboratorio) el depurar el código.  





