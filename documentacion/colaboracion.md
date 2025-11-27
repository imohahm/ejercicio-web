# Guía colabriacion para el despliegue de PachangApp 🏐⚽️🏀

Esta guía cubre el ciclo completo de trabajo colaborativo para **PachangApp**.

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