# Bitácora y relevo — INC-03

## Bitácora de trabajo

| Orden | Actividad | Resultado o decisión | Estado / pendiente |
|---|---|---|---|
| 1 | Revisión inicial con `df -h`, `fdisk -l /dev/sda`, `df -i`, `mount` y `blkid`. | `/dev/sda3` era ext4 en `/`; tenía 2.6G totales y 2.4G libres. La VM veía disco de 10 GB, mientras la partición raíz medía 2771 MB. Inodos disponibles; raíz montada en `rw`. | Se identificó capacidad del disco sin asignar a la raíz. |
| 2 | Ampliación del disco virtual en Proxmox, según el informe del equipo. | Se informó cambio de 4 GB a 10 GB (+6 GiB). | La VM ya veía el disco de 10 GB al ejecutar las comprobaciones de particiones. |
| 3 | Ampliación de `/dev/sda3` con `parted` y de ext4 con `resize2fs`, según el registro del equipo. | `fdisk` mostró 8915 MB para sda3; `df -hT` mostró 8.5G para `/` y 8.0G disponibles. `dmesg` registró el redimensionamiento. | Capacidad recuperada. |
| 4 | Verificación del servicio y de la raíz web. | `rc-status` mostró nginx como `started`; configuración nginx indicó `/var/lib/nginx/html`. | Servicio activo durante las comprobaciones. |
| 5 | Inspección de permisos y prueba temporal de escritura. | Directorio `root:root`, modo 755; creación/listado/eliminación de archivo temporal pasó en sesión root. | Falta probar con el usuario real de desarrollo y reproducir el error original. |

## Transferencia entre turnos

- **Estado actual:** ampliación de capacidad completada; sugerido mantener INC-03 abierto hasta validar el flujo de guardado del desarrollador.
- **Hechos confirmados:** disco virtual visible de 10 GB; `/dev/sda3` de 8915 MB; ext4 en `/` de 8.5G con 8.0G disponibles; nginx activo; escritura temporal comprobada como root.
- **Causa confirmada:** la partición raíz y el sistema de archivos eran menores que el disco virtual y no usaban el espacio disponible.
- **Causa aún no confirmada:** los errores originales de escritura, dado que antes del cambio aún quedaban 2.4G libres y la prueba se hizo como root.
- **Precaución:** no cambiar permisos de forma recursiva ni dar escritura general (`777`). Confirmar primero usuario y ruta del despliegue; tomar respaldo antes de cualquier nuevo cambio de disco.
- **Próximo paso:** reproducir el guardado como el usuario de desarrollo y registrar el error exacto; confirmar si los 10 GB se refieren al disco virtual completo o al espacio utilizable en `/`.

## Dificultades y decisiones

| Dificultad o decisión | Contexto y evidencia | Resolución o motivo | Pendiente |
|---|---|---|---|
| `lsblk` no estaba disponible. | La VM devolvió `lsblk: not found`. | Se usó `fdisk -l /dev/sda` para consultar el disco y las particiones. | Ninguno para el dimensionamiento ya comprobado. |
| Prueba de escritura limitada a root. | El archivo temporal quedó como `root:root`; permisos de la carpeta 755. | Se documentó exactamente el alcance de la prueba, sin inferir que el usuario de desarrollo pueda escribir. | Probar bajo la cuenta real del despliegue. |
