# K6-Node.js Template
Basado en https://k6.io/docs/using-k6/modules/  

### Problema:  
K6 está hecho en Go y por tema de seguridad no soporta módulos de Node.js que resuelvan librerías externas. Por ejemplo, faker.js  

### Solución:
Los módulos que usan librerías externas deben convertirse a paquetes independientes y aislados. Esto se puede hacer con muchas librerías. K6 recomienda hacerlo con `webpack` (https://webpack.js.org/)

### Observaciones:
Agregar módulos externos a un proyecto de prueba tienen un impacto negativo en el rendimiento de cpu y memoria. K6 crea un máquina virtual Javascript separada por cada VU por lo que usar librerias externas puede ser un problema en pruebas que requieran grandes cantidades de VU. Este repositorio utiliza las dependencias de Babel y CoreJS para disminuir el impacto de usar librerías externas usando `--compability-mode=base`. Para más información sobre modo de compatibilidad y rendimiento: https://k6.io/docs/using-k6/javascript-compatibility-mode/#base 

### Implementación:
Siguiendo el paso a paso de la documentación de k6:  
1. Se debe crear un proyecto node: `npm init`  
2. Instalar las dependencias necesarias:  
```
npm install --save-dev \
    webpack \
    webpack-cli \
    k6 \
    babel-loader \
    @babel/core \
    @babel/preset-env \
    core-js

```
3. Crear un archivo `webpack.config.js` y agregar la siguiente configuración:

```
const path = require('path');

module.exports = {
  mode: 'production',
  entry: {
    <bundle_name>: <path to .js to pack>
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    libraryTarget: 'commonjs',
    filename: '[name].bundle.js',
  },
  module: {
    rules: [{ test: /\.js$/, use: 'babel-loader' }],
  },
  target: 'web',
  externals: /k6(\/.*)?/,
};

```
Tener en cuenta que hay completar la configuración `entry` de acuerdo al proyecto. Por ejemplo. En este repositorio lo completamos con los datos del archivo hello_world.js.  

4. Configurar los scripts de test al `package.json`
```
...
  "scripts": {
    "pretest": "webpack",
    "test": "k6 run --compatibility-mode=base <path to bundle.js>" 
  }
...  
```
Tener en cuenta que hay completar el path a los archivos `bundle.js` de acuerdo al proyecto. Estos archivos se crean automáticamente cuando ejecutamos `webpack` en una carpeta llama `dist`. Por ejemplo. En este repositorio lo completamos con los datos del archivo `hello_world.bundle.js`.

5. Ejecutar los test con: `npm run test` 

 Al ejecutar los test se deberían crear los paquetes en la carpeta `dist`. Si no se crean se los puede crear con el comando `npm run pretest`

 **Aclaración**: El archivo `.babelrc` es un archivo de configuración de Babel. No es necesario al menos que quieran cambiar alguna configuración de compilación de Javascript y mejorar el rendimiento. Para más info ver: https://babeljs.io/ 



