
# Guía de Despliegue - Ejercicio-Web

Esta guía cubre el ciclo completo de desarrollo y despliegue para **ejercicio-web**. Incluye la simulación de trabajo colaborativo, la **configuración manual** de la infraestructura en AWS, y la automatización del despliegue con **GitHub Actions** tambien hay informacion sobre como acceder a la web y a la web/docs para er la documentacion..

## En este README esta recogido globalmente casi todo lo que concierne a los demas .md del proyecto


---

# Guía colabriacion para el despliegue de ejercicio-web

Esta guía cubre el ciclo completo de trabajo colaborativo para **ejercicio-web**.

---

## 1. Trabajo Colaborativo en GitHub

### Roles
*   **Anfitrión (imohahm):** El creador del repositorio original.
*   **Colaborador (ibrahim635z):** Quien hará los cambios y enviará las mejoras.

### Paso 1: Configuración Inicial (Anfitrión)
1.  Crea el repositorio en GitHub y sube el código base (`main`) con README.
2.  Voy a **Settings** > **Collaborators** > **Add people**.
3.  Invita al usuario de GitHub del **Colaborador**.

### Paso 2: Flujo de Trabajo (Colaborador)
El colaborador **NO** va trabajar directamente en la rama `main` del repo original.
1.  **Fork:** no es necesario en este caso al ser colaboradores.
2.  **Clonar:** Clona el **Repositorio** del anfitrion en el ordenador.
    ```bash
    git clone https://github.com/imohahm/ejercicio-web
    ```
3.  **Crear Ramas de trabajo "Saber Estar":** Creas  ramas para las diferentes funcionalidades algunos ejemplos de lo que hemos realizado.
    ```bash
    git checkout -b deploy (la que vamos a probar desplegando)
    git checkout -b release/test(para las pruebas unitarias)
    git checkout -b feature/estructura (estructura base)
    git checkout -b feature/frontend (codigo del frontend)


    instalar: npm init -y (genera el package.json)
    npm install jest jsdoc --save-dev (instala dependencias jest para pruebas jsdoc para documentar)
    añades a la sección scripts:
    "test": "jest",
    "docs": "jsdoc -c jsdoc.conf.json"
    npm install --save-dev jest-environment-jsdom
    
    ```
4.  **Trabajar:** Hacemos los cambios necesarios en todo momento, `git add .`, `git commit -m "..."`.
5.  **Subir Cambios:**
    ```bash
    git push origin feature/nueva-funcionalidad(la rama que corresponda)
    ```

### Paso 3: Pull Request (Colaborador -> Anfitrión)
1.  Realizas un push origin deploy 
2.  Desde Github Crea la PR comparando tu rama `feature` con la `main` del Anfitrión.

### Paso 4: Aceptar Cambios (Anfitrión)
1.  El Anfitrión revisa y acepta la PR (**Merge pull request**).

---

## 2. AWS S3: Configuración y Despliegue

* En mi caso lo hare a traves de un Bucket S3 LLamado **ejercicio-web**
* 
### A. Configuración Manual del Bucket (Infraestructura)
Antes de desplegar, necesitamos crear y configurar el bucket.

1.  **Crear Bucket:**
    *   Ve a AWS Console > **S3** > **Create bucket**.
    *   Nombre: `ejercicio-web` (debe ser único y llamarse igual que el dominio).
    *   Desmarcamos **"Block all public access"** y confirma la advertencia.
    *   le Damos a **Create bucket**.

2.  **Habilitar Alojamiento Web:**
    *   Entra al bucket > **Properties** > **Static website hosting** > **Edit**.
    *   Seleccionamos **Enable**.
    *   Index document: `index.html`.
    *   Guardar cambios.

3.  **Política de Bucket (Permisos Públicos):**
    *   Ve a **Permissions** > **Bucket policy** > **Edit**.
    *   Pega este JSON (cambia `ejercicio-web`):
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::ejercicio-web/*"
            }
        ]
    }
    ```

### B. Despliegue Automático (GitHub Actions)
Configura el workflow para que se despliegue solo.

1.  **Secrets en GitHub:** (Settings > Secrets > Actions)
    *   `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, `AWS_REGION`, `S3_BUCKET_NAME`.

2.  **Workflow (`.github/workflows/deploy-s3.yml`):**

```yaml

name: ejercicio-web Despliegue Secuencial en AWS S3

# Disparadores
on:
  # Disparador manual
  workflow_dispatch:

jobs:
  # PROBAR
  test:
    name: 1. Instalar y Probar
    runs-on: ubuntu-latest
    steps:
      - name: Clonar el repositorio
        uses: actions/checkout@v4

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22.x' 
          cache: 'npm' 

      - name: Instalar dependencias
        run: npm ci

      - name: Ejecutar pruebas (npm test)
        run: npm test

  # CONSTRUIR ARTEFACTOS
  build:
    name: 2. Construir Artefactos
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Clonar el repositorio
        uses: actions/checkout@v4

      - name: Configurar Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22.x' 
          cache: 'npm' 

      - name: Instalar dependencias
        run: npm ci
      - name: Crear archivo de configuración JSDoc
        run: |
          cat <<EOT >> jsdoc.conf.json
          {
              "source": {
                  "include": ["src"], 
                  "includePattern": ".+\\\\.js(doc|x)?$",
                  "excludePattern": "(^|\\\\/|\\\\\\\\)_"
              },
              "opts": {
                  "encoding": "utf8",
                  "destination": "./docs",
                  "recurse": true
              }
          }
          EOT
      - name: Generar documentacion
        run: npm run docs

      - name: Subir artefacto del sitio web
        uses: actions/upload-artifact@v4
        with:
          name: static-site
          path: src/ 

      - name: Subir artefacto de la documentación
        uses: actions/upload-artifact@v4
        with:
          name: documentation
          path: docs/
          
      - name: Subir artefacto de las imágenes 
        uses: actions/upload-artifact@v4
        with:
          name: images-folder
          path: images/

  # DESPLEGAR EN S3
  deploy:
    name: 3. Desplegar en AWS S3
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'workflow_dispatch'
    steps:
      - name: Configurar credenciales de AWS
        uses: aws-actions/configure-aws-credentials@v5.1.0
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Descargar artefacto del sitio web
        uses: actions/download-artifact@v4
        with:
          name: static-site
          path: ./sitio-web

      - name: Descargar artefacto de la documentación
        uses: actions/download-artifact@v4
        with:
          name: documentation
          path: ./sitio-docs
          
      - name: Descargar artefacto de las imágenes 
        uses: actions/download-artifact@v4
        with:
          name: images-folder
          path: ./sitio-imagenes

      - name: Desplegar sitio web en S3
        run: |
          aws s3 sync ./sitio-web/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

      - name: Desplegar documentación en S3
        run: |
          aws s3 sync ./sitio-docs/ s3://${{ secrets.S3_BUCKET_NAME }}/docs --delete

      - name: Desplegar imágenes en S3
        run: |
          aws s3 sync ./sitio-imagenes/ s3://${{ secrets.S3_BUCKET_NAME }}/images --delete


```

## Luego IONOS
- tenemos esta URL de AWS: http://ejercicio-web.ejercicio-web.es.s3-website-us-east-1.amazonaws.com/

- Para el DNS, tenemos que limpiarla. Quítandole el http:// del principio y la barra / del final.

- Lo que necesitamos: ejercicio-web.ejercicio-web.es.s3-website-us-east-1.amazonaws.com
- 
### 2. Configura el registro en IONOS
- Vamos a panel de DNS en IONOS > Tu dominio ejercicio-web.es > mi subdominio igual que el bucket ejercicio-web.ejercicio-web.es > DNS > Añadir registro.

- Seleccionamos CNAME.

- Nombre de host (Host Name): Escribe SOLO ejercicio-web

⚠️ Error que se podria cometer: Si escribes ejercicio-web.ejercicio-web.es aquí, IONOS creará ejercicio-web.ejercicio-web.es.ejercicio-web.es y no funcionará. Se escribe solo el subdominio.

Apunta a : ejercicio-web.ejercicio-web.es.s3-website-us-east-1.amazonaws.com

ahora introduciendo: ejercicio-web.pachangap.es Podras acceder a la Web.

Si introduces: ejercicio-web.pachangap.es/docs puedes comprobar la documentacion del proyecto


---
