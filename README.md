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

- `extensions_custom.conf` — dialplan y macros para el servicio *222#
- `sounds/custom/` — ubicación prevista para los WAV de recordatorios y audios del sistema

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

1. El dialplan detecta accesos directos que empiezan por `*222#` y extrae los tokens que vienen después del primer `#`.
2. El primer token es la contraseña; el segundo token suele ser la opción (1..5). Los siguientes tokens se interpretan según la opción.
3. Si la contraseña es válida y la opción está presente, el handler procesa la operación correspondiente (listar, crear, modificar, borrar o listar repetidos).

En `extensions_custom.conf` hay macros/funciones como `create-reminder-directaccess`, `modify-reminder-directaccess`, `delete-reminder-directaccess`, `list-reminders`, `list-repeated-reminders` y `schedule-reminder`.

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
*222#[CONTRASEÑA]*3*[ID]*[NUEVA_FECHA]*[NUEVA_HORA]*[NUEVO_OFFSET]*[NUEVAS_REPETICIONES]#
```

Parámetros

- ID: identificador del recordatorio (obtenido al listar)
- NUEVA_FECHA: AAAAMMDD
- NUEVA_HORA: HHMM
- NUEVO_OFFSET: 1, 2 o 3 (o valores literales 15, 30, 1440)
- NUEVAS_REPETICIONES: 1 a 9

Ejemplo

```
*222#12345*3*1*20251226*1900*2*4#
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

## Tabla resumen de formatos

| Acción | Formato | Ejemplo |
|---|---:|---|
| Menú principal | `*222#[PASS]#` | `*222#12345#` |
| Listar | `*222#[PASS]*1#` | `*222#12345*1#` |
| Crear | `*222#[PASS]*2*[FECHA]*[HORA]*[TIPO]*[OFFSET]*[REPS]#` | `*222#12345*2*20251225*1800*1*1*3#` |
| Modificar | `*222#[PASS]*3*[ID]*[FECHA]*[HORA]*[TIPO_RECORDATORIO]*[OFFSET]*[REPS]#` | `*222#12345*3*1*20251226*3*1900*2*4#` |
| Borrar | `*222#[PASS]*4*[ID]#` | `*222#12345*4*3#` |
| Listar repetidos | `*222#[PASS]*5#` | `*222#12345*5#` |

---
