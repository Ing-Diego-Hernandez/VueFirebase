# Storage

## Atención acuérdate de intalar los Módulos

Instala los modulos, vete a terminal, nuevo terminal y escribe:
```
npm install
```

## Obtener Imagen

Antes de comenzar voy a cambiar copiar y pegar el archivo Home.vue a About.vue

- Solo tenemos que modificar el nombre de en exportar

```js
export default {
  name: 'About',
```

En Home.vue pegaremos **v-for** del componente **Cards** de Bootstrap y el método de obtenerDatos de Firestore.

TEMPLATE CARDS

```html
<template>
    <div>
        <Navbar/>
        <div class="container my-5">
            <div class="row row-cols-1 row-cols-md-3 g-4">
                <div class="col" v-for="(item, index) in usuarios" :key="index">
                    <div class="card">
                    <img :src= "item.foto" class="card-img-top">
                    <div class="card-body">
                        <h5 class="card-title"> {{item.nombre}}</h5>
                        <p class="card-text"> {{item.correo}}</p>
                    </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
```

SCRIPT obtenerDatos / getDocs

```js
<script>
import Navbar from '../components/Navbar.vue'
import { collection, getDocs } from 'firebase/firestore/lite';
import { db } from "../main";
export default {
    name: 'Home',
    components: { 
      Navbar,   
    },
    data() {
        return {
          usuarios: [],
          usuario: {
            nombre: '',
            correo: '',
            foto: ''
          }
        }
    },
    methods:{
  // GET
    async obtenerDatos () { 
      const querySnapshot = await getDocs(collection(db, "usuarios"));
        querySnapshot.forEach((doc) => {
        let usuario = doc.data()
        usuario.id = doc.id
        this.usuarios.push(usuario)
        console.log(usuario)
      });
    }
    },
    mounted() {
        this.obtenerDatos();
    },
}
</script>
```

### Template Obtener Imagen

En la parte de **template** añadimos un **input type="file"** que nos permite buscar un archivo a través del **finder** antes del botón **Guardar**.

```html
<div class="input-group my-3">
    <input type="file" @change="buscarImagen($event)">
</div>
```
Si queremos previsualizar la imagen utilizamos el dato **datoImagen**, después del botón Guardar.

```html
<div class="mt-4">
    <img :src="datoImagen">
</div>
```

### Script Obtener Imagen

- Buscar la imagen y guardarla en una variable **file** y el contenido **datoImagen**, y así visualizar el contenido de la imagen. Fíjate que también utilizamos un condicional para verificar que el tipo de archivo sea válido (jpeg, png). 

DATA

Añadimos **file y datoImagen**

```js
data() {
    return {
      file: null,
      datoImagen: null,
}
```
Obtenemos el archivo y lo guardamos en data, comprobamos el tipo de archivo si es válido y obtenemos la url para poder visualizarla en la página.

FUNCIÓN

```js
 // BUSCAR IMAGEN
buscarImagen(event){
    console.log(event.target.files[0]);
    const tipoArchivo = event.target.files[0].type;
    if(tipoArchivo === 'image/jpeg' || tipoArchivo === 'image/png'){
        this.file = event.target.files[0]
        this.error = null
    }
        else{
        this.error = 'Archivo no válido'
        this.file = null
        return 
        }
        const reader = new FileReader();
        reader.readAsDataURL(this.file);
        reader.onload = (e) => {
        this.datoImagen = e.target.result
        }
},
```

## Botón Editar / obtenerDatoID

### Template botón editar

Con obtener dato por id podemos rellenar los campos del formulario para editar y actualizar los datos.

Añadimos a la tabla la columna **Editar** después de **Correo**

```html
<th scope="col">Editar</th>
```

Añadimos en la fila el **botón Editar** antes del botón **Eliminar**

```html
<td>
    <button @click.prevent="obtenerDatoID( item.id );this.editar = !this.editar;" 
    class="btn btn-primary">Editar
    </button>
</td>
```
### Script Botón Editar

DATA

```js
data() {
    return {
      editar: false,
    }
}
```

MÉTODO

Importa **getDoc**

Método obtenerDatoID [getDoc](https://firebase.google.com/docs/firestore/query-data/get-data)

```js
import { doc, getDoc } from "firebase/firestore";
```

```js
// GET BY ID / OBTENER POR ID
async obtenerDatoID (id){
  const docRef = doc(db, "usuarios", id);
  const docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        this.usuario = docSnap.data()
        this.usuario.id = docSnap.id
        } 
        else {
        console.log("¡No existe el documento!");
        }
},
```

## Storage / subir imagen y guardar el enlace

- Subir la imagen a **storage** y guardar la dirección/enlace en una variable.


### Template botón Guardar / Editar

Añadir botón Actualizar y mostrar / ocultar los botones **editar: false**

```html
<!-- Mostrar/Ocultar Botones Guardar-Editar -->
<div class="mt-3">  
  <button v-show="this.editar === true" 
    @click.prevent="actualizarDato(id)" 
    class="btn btn-primary">
    Actualizar
  </button>
  <button v-show="this.editar === false" 
    @click.prevent="agregarDato()" 
    class="btn btn-primary">
    Guardar
  </button>
```

### Importar Storage en main.js

```js
import firebase from 'firebase/compat/app';
import 'firebase/compat/storage';

// Initialize Firebase
firebase.initializeApp(firebaseConfig)
var storageRef = firebase.storage().ref();

export { db, storageRef };
```

### Script / subirImagenDatos / actualizarImagenDatos

MÉTODO subirImagenDatos

Importa Storage

```js
import { updateDoc } from 'firebase/firestore/lite';
import { db, storageRef} from "../main";

```


```js
// SUBIR IMAGEN STORAGE
async agregarDato(){
  try {
    await storageRef.child('imagenes').child(this.file.name).put(this.file)
    const urlDescarga = await storageRef.child('imagenes').child(this.file.name).getDownloadURL()
    await addDoc(collection(db, "usuarios"), {
        nombre: this.usuario.nombre,
        correo: this.usuario.correo,
        foto: urlDescarga
      })
      this.error = 'Imagen subida con éxito'
      this.file = null
  } 
    catch (error) {
      console.log(error);
    } 
    finally {
      router.push('/')
    }
},
```
MÉTODO actualizarImagenDatos

```js
// MÉTODO actualizarDato
async actualizarDato(){
  try {
    await storageRef.child('imagenes').child(this.file.name).put(this.file)
    const urlDescarga = await storageRef.child('imagenes').child(this.file.name).getDownloadURL()
    const elemento = doc(db, "usuarios", this.usuario.id )
    await updateDoc(elemento, {
        nombre: this.usuario.nombre,
        correo: this.usuario.correo,
        foto: urlDescarga
      })
      this.error = 'Imagen subida con éxito'
      this.file = null
  } 
    catch (error) {
      console.log(error);
    } 
    finally {
      router.push('/')
    }
}
``` 
