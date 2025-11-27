  # Guia para la instalacion del generador de documentacion en nuestro caso JSdoc
  
* instalar: npm init -y (genera el package.json)
* npm install jest jsdoc --save-dev (instala dependencias jest para pruebas jsdoc para documentar)
* añades a la sección scripts:
* "test": "jest",
* "docs": "jsdoc -c jsdoc.conf.json"
* npm install --save-dev jest-environment-jsdom
   