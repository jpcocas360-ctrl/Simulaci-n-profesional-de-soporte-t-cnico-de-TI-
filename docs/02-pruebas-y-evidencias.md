# Pruebas y evidencias — INC-03

Registrar las evidencias en orden cronológico. Las cifras de consola son las observadas en la VM Alpine; las capturas de Proxmox y el informe visual se encuentran en [SOP-INC-03 (1).pdf](https://github.com/jpcocas360-ctrl/Simulaci-n-profesional-de-soporte-t-cnico-de-TI-/blob/main/SOP-INC-03%20(1).pdf).

## Pruebas realizadas

| ID | Comando / comprobación | Resultado observado | Interpretación |
|---|---|---|---|
| P1 | `df -h` antes de ampliar la partición | `/dev/sda3` montado en `/`, tamaño 2.6G, usado 62.8M, disponible 2.4G (3%). `/dev/sda1` montado en `/boot`, 271.1M. | El sistema de archivos raíz no estaba lleno. |
| P2 | `fdisk -l /dev/sda` antes de ampliar la partición | Disco virtual visible de 10 GB. `/dev/sda3` medía 2771 MB; la partición terminaba en el sector 8388607, aunque el disco llegaba al sector 20971519. | La partición raíz no ocupaba todo el disco virtual; había capacidad sin asignar después de ella. |
| P3 | `df -i` | `/dev/sda3`: 177408 inodos totales, 2431 usados y 174977 disponibles (1%). | El agotamiento de inodos no explica los síntomas. |
| P4 | `mount` y `blkid` | `/dev/sda3` está montada como ext4 en `/` con `rw`; UUID y tipo ext4 identificados. | Sistema de archivos raíz identificable y montado para lectura/escritura. |
| P5 | `fdisk -l /dev/sda` después de ampliar la partición | `/dev/sda3` pasó a 8915 MB y termina en el último sector disponible del disco. | La partición raíz ocupa ahora el espacio disponible del disco virtual. |
| P6 | `df -hT` después del redimensionamiento | `/dev/sda3`, ext4: 8.5G total, 64.2M usados, 8.0G disponibles, 1%. | El sistema de archivos ext4 reconoció la ampliación. |
| P7 | `dmesg | tail -n 30` | El kernel informó `resizing filesystem` y `resized filesystem` para `sda3`. | Confirmación del redimensionamiento de ext4 por el kernel. |
| P8 | `rc-status` | Servicio `nginx` aparece como `started`. | Nginx estaba activo durante la revisión. |
| P9 | `grep -R "^[[:space:]]*root " /etc/nginx` | La raíz web configurada es `/var/lib/nginx/html`. | Ruta usada para revisar permisos y probar escritura. |
| P10 | `ls -ld /var/lib/nginx/html` | `drwxr-xr-x`, propietario `root`, grupo `root` (permisos 755). | Root puede escribir; otros usuarios solo tienen lectura y acceso al directorio. |
| P11 | Crear, listar y borrar `prueba-inc03.tmp` en la raíz web | `touch` y `ls -l` finalizaron sin error; el archivo temporal fue eliminado. La sesión mostraba prompt `#` y el archivo quedó como `root:root`. | Se comprobó escritura con privilegios de root. No demuestra que el usuario de desarrollo pueda desplegar ahí. |

## Cambios efectuados

| Cambio | Resultado | Precaución / reversión |
|---|---|---|
| Según el informe del equipo, el disco virtual se amplió en Proxmox de 4 GB a 10 GB (+6 GiB). | La consola de la VM detectó un disco de 10 GB. | El crecimiento del disco virtual y el de la partición/sistema de archivos son operaciones distintas. El registro no consigna si se tomó snapshot antes del cambio. |
| Se amplió `/dev/sda3` con `parted` y se redimensionó ext4 con `resize2fs`, según el procedimiento reportado por el equipo. | `/dev/sda3` quedó en 8915 MB y ext4 en 8.5G, con 8.0G disponibles. Nginx aparecía iniciado después del cambio. | No se conservó en el informe la línea de comandos exacta de `parted`; no afirmar ausencia de interrupción durante todo el cambio solo con el estado posterior del servicio. |

## Archivos de evidencia

| Archivo / captura | Qué demuestra |
|---|---|
| Figura 1 del PDF | Captura inicial de Proxmox: disco virtual de 4 GB. |
| Figuras 2–3 del PDF | Ampliación en Proxmox en 6 GiB y disco virtual resultante de 10 GB. |
| Figura 5 del PDF | `df -hT` luego de crecer el disco virtual, pero antes de extender `/dev/sda3`: `/` aún aparece con 2.6G. No usarla como evidencia del tamaño final. |
| Figura 6 (captura superior) del PDF | Secuencia de ampliación de la partición y ext4, con `df -h` final mostrando 8.5G y 8.0G libres. |
| Figuras 6 (captura inferior) y 8 del PDF | Salida final de `fdisk -l /dev/sda`; el PDF repite la etiqueta Figura 6 y luego salta a Figura 8. |
| Figura 9 del PDF | Propietario y permisos de `/var/lib/nginx/html`. |
| [Informe PDF completo](https://github.com/jpcocas360-ctrl/Simulaci-n-profesional-de-soporte-t-cnico-de-TI-/blob/main/SOP-INC-03%20(1).pdf) | Capturas y narrativa reunidas por el equipo. |

No adjuntar salidas que expongan contraseñas, tokens, direcciones privadas u otros datos sensibles. Las cifras y conclusiones principales están resumidas en [el informe técnico](06-informe-tecnico-inc-03.md). El [anexo de revisión](07-anexo-aclaratorio-inc-03.md) documenta los ajustes necesarios al informe PDF original.
