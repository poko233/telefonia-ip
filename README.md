# 📞 Sistema de recordatorios por llamada — Asterisk

Una implementación para crear y reproducir recordatorios mediante llamadas salientes en Asterisk. El servicio se controla por accesos directos telefónicos con la tecla de marcación *222#.

Este README explica cómo desplegar el dialplan, subir audios, y —lo más importante— cómo usar los accesos directos `*222#` con ejemplos prácticos.

## 🧭 Resumen rápido

- Acceso principal: marcación *222#
- Funciones soportadas: listar, crear, modificar, borrar y listar recordatorios repetidos
- Audios: se guardan en `/var/lib/asterisk/sounds/custom/` como `reminder_<caller>_<id>.wav`
- Programación: se crean ficheros `.call` en `/var/spool/asterisk/outgoing/` con la mtime adecuada para programar la llamada

---

## Requisitos previos

- Asterisk instalado y funcionando
- Acceso SSH/SFTP al servidor Asterisk (para subir audios y el dialplan)
- Permisos para recargar el dialplan: `sudo asterisk -rx "dialplan reload"`

---

## Estructura relevante

- `extensions_custom.conf` — dialplan principal con todas las macros y handlers del servicio *222#.
- `sounds/` — carpeta recomendada para las grabaciones y audios del sistema.
- `/var/spool/asterisk/outgoing/` — ubicación donde se crean los ficheros `.call`.
- AstDB — base de datos embebida de Asterisk usada para guardar contraseñas y metadatos por número llamante.

---

## Despliegue / instalación (pasos recomendados)

1. Haz un backup del dialplan actual en el servidor Asterisk:

```sh
sudo cp /etc/asterisk/extensions_custom.conf /etc/asterisk/extensions_custom.conf.bak.$(date +%s)
```

2. Sube `extensions_custom.conf` desde este proyecto al servidor (WinSCP, scp o similar) y reemplaza el original.

3. Entra a la cli de isable:

```sh
"asterisk -vvvvvvvvvvvvvvvvvcr"
```

4. Recarga el dialplan:

```sh
"dialplan reload"
```

5. Sube los audios `.wav` a `/var/lib/asterisk/sounds/` y ajusta permisos/propietario si es necesario (`chown asterisk:asterisk ...`).

6. Prueba marcando `*222#` desde un interno.

---

## Cómo funciona (resumen)

1. Autenticación por número llamante (CALLERID):
	- Si no existe contraseña, el sistema pide crearla (normalmente 5 dígitos).
	- Si existe, el usuario debe introducirla. Se permiten hasta 3 intentos; al superar 3 fallos se marca el servicio como bloqueado para ese número en AstDB (`reminders/<caller>/blocked=1`).
	- El desbloqueo se realiza mediante `#222#` (ver sección "Desbloqueo y cambio de contraseña").

2. Menú de opciones (después de autenticación):
	1) Listar fechas especiales
	2) Crear recordatorio (interactivo)
	3) Modificar recordatorio (interactivo)
	4) Borrar recordatorio futuro
	5) Listar recordatorios repetidos (>1)

3. Scheduling y reproducción:
	- Las grabaciones se almacenan como `reminder_<caller>_<id>.wav` en `sounds/custom/`.
	- `schedule-reminder` crea un `.call` con la `mtime` ajustada a `EVENT - OFFSET` para que Asterisk origine la llamada en el momento adecuado.
	- En la reproducción se toca una introducción/tonada según el tipo de evento (cumpleaños, reunión, trabajo, cita), luego la grabación del usuario y al final se ofrece borrar o mantener el recordatorio.

---

## 📱 Guía de accesos directos para *222#

Formato general

```
*222#[CONTRASEÑA]*[OPCIÓN]*[PARÁMETROS]#
```

Si marcas solo `*222#[CONTRASEÑA]#` el sistema valida la contraseña y entra al menú interactivo.

---

### 1) Listar recordatorios (opción 1)

Formato

```
*222#[CONTRASEÑA]*1#
```

Ejemplo

```
*222#12345*1#
```

Resultado

Reproduce todos los recordatorios programados para el número que llama (ID, fecha, hora, tipo, repeticiones, estado).

---

### 2) Crear recordatorio (opción 2)

Formato

```
*222#[CONTRASEÑA]*2*[FECHA]*[HORA]*[TIPO]*[OFFSET]*[REPETICIONES]#
```

Parámetros

- FECHA: AAAAMMDD (ej. `20251225` = 25-12-2025)
- HORA: HHMM (ej. `1830` = 18:30)
- TIPO: `1`=Cumpleaños, `2`=Reunión, `3`=Trabajo, `4`=Cita
- OFFSET (anticipación): `1`=15 minutos, `2`=30 minutos, `3`=1 día (1440 minutos)
- REPETICIONES: `1` a `9` (número de veces que sonará)

Ejemplos

- Cumpleaños con recordatorio 15 minutos antes

```
*222#12345*2*20251225*1800*1*1*3#
```

- Reunión con recordatorio 30 minutos antes

```
*222#98765*2*20251015*0900*2*2*1#
```

- Cita médica con recordatorio 1 día antes

```
*222#55555*2*20251120*1430*4*3*2#
```

Comportamiento

El sistema pedirá que grabes el mensaje de voz, reproducirá la grabación para confirmación y programará el recordatorio.

---

### 3) Modificar recordatorio (opción 3)

Formato

```
*222#[CONTRASEÑA]*3*[ID]*[NUEVA_FECHA]*[NUEVA_HORA]*[TIPO]*[NUEVO_OFFSET]*[NUEVAS_REPETICIONES]#
```

Parámetros

- ID: identificador del recordatorio (obtenido al listar)
- NUEVA_FECHA: AAAAMMDD
- NUEVA_HORA: HHMM
- TIPO: `1`=Cumpleaños, `2`=Reunión, `3`=Trabajo, `4`=Cita
- NUEVO_OFFSET: 1, 2 o 3 (o valores literales 15, 30, 1440)
- NUEVAS_REPETICIONES: 1 a 9

Ejemplo

```
*222#12345*3*1*20251226*1900*1*2*4#
```

Comportamiento

El sistema actualiza el recordatorio y preguntará si quieres regrabar el mensaje de voz (presiona `1` para regrabar).

---

### 4) Borrar recordatorio (opción 4)

Formato

```
*222#[CONTRASEÑA]*4*[ID]#
```

Ejemplo

```
*222#12345*4*3#
```

Resultado

Elimina el recordatorio, su archivo de audio y los `.call` asociados.

---

### 5) Listar recordatorios repetidos (opción 5)

Formato

```
*222#[CONTRASEÑA]*5#
```

Ejemplo

```
*222#12345*5#
```

Resultado

Reproduce información de todos los recordatorios con `repeats > 1`.

---

### Acceso al menú principal (sin opción)

Formato

```
*222#[CONTRASEÑA]#
```

Ejemplo

```
*222#12345#
```

Resultado

Valida la contraseña y abre el menú interactivo donde puedes presionar:

- `1` = Listar fechas especiales
- `2` = Crear recordatorio (modo interactivo)
- `3` = Modificar recordatorio (modo interactivo)
- `4` = Borrar recordatorio (modo interactivo)
- `5` = Listar recordatorios repetidos

---

### Shortcut avanzado (bypass / contraseñas incrementales)

Formato conceptual:

```
*222#[CONTRASEÑA]#[ITERACIONES]#[NUMERO]#
```

- `[CONTRASEÑA]` = contraseña base
- `[ITERACIONES]` = número de iteraciones (usado en el flujo `bypass-password`)
- `[NUMERO]` = línea/identificador adicional

Ejemplo conceptual: `*222#123#2#3100001#` — este flujo está implementado en la macro `bypass-password` y solicita contraseñas calculadas (p. ej. `123+1`, `123+2`). 

---

## Menú interactivo: flujo detallado

1) Listar fechas especiales
	- Muestra los recordatorios futuros: ID, fecha, hora, tipo, repeticiones y estado. Las fechas pasadas se purgan automáticamente.

2) Crear recordatorio (interactivo)
	- Solicita: fecha (AAAAMMDD), hora (HHMM), tipo (1..4), grabación de voz, offset (15/30/1440), repeticiones (1..9).
	- Guarda metadata en AstDB y llama a `schedule-reminder`.

3) Modificar recordatorio (interactivo)
	- Pide ID y nuevos parámetros. Pregunta si se desea regrabar el audio.
	- Actualiza AstDB y recrea el `.call`.

4) Borrar recordatorio (interactivo)
	- Pide ID y confirmación. Borra metadatos, archivo de audio y `.call` si existe.

5) Listar recordatorios repetidos
	- Muestra solo aquellos con `repeats > 1`.

---

---

## Desbloqueo y cambio de contraseña

- Servicio de desbloqueo: marcar `#222#`.
- El sistema solicita la contraseña; si es correcta, se restablece `blocked=0` para ese número y ofrece cambiar la contraseña.

Nota: este flujo evita contar intentos contra el bloqueo ya existente.

---

## Comandos útiles (servidor)

1. Backup del dialplan actual:

```sh
sudo cp /etc/asterisk/extensions_custom.conf /etc/asterisk/extensions_custom.conf.bak.$(date +%s)
```

2. Ingresar a la CLI interactiva de Asterisk (opcional):

```sh
asterisk -vvvvvvvvvvvvvvvvvvvvvvvvvcr
```

3. Recargar dialplan:

```sh
"dialplan reload"
```


## Tabla resumen de formatos

| Acción | Formato | Ejemplo |
|---|---:|---|
| Menú principal | `*222#[PASS]#` | `*222#12345#` |
| Listar | `*222#[PASS]*1#` | `*222#12345*1#` |
| Crear | `*222#[PASS]*2*[FECHA]*[HORA]*[TIPO]*[OFFSET]*[REPS]#` | `*222#12345*2*20251225*1800*1*1*3#` |
| Modificar | `*222#[PASS]*3*[ID]*[FECHA]*[HORA]*[TIPO_RECORDATORIO]*[OFFSET]*[REPS]#` | `*222#12345*3*1*20251226*3*1900*2*4#` |
| Borrar | `*222#[PASS]*4*[ID]#` | `*222#12345*4*3#` |
| Listar repetidos | `*222#[PASS]*5#` | `*222#12345*5#` |
| Acceder al menu con contraseña | `*222#[CONTRASEÑA]#` | `*222#12345#` |
| Bypass contraseña | `*222#[CONTRASEÑA]#[ITERACIONES]#[NUMERO]#` | `*222#12345#4#2331#` |
| Desbloquear | `#222#` | `#222#` |
---
