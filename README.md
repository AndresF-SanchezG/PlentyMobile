# PlentyMobile

## 1. El Corazón del Servidor (server.js y app.js)
El código inicia configurando un servidor web con Express.

Escucha Global: Se configura para escuchar en 0.0.0.0, lo que permite que el servidor reciba peticiones no solo de tu computadora, sino de internet o de una red local.

Seguridad CORS: Permite que aplicaciones externas (como un frontend en otro dominio) consuman la API.

Panel de Administración: El servidor sirve una página estática (admin.html) protegida por una contraseña básica (Basic Auth). Es aquí donde el administrador inicia el proceso de vinculación con QuickBooks.

## 2. Autenticación con Firebase (tokenGenerator.js)
Antes de que un usuario (o tú usando Insomnia/Postman) pueda hacer algo, necesita un token de Firebase.

Generación de Custom Token: Usa la llave privada de Firebase para crear un token de acceso basado en un UID de usuario específico.

Intercambio: Ese "Custom Token" se envía a las APIs de Google para convertirlo en un idToken real.

Resultado: Te imprime en la consola un código largo que debes usar en el encabezado Authorization: Bearer <TOKEN> para que el servidor confíe en tus peticiones.

## 3. Gestión de Tokens de QuickBooks (tokenService.js)
Esta es la parte más compleja y crítica. QuickBooks entrega tokens que vencen rápido (60 minutos).

Almacenamiento Seguro: Cuando el usuario se vincula, el saveTokens guarda el access_token y el refresh_token en Firebase Firestore, pero los encripta antes de guardarlos para que nadie pueda verlos si accede a la base de datos.

Auto-Refresco (Lógica Inteligente):

Cada vez que la app necesita datos, llama a getValidToken.

Si el token aún sirve (con un margen de seguridad de 50 minutos), lo desencripta y lo usa.

Si está por vencer, usa automáticamente el refresh_token para pedirle a QuickBooks uno nuevo, lo actualiza en la base de datos y sigue adelante sin que el usuario note nada.

## 4. Consulta de Inventario (qboService.js)
Una vez que el túnel de autenticación está listo, esta parte hace el trabajo real.

Instancia de QBO: Crea un objeto de conexión usando las credenciales (Client ID, Secret y el Token válido obtenido en el paso anterior).

Filtro de Productos: Ejecuta una consulta (findItems) filtrando específicamente por productos de tipo 'Inventory' (inventario físico).

Promesa: Devuelve la lista de productos para que puedan ser mostrados en la App o el Panel de Administración.
