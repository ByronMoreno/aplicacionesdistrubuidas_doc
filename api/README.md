# 🚀 ¿Qué es una API?

## 💡 Introducción

En el mundo actual, las aplicaciones **no funcionan de manera aislada**, sino que **intercambian información constantemente** entre sí.  

Por ejemplo:  
- Cuando usas **Facebook** para iniciar sesión en otra app, se está utilizando una API.  
- Cuando una app del clima muestra el pronóstico, obtiene esos datos de otra aplicación mediante una API.  

👉 **API** es el puente que permite esa comunicación entre aplicaciones.

---

## 🧩 Significado del término

**API** significa **Application Programming Interface**  
(Interfaz de Programación de Aplicaciones).

Una **interfaz** es un punto de contacto, una forma de comunicarse.  
Así como los humanos usamos un **lenguaje** para comunicarnos, las aplicaciones usan **APIs**.

---

## 📘 Definición formal

Una **API** es un **conjunto de reglas, métodos y protocolos** que permiten que dos sistemas de software se comuniquen entre sí, sin necesidad de conocer los detalles internos de cómo están construidos.

> En otras palabras: una API **define cómo un programa puede interactuar con otro**.

---

## 🍽️ Ejemplo simple

Imagina que estás en un restaurante:

- Tú eres el **cliente**.  
- La **cocina** es el **servidor**.  
- El **mesero** es la **API**.

Tú no entras a la cocina; simplemente le dices al mesero lo que quieres.  
El mesero lleva tu pedido, la cocina prepara la comida y luego el mesero te la entrega.  

➡️ La **API** cumple el papel del **mesero digital**, llevando solicitudes y trayendo respuestas entre sistemas.

---

## ⚙️ Cómo funciona una API

El proceso básico sigue este flujo:

1. **Solicitud (Request)**  
   Una aplicación cliente (por ejemplo, una app móvil) envía una petición a la API.  
   Ejemplo: *Quiero los datos del usuario con ID 10.*

2. **Procesamiento (Processing)**  
   El servidor recibe la solicitud, busca la información y la prepara.

3. **Respuesta (Response)**  
   La API devuelve la respuesta al cliente (por ejemplo, en formato JSON).  

   Ejemplo:
   ```json
   {
     "id": 10,
     "nombre": "Ana",
     "email": "ana@example.com"
   }
   ```

---

## 🌐 Tipos de APIs más comunes

| Tipo de API | Descripción | Ejemplo |
|--------------|--------------|----------|
| **REST** | Usa HTTP y devuelve datos normalmente en formato JSON. Es la más usada actualmente. | API de Twitter, GitHub |
| **SOAP** | Usa XML y es más formal y estructurada. | APIs empresariales o bancarias |
| **GraphQL** | Permite consultar solo los datos necesarios. | Facebook, GitHub GraphQL API |
| **WebSocket** | Permite comunicación en tiempo real. | Chats o juegos online |

---

## 🧱 Componentes de una API REST

1. **Endpoint:** la URL que identifica un recurso.  
   Ejemplo: `https://api.tienda.com/clientes/1`  
2. **Métodos HTTP:** definen la acción.  
   - `GET`: obtener información  
   - `POST`: crear un nuevo recurso  
   - `PUT`: actualizar un recurso  
   - `DELETE`: eliminar un recurso  
3. **Formato de datos:** JSON o XML.  
4. **Autenticación:** controla quién puede acceder (por ejemplo, con **JWT Tokens**).

---

## 💪 Ventajas de usar APIs

✅ **Interoperabilidad:** Permiten que diferentes sistemas se comuniquen.  
✅ **Reutilización:** Una API puede usarse en múltiples aplicaciones.  
✅ **Seguridad:** No se exponen los detalles internos del sistema.  
✅ **Escalabilidad:** Facilita construir aplicaciones distribuidas y modulares.  
✅ **Evolución tecnológica:** Permite integrar servicios externos (Google Maps, PayPal, etc.).

---

## 🌍 Ejemplo real

Una aplicación de delivery quiere mostrar un mapa con la ubicación del repartidor 🚴‍♂️.  
No necesita crear su propio mapa, simplemente **usa la API de Google Maps**:

```
https://maps.googleapis.com/maps/api/geocode/json?address=Quito
```

La API responde con coordenadas, calles y nombres de lugares que la app puede usar directamente.

---

## 🧠 Conclusión

Las **APIs son la base de las aplicaciones distribuidas modernas**.  
Permiten que múltiples servicios (por ejemplo: pagos, mapas, usuarios, productos) **trabajen juntos sin estar en el mismo servidor**.

> 💬 En resumen:
> **Una API es el puente que conecta aplicaciones, servicios y sistemas, permitiendo el intercambio de datos de manera estandarizada, segura y eficiente.**

---

📚 **Materia:** Aplicaciones Distribuidas  
👨‍🏫 **Tema:** Introducción a las APIs 
