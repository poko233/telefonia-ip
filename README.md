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

# 3️⃣ Probar el menú interactivo

## Marca \*222# desde un teléfono interno.

## 

## Flujo:

## 

## Escuchas el menú principal.

## 

## Presiona 2 para “crear recordatorio”.

## 

## Ingresa la fecha, hora y graba el mensaje.

## 

## Espera siempre el pitido (beep) antes de marcar, excepto en el primer audio.

# 

# 4️⃣ Resultado esperado

## Se crea un archivo .call en /var/spool/asterisk/outgoing/.

## 

## A la hora programada, Asterisk originará una llamada que reproducirá tu audio grabado.

# 

# 📝 Tip:

## Solo puedes marcar opciones sin esperar el audio completo en el primer menú.

## En todos los demás, debes esperar el pitido antes de ingresar tu respuesta.

