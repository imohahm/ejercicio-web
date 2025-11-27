# Guia para **Git, GitHub y GitActions**

## 1. Comandos básicos

### **Configuración**

```bash
git config --global user.name "Nombre"
git config --global user.email "email"
git config --list
```

---

## 2. El saber estar

1. Rama Main (no se toca)
2. Rama Develop (principal de desarrollo)
3. Rama Feature (para desarrollar partes especificas)
4. Rama Release (la de las pruebas unitarias. Paso previo para mergear con main)
5. Rama Hotfix (errores reapidos se mergea directamente a main)

---

##  3. Crear o clonar repos

```bash
git init
git clone <url>
```

---

##  4. Workflow diario 

```bash
git status
git add archivo
git add .
git commit -m "mensaje"
git push origin main
git pull
```

Diferencias y log:

```bash
git diff
git diff --staged
git log --oneline --graph
```

Restaurar:

```bash
git restore archivo
```

---

##  5. Ramas 

Crear / cambiar:

```bash
git switch -c nueva-rama
git switch rama
```

Borrar:

```bash
git branch -d rama
```

**Merge** (une ramas):

```bash
git switch main
git merge mi-rama
```

Resolver conflictos:
Git marca: `<<<<<<<`, `=======`, `>>>>>>>`
Elegir versión → guardar → commit.

---

## 6. Conectar a GitHub 

Añadir remoto:

```bash
git remote add origin <url>
git remote -v
```

Primera subida:

```bash
git branch -M main
git push -u origin main
```

Sincronizar:

```bash
git fetch      # descarga sin mezclar
git pull       # descarga + merge
git push
```

---

##  7. Reset y recuperación rápida

```bash
git reset --soft HEAD~1   # vuelve un commit manteniendo staging
git reset --mixed HEAD~1  # mantiene archivos
git reset --hard HEAD~1   # ⚠️ borra cambios
git reflog                # historial oculto para recuperar
```

---

## 8. Gitignore (lo que sí cae en examen)

Ejemplo:

```
*.log
node_modules/
.env
```

Crear global:

```bash
git config --global core.excludesfile ~/.gitignore_global
```

---

##  9. GitHub: colaboración (básico y suficiente)

### **Fork**

Copia del repo en tu cuenta para modificar sin tocar el original.

### **Pull Request**

Pides integrar tus cambios:

1. Cambios → commit → push.
2. Desde GitHub → **New Pull Request**.
3. El dueño revisa y acepta.

### **Issues**

Tareas, bugs o mejoras.
Se pueden etiquetar y asignar.

---

## 10. Documentación y Tests

En la terminal iniciamos un nuevo proyecto e instalamos las dependencias necesarias

```bash
nmp init -y

npm install --save-dev jsdoc jest-environment-jsdom jest
```

En el package.json añadimos lo siguiente

```json
"scripts": {
"test": "jest",
"doc": "jsdoc -r -d docs ."
}
```

Para luego ejecutar los test y la documentacion se ejecuta con ` npm test` y `npm run doc`
