# 📞 Sistema de Recordatorios por Llamada — Asterisk

# 

## Este proyecto permite crear recordatorios por teléfono:  

## marcas `\*222#`, eliges una opción, ingresas fecha/hora y grabas un audio.  

## Asterisk programará automáticamente una llamada para reproducir ese mensaje.

# 

# 🚀 PASOS

# 1️⃣ Subir archivos `.wav` al servidor

## **Resultado esperado:** todos los `.wav` deben estar en `/var/lib/asterisk/sounds/`.

## 

## **Recomendado (WinSCP):**

## \- Conéctate al IP de tu máquina virtual (MV) con usuario y contraseña.

## \- Sube todos los audios `.wav` al directorio `/var/lib/asterisk/sounds/`.

# 

# 2️⃣ Subir el extensions\_custom.conf

# 

# Pasos:

## Copiar Código actual por si quieres un backup de lo que tienes en estos momentos

## &nbsp;	sudo cp /etc/asterisk/extensions\_custom.conf /etc/asterisk/extensions\_custom.conf.bak.$(date +%s)

## Luego reemplaza el contenido con el del archivo del proyecto (usando WinSCP o nano)

## &nbsp;	sudo asterisk -rx "dialplan reload"

# 

# 📞 Sistema de Recordatorios por Llamada (Asterisk)

Una implementación para crear y reproducir recordatorios vía llamadas salientes usando Asterisk y accesos directos telefónicos con la tecla marcación *222#.

Este README documenta cómo funciona el servicio, cómo desplegar el dialplan (`extensions_custom.conf`), y —lo más importante— una guía completa y clara de los accesos directos `*222#` con ejemplos prácticos.

## 📌 Resumen rápido
- Servicio principal: marcación *222#
- Soporta: listar, crear, modificar, borrar y listar recordatorios repetidos.
- Grabaciones: los audios se guardan en `/var/lib/asterisk/sounds/custom/` como `reminder_<caller>_<id>.wav`.
- Scheduling: se generan ficheros `.call` en `/var/spool/asterisk/outgoing/` con la mtime correcta para programar la llamada.

---

## Requisitos previos
- Asterisk instalado y funcionando.
- Acceso SSH/SFTP al servidor Asterisk (para subir audios y el dialplan).
- Permisos para recargar el dialplan: `asterisk -rx "dialplan reload"`.

---

## Estructura relevante del proyecto
- `extensions_custom.conf` — contiene todo el dialplan y macros para el servicio *222# (shortcuts, menús, creación y scheduling de reminders).
- `sounds/custom/` — ubicación prevista para los WAV de recordatorios y audios del sistema (en el servidor Asterisk).

---

## Despliegue / instalación (pasos recomendados)
1. Hacer backup del dialplan actual (en el servidor Asterisk):

```sh
sudo cp /etc/asterisk/extensions_custom.conf /etc/asterisk/extensions_custom.conf.bak.$(date +%s)
```

2. Subir el archivo `extensions_custom.conf` del proyecto al servidor (WinSCP, scp o similar) y reemplazar el original.

3. Recargar el dialplan:

```sh
sudo asterisk -rx "dialplan reload"
```

4. Subir audios `.wav` a `/var/lib/asterisk/sounds/custom/` y asegurarte de que permisos/propietario sean `asterisk:asterisk` (si el servidor lo requiere).

5. Probar desde un interno marcando `*222#`.

---

## Cómo funciona (visualmente)

1) El dialplan detecta accesos directos que empiecen con `*222#` y extrae los tokens que van después del primer `#`.
2) El primer token es la contraseña del usuario; el segundo puede ser la opción (1..5) y el resto parámetros según la opción.
3) Si la contraseña es correcta y la opción viene, el handler directo procesa la operación (listar, crear directo con parámetros, modificar, borrar o listar repetidos).

En `extensions_custom.conf` hay macros para cada acción: `create-reminder-directaccess`, `modify-reminder-directaccess`, `delete-reminder-directaccess`, `list-reminders`, `list-repeated-reminders` y `schedule-reminder`.

---

## 📱 Guía de Accesos Directos para *222#

Formato General

```
*222#[CONTRASEÑA]*[OPCIÓN]*[PARÁMETROS]#
```

Nota: si marcas solo `*222#[CONTRASEÑA]#` sin opción, el sistema validará y mostrará el menú interactivo.

---

### 1️⃣ LISTAR RECORDATORIOS (Opción 1)

Formato:

```
*222#[CONTRASEÑA]*1#
```

Ejemplo:

```
*222#12345*1#
```

Resultado: Reproduce todos los recordatorios programados para el número que llama (ID, fecha, hora, tipo, repeticiones, estado).

---

### 2️⃣ CREAR RECORDATORIO (Opción 2)

Formato:

```
*222#[CONTRASEÑA]*2*[FECHA]*[HORA]*[TIPO]*[OFFSET]*[REPETICIONES]#
```

Parámetros:
- FECHA: AAAAMMDD (ej: 20251225)
- HORA: HHMM (ej: 1830)
- TIPO: 1=Cumpleaños, 2=Reunión, 3=Trabajo, 4=Cita
- OFFSET (anticipación): 1=15 minutos, 2=30 minutos, 3=1 día (1440 minutos)
- REPETICIONES: 1 a 9 (número de veces que sonará el recordatorio)

Ejemplos prácticos:

- Cumpleaños con recordatorio 15 minutos antes:

```
*222#12345*2*20251225*1800*1*1*3#
```

- Reunión con recordatorio 30 minutos antes:

```
*222#98765*2*20251015*0900*2*2*1#
```

- Cita médica con recordatorio 1 día antes:

```
*222#55555*2*20251120*1430*4*3*2#
```

Resultado: El sistema pedirá que grabes el mensaje de voz, lo reproducirá para confirmar y programará el recordatorio.

---

### 3️⃣ MODIFICAR RECORDATORIO (Opción 3)

Formato:

```
*222#[CONTRASEÑA]*3*[ID]*[NUEVA_FECHA]*[NUEVA_HORA]*[NUEVO_OFFSET]*[NUEVAS_REPETICIONES]#
```

Parámetros:
- ID: identificador del recordatorio (obtenido al listar)
- NUEVA_FECHA: AAAAMMDD
- NUEVA_HORA: HHMM
- NUEVO_TIPO: 1-4 (mapeo al igual que crear)
- NUEVO_OFFSET: 1, 2, o 3 (mapeo igual que al crear) tambien se puede poner 15, 30 y 1440
- NUEVAS_REPETICIONES: 1 a 9

Ejemplo:

```
*222#12345*3*1*20251226*1900*2*4#
```

Resultado: El sistema actualizará el reminder y preguntará si quieres regrabar el mensaje de voz (presiona 1 para regrabar).

---

### 4️⃣ BORRAR RECORDATORIO (Opción 4)

Formato:

```
*222#[CONTRASEÑA]*4*[ID]#
```

Ejemplo:

```
*222#12345*4*3#
```

Resultado: Elimina el recordatorio, su archivo de audio y los `.call` asociados.

---

### 5️⃣ LISTAR RECORDATORIOS REPETIDOS (Opción 5)

Formato:

```
*222#[CONTRASEÑA]*5#
```

Ejemplo:

```
*222#12345*5#
```

Resultado: Reproduce información de todos los recordatorios con `repeats > 1`.

---

### 🎯 Acceso al Menú Principal (sin opción)

Formato:

```
*222#[CONTRASEÑA]#
```

Ejemplo:

```
*222#12345#
```

Resultado: Valida la contraseña y entra al menú interactivo (1=Listar, 2=Crear interactivo, 3=Modificar, 4=Borrar, 5=Listar repetidos).

---

## 📊 Tabla resumen de formatos

| Acción | Formato | Ejemplo |
|---|---|---|
| Menú principal | `*222#[PASS]#` | `*222#12345#` |
| Listar | `*222#[PASS]*1#` | `*222#12345*1#` |
| Crear | `*222#[PASS]*2*[FECHA]*[HORA]*[TIPO]*[OFFSET]*[REPS]#` | `*222#12345*2*20251225*1800*1*1*3#` |
| Modificar | `*222#[PASS]*3*[ID]*[FECHA]*[HORA]*[TIPO_RECORDATORIO]*[OFFSET]*[REPS]#` | `*222#12345*3*1*20251226*3*1900*2*4#` |
| Borrar | `*222#[PASS]*4*[ID]#` | `*222#12345*4*3#` |
| Listar repetidos | `*222#[PASS]*5#` | `*222#12345*5#` |

---
