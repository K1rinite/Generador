# 📄 Generador Masivo de Inserts SQL (PDV Fybeca y SanaSana)

## 📌 Descripción del Proyecto
Esta es una herramienta web estática (Front-end puro en HTML, CSS/Tailwind y JavaScript) diseñada para automatizar la creación de sentencias SQL (`INSERT`) necesarias para registrar nuevos Puntos de Venta (PDV) y sus respectivos nodos (POS) en la base de datos corporativa. 

Reemplaza el proceso manual basado en plantillas de Excel, mitigando errores humanos en la asignación de IPs, cruces de identificadores entre cadenas comerciales, y optimizando el tiempo de despliegue.

## 🚀 Características Principales
* **Zero-Dependencies:** Funciona 100% en local desde el navegador. No requiere servidor back-end, Node.js, ni instalaciones de paquetes (ideal para entornos con restricciones de terminal).
* **Procesamiento por Lotes (Masivo):** Permite encolar múltiples PDVs en una misma sesión y consolidar todos los scripts en un único archivo `.txt` de salida.
* **Lógica de Red Dinámica:** Calcula automáticamente el último octeto de las IPs basándose en el segmento de red proporcionado y el número de POS.
* **Multicadena:** Adapta automáticamente los códigos internos (`LOCALID`, `NODEPROPERTIES`, códigos de cadena) dependiendo de si el local es Fybeca o SanaSana.

---

## ⚙️ Reglas de Negocio Implementadas (Lógica Interna)

### 1. Parámetros de Cadena
El sistema modifica los valores de inserción según la cadena seleccionada:
| Tabla | Campo Modificado | Fybeca | SanaSana |
| :--- | :--- | :--- | :--- |
| `GEOPOS.LOCALS` | Código de Cadena | `60` | `70` |
| `NODEPROPERTIES` | Origen de Copia (LOCALID) | `901` | `908` |
| `GEOPOS.NODES` | Código Nodo 99 | `9210` | `9211` |

### 2. Asignación de IPs por Tipo de Servidor
* **Físico:** El Nodo 99 asume automáticamente la terminación `.131` de la IP base. El POS 90 asume `.170`.
* **OCI (Oracle Cloud):** Habilita un input manual para la IP del Nodo 99, ya que esta se encuentra en un segmento de red completamente distinto al de los POS. El POS 90 asume `.132`.

### 3. Cálculo de Nodos (Cajas / POS)
La herramienta limpia la IP ingresada (extrayendo solo la base, ej: `10.8.74`) y calcula el último octeto así:
* **Nodo 1 al 9:** Terminación `.141` a `.149`.
* **Nodo 10 en adelante:** Fórmula `140 + N` (Ej: Nodo 10 = `.150`).
* **Nodos Especiales:** POS 81 (`.181`) y POS 89 (`.169`) disponibles mediante casillas de verificación.

---

## 🛠️ Estructura de las Tablas Afectadas
El archivo de salida generará scripts ordenados para poblar las siguientes tablas por cada local:
1. `GEOPOS.LOCALS`: Información general del PDV.
2. `GEOPROMOTION.ENGINENODES`: Registro para motor de promociones (concatena la IP del Nodo 99 en la descripción).
3. `USERLOCAL`: Asignación de usuario por defecto (`1`).
4. `NODEPROPERTIES`: Clonación de propiedades base según la cadena.
5. `GEOPOS.NODES`: Registro individual del servidor (Nodo 99) y todos los equipos POS.

---

## 📖 Instrucciones de Uso
1. Abrir el archivo `generador.html` con cualquier navegador web moderno (Chrome, Edge, Firefox).
2. Seleccionar la **Cadena** (Fybeca/SanaSana).
3. Ingresar los datos maestros: **ID**, **Nombre** y **Dirección**.
4. Pegar la **IP de un POS** (el sistema extraerá la base automáticamente).
5. Configurar el **Tipo de Servidor** y la **Cantidad de POS**.
6. Hacer clic en `+ Agregar PDV a la lista`.
7. Repetir el proceso si hay más locales.
8. Hacer clic en `Descargar Archivo` para obtener el archivo de texto final con las sentencias SQL listas para ejecutar.
