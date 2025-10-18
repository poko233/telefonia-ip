📋 Guía de Pruebas - Sistema de Recordatorios *222#
📌 Información General
Número de prueba: 4232000
Contraseña de prueba: 12345 (5 dígitos)

🚀 PASO 1: Cargar Datos de Prueba en la Base de Datos
Ejecuta estos comandos para cargar 3 recordatorios de ejemplo:
Recordatorio 1: FUTURO - Cumpleaños (REPETIDO)
bashasterisk -rx "database put reminders 4232000/1/date 20251025"
asterisk -rx "database put reminders 4232000/1/time 0930"
asterisk -rx "database put reminders 4232000/1/type cumpleanos"
asterisk -rx "database put reminders 4232000/1/repeats 3"
asterisk -rx "database put reminders 4232000/1/state scheduled"

Fecha: 25 de octubre de 2025
Hora: 09:30 (9 horas 30 minutos)
Tipo: cumpleaños
Repeticiones: 3 veces ✅ (aparecerá en Opción 5)
Estado: programado


Recordatorio 2: FUTURO - Reunión (SIN REPETIR)
bashasterisk -rx "database put reminders 4232000/2/date 20251028"
asterisk -rx "database put reminders 4232000/2/time 1415"
asterisk -rx "database put reminders 4232000/2/type reunion"
asterisk -rx "database put reminders 4232000/2/repeats 1"
asterisk -rx "database put reminders 4232000/2/state scheduled"

Fecha: 28 de octubre de 2025
Hora: 14:15 (14 horas 15 minutos)
Tipo: reunión
Repeticiones: 1 vez (NO aparecerá en Opción 5)
Estado: programado


Recordatorio 3: PASADO - Trabajo (REPETIDO)
bashasterisk -rx "database put reminders 4232000/3/date 20241001"
asterisk -rx "database put reminders 4232000/3/time 0800"
asterisk -rx "database put reminders 4232000/3/type trabajo"
asterisk -rx "database put reminders 4232000/3/repeats 2"
asterisk -rx "database put reminders 4232000/3/state scheduled"

Fecha: 1 de octubre de 2024 ⏰ (PASADO - se borrará automáticamente)
Hora: 08:00 (8 horas 0 minutos)
Tipo: trabajo
Repeticiones: 2 veces
Estado: programado

Verificar que se cargaron correctamente:
bashasterisk -rx "database show reminders/4232000"
Debería salir algo como:
reminders/4232000/1/date      : 20251025
reminders/4232000/1/time      : 0930
reminders/4232000/1/type      : cumpleanos
reminders/4232000/1/repeats   : 3
reminders/4232000/1/state     : scheduled

reminders/4232000/2/date      : 20251028
reminders/4232000/2/time      : 1415
reminders/4232000/2/type      : reunion
reminders/4232000/2/repeats   : 1
reminders/4232000/2/state     : scheduled

reminders/4232000/3/date      : 20241001
reminders/4232000/3/time      : 0800
reminders/4232000/3/type      : trabajo
reminders/4232000/3/repeats   : 2
reminders/4232000/3/state     : scheduled

PASO 2: Probar marcando 1, 4 o 5
