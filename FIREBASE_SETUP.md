# Configuración de Firebase para CrowdHelp

## Paso 1: Crear proyecto en Firebase

1. Ve a https://console.firebase.google.com/
2. Haz clic en "Crear un proyecto"
3. Nombre del proyecto: `choco-crowd` (o el que prefieras)
4. Puedes desactivar Google Analytics si no lo necesitas
5. Haz clic en "Crear proyecto"

## Paso 2: Configurar Firestore Database

1. En el panel izquierdo, haz clic en "Build" → "Firestore Database"
2. Haz clic en "Crear base de datos"
3. Selecciona "Comenzar en modo de producción"
4. Selecciona la ubicación más cercana (ej: `us-central1`)
5. Haz clic en "Habilitar"

## Paso 3: Configurar reglas de seguridad

1. En Firestore, ve a la pestaña "Reglas"
2. Reemplaza las reglas con estas (permite lectura pública y escritura limitada):

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Permitir lectura pública de comentarios
    match /comments/{document=**} {
      allow read: if true;
      // Permitir escritura solo si incluye campos requeridos
      allow create: if request.resource.data.keys().hasAll(['name', 'message', 'amount', 'timestamp'])
        && request.resource.data.name is string
        && request.resource.data.name.size() > 0
        && request.resource.data.name.size() <= 100
        && request.resource.data.message is string
        && request.resource.data.message.size() > 0
        && request.resource.data.message.size() <= 500
        && request.resource.data.amount is number
        && request.resource.data.amount >= 0;
      // No permitir actualización ni eliminación
      allow update, delete: if false;
    }
  }
}
```

3. Haz clic en "Publicar"

## Paso 4: Obtener configuración de Firebase

1. Ve a "Configuración del proyecto" (ícono de engranaje)
2. En la pestaña "General", baja hasta "Tus aplicaciones"
3. Haz clic en el ícono de Web (</>)
4. Nombre de la app: `choco-crowd-web`
5. No necesitas Firebase Hosting
6. Haz clic en "Registrar app"
7. Copia la configuración que aparece:

```javascript
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

## Paso 5: Actualizar index.html

1. Abre el archivo `index.html`
2. Busca la sección `// --- Firebase Configuration ---`
3. Reemplaza los valores de `firebaseConfig` con los que copiaste
4. Guarda el archivo

## Paso 6: Crear índice en Firestore (Opcional pero recomendado)

1. En Firestore, ve a la pestaña "Índices"
2. Haz clic en "Crear índice"
3. Configura:
   - ID de colección: `comments`
   - Campos a indexar: `timestamp` (Descendente)
   - Ámbito de consulta: Colección
4. Haz clic en "Crear índice"

## Características implementadas

- ✅ Los comentarios se guardan en Firestore
- ✅ Los comentarios se sincronizan en tiempo real entre todos los usuarios
- ✅ Fallback a localStorage si Firebase falla
- ✅ Validación de datos en las reglas de seguridad
- ✅ Límite de caracteres en nombre (100) y mensaje (500)
- ✅ Los comentarios se ordenan por fecha (más recientes primero)

## Notas de seguridad

- Las reglas actuales permiten que cualquiera lea y escriba comentarios
- No se pueden editar ni eliminar comentarios una vez publicados
- Los campos están validados para evitar spam
- Para producción real, considera agregar:
  - Autenticación de usuarios
  - Rate limiting
  - Moderación de contenido
  - CAPTCHA para evitar bots