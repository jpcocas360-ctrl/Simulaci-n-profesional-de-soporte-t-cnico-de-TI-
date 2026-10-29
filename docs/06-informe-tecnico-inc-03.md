# Informe técnico — INC-03

## 1. Resumen ejecutivo

Se investigó una limitación de capacidad en un servidor web con Alpine Linux. Aunque el disco virtual ya era visible para la máquina como un disco de 10 GB, la partición raíz `/dev/sda3` ocupaba aproximadamente 2.7 GB y el sistema de archivos ext4 montado en `/` tenía 2.6 GB. La diferencia entre el tamaño del disco virtual y el tamaño de `/` confirmó que la capacidad disponible no se había asignado por completo a la partición y al sistema de archivos.

El equipo amplió `/dev/sda3` y ext4. Después del cambio, la partición mide 8915 MB; `df -hT` reporta 8.5 GB para `/`, con 8.0 GB disponibles. La consola del kernel confirmó el redimensionamiento de ext4. Nginx permaneció iniciado y se pudo crear y borrar un archivo temporal en la raíz web con privilegios de root.

**Resultado:** recuperación de capacidad completada. Los síntomas originales de error de escritura no se reprodujeron de forma concluyente; por ello se recomienda dejar el incidente pendiente de validación por el usuario de desarrollo.

## 2. Datos del incidente

- **Ticket:** INC-03
- **Incidente:** Capacidad insuficiente en servidor web
- **Categoría:** Infraestructura / almacenamiento
- **Prioridad:** Alta
- **Estado inicial:** Abierto
- **Servicio afectado:** Servidor web interno
- **Sistema:** Alpine Linux; sistema de archivos ext4
- **Servidor web:** Nginx
- **Solicitante:** Desarrollador web
- **Equipo:** Soporte técnico de TI

## 3. Síntomas y alcance solicitado

El solicitante reportó errores al guardar archivos y realizar operaciones del proyecto. La página web todavía respondía, pero no se podía garantizar que las operaciones de escritura funcionaran. El solicitante estimó una necesidad de al menos 10 GB de almacenamiento.

El objetivo fue diferenciar la capacidad del disco virtual de la capacidad asignada y disponible en el sistema de archivos, determinar la causa con comprobaciones verificables y aplicar una recuperación segura.

## 4. Diagnóstico

### Estado previo a ampliar la partición raíz

- `fdisk -l /dev/sda` mostró un disco de **10 GB** y `/dev/sda3` de **2771 MB**, con fin en el sector **8388607**; el disco llegaba al sector **20971519**.
- `df -h` mostró `/dev/sda3` montado en `/`, con **2.6 GB** totales, **62.8 MB** usados y **2.4 GB** disponibles (3%).
- `df -i` mostró **174977 inodos disponibles** en `/` (1% usado).
- `mount` identificó `/dev/sda3` como ext4 montado con opciones `rw`.

### Interpretación

La causa confirmada de la capacidad de trabajo reducida fue que la partición raíz y su sistema de archivos no ocupaban toda la capacidad del disco virtual. Los datos iniciales no indican que `/` estuviera lleno: quedaban 2.4 GB libres y los inodos también estaban disponibles. Por lo tanto, la falta de espacio o de inodos no quedó demostrada como causa directa de los errores originales de escritura.

## 5. Recuperación realizada

Según el registro del equipo y el informe visual, el disco virtual se amplió en Proxmox de **4 GB a 10 GB** (+6 GiB). Cuando se ejecutaron las comprobaciones de particiones, la VM ya veía el disco de 10 GB.

Luego se amplió `/dev/sda3` con `parted` y el sistema de archivos ext4 con `resize2fs`, según lo reportado por el equipo. El PDF conserva la salida de `parted` y `resize2fs`, pero el registro no transcribe la línea exacta de `parted`. La salida posterior confirmó:

- `/dev/sda3`: **8915 MB** en `fdisk -l /dev/sda`.
- `/`: **8.5 GB** totales y **8.0 GB** disponibles en `df -hT`.
- `dmesg`: mensajes de ext4 indican que el sistema de archivos se redimensionó de 709376 a 2282240 bloques.

El informe del equipo indica que los cambios se hicieron sin detener el servicio y sin pérdida de información. Nginx aparecía como `started` después del cambio; no se adjuntó una prueba HTTP sintética antes y después ni un registro continuo de disponibilidad que mida posibles interrupciones breves.

## 6. Verificación de escritura y permisos

La configuración de Nginx indicó `/var/lib/nginx/html` como raíz del sitio. `ls -ld` mostró propietario y grupo `root`, con permisos `755`. En la sesión con prompt `#`, se creó, listó y eliminó `prueba-inc03.tmp` sin error. No se registró una prueba de guardado iniciada por el desarrollador ni una validación de escritura del usuario de despliegue.

Esta prueba demuestra que la sesión con privilegios de root podía escribir en la raíz del sitio. No demuestra que la cuenta del desarrollador o el usuario empleado por el despliegue tenga permiso de escritura. Además, no se obtuvo el mensaje original ni se reprodujo el error bajo la cuenta afectada.

## 7. Estado final y causa

- **Capacidad de la VM:** 10 GB de disco virtual según `fdisk` tras la ampliación en Proxmox.
- **Capacidad de `/`:** 8.5 GB totales, 8.0 GB disponibles tras ampliar partición y ext4.
- **Nginx:** activo (`started`).
- **Escritura como root en la raíz web:** verificada y prueba eliminada.
- **Causa de la capacidad de trabajo reducida:** partición raíz y sistema de archivos más pequeños que el disco virtual; confirmada por `fdisk` y `df` antes del cambio.
- **Causa de los errores originales de escritura:** no confirmada. Se requiere repetir la operación con el usuario del desarrollador y registrar el error exacto.
- **Estado recomendado:** capacidad recuperada; incidente pendiente de validación funcional antes del cierre.

La asignación de todo el espacio restante del disco virtual a `/` no produce 10 GB utilizables en la raíz, porque también existen la partición de arranque, el área swap y la sobrecarga del sistema de archivos. El resultado observado es 8.5 GB para `/` y 8.0 GB disponibles. Si el requisito significa 10 GB utilizables dentro de `/`, el objetivo no se alcanzó: será necesario ampliar el disco virtual por encima de 10 GB, acordar el tamaño requerido y repetir la ampliación con un plan y respaldo aprobados. Si significa un disco virtual de 10 GB en total, se alcanzó el tamaño solicitado para el disco virtual, pero aún falta validar el guardado del usuario.

## 8. Próximos pasos

1. Pedir al desarrollador que repita la operación que fallaba y anote el usuario, la ruta y el mensaje exacto.
2. Si falla, comprobar los permisos y propietario de la ruta de destino para esa cuenta; aplicar el cambio mínimo autorizado y volver a probar.
3. Confirmar con el solicitante qué significa el requisito de 10 GB: tamaño total del disco virtual o capacidad utilizable en `/`.
4. Mantener el ticket abierto hasta la validación. Registrar la aceptación del solicitante, una prueba HTTP funcional del sitio, el resultado del guardado como usuario real y las salidas finales de `df -hT` y `fdisk -l /dev/sda`.

## 9. Referencias

- [Pruebas y evidencias](02-pruebas-y-evidencias.md)
- [Bitácora y relevo](03-bitacora-y-relevo.md)
- [Anexo aclaratorio del informe PDF](07-anexo-aclaratorio-inc-03.md)
- [Informe PDF con capturas](https://github.com/jpcocas360-ctrl/Simulaci-n-profesional-de-soporte-t-cnico-de-TI-/blob/main/SOP-INC-03%20(1).pdf)
