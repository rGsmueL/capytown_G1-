# CapyTown G1 · RC-1 de control PID

Universidad ESAN · Robótica (08079) · 2026-2 · ROS 2 Humble · MicroROS-Pi5.

Los registros del simulador se recuperaron del proyecto. Los seis ensayos con nombre `*_real*`, sus cuatro bags, la cinta y el transportador son **datos sintéticos de demostración**. Las combinaciones de giro 2E y 2F también son sintéticas. La columna `origen` en la hoja de resultados indica la procedencia de cada fila. El nombre G1 corresponde a la carpeta solicitada y el dominio **20** consta en el README del RC-0.

## Integrantes

| Integrante | Rol | Responsabilidad en el RC-1 |
| --- | --- | --- |
| Maykol | Integración | Bringup, ROS_DOMAIN_ID, repositorio, tag, bags y hoja de resultados |
| Joaquin | Control | Sintonía de recta y yaw, ganancias por planta |
| Samuel | Planificación | Predicción, matriz de ensayos y procedimiento de medición |
| Wilmer | Percepción | Gráficas y comparación de /odom con la referencia externa |

## Entorno y ganancias finales

El dominio de trabajo del equipo indicado en RC-0 es `ROS_DOMAIN_ID=20`. En el simulador, `high` corresponde a la teleoperación A, `low` a las pruebas B–D y `medium` al estrés del cuadrado. La comprobación aislada del frenado usó `none`. En 2E y 2F, `low_modelado` identifica la aproximación numérica con ruido de odometría, sin el simulador ROS 2. El escenario de robot físico no tiene un parámetro `noise` de simulador.

| Perfil final del cuadrado | Línea Kp / Ki / Kd | Yaw Kp / Ki / Kd | v (m/s) | Límite angular (rad/s) | bias_w (rad/s) |
| --- | --- | --- | --- | --- | --- |
| Simulador, prueba 3opt | 3 / 0.3 / 2 | 10 / 0.1 / 1.2 | 0.20 | 2.0 | 0.05 |
| Robot físico sintético, prueba 3B | 3 / 0.3 / 2 | 5 / 0.1 / 1.2 | 0.16 | 1.0 | 0.00 |

[config/rc1_pid.yaml](config/rc1_pid.yaml) contiene los dos perfiles. Cada bloque contiene una configuración ROS 2 para `/cuadrado_controller` bajo `ros__parameters`. Para usar uno con `--params-file`, se extrae su contenido y se guarda desde la clave `/cuadrado_controller`.

## Transferencia simulador → escenario físico

| Prueba y ganancias comunes | Simulador | Robot físico sintético | Interpretación |
| --- | --- | --- | --- |
| Giro 2D, yaw 10 / 0.1 / 1.2 | Asentamiento 1,17 s, pico 0,64 %, ángulo externo 90,40° | 1,80 s, pico 1,06 %, ángulo externo 87,7° | El límite baja de 2 a 1 rad/s y el modelo añade inercia y retardo, por lo que cambia la respuesta con el mismo PID. |
| Recta 1D, línea 3 / 0.5 / 4 | Asentamiento 5,83 s, error final odom 0,46 cm | 7,17 s, error final odom 0,26 cm y externo 0,7 cm | Cambian la velocidad (0.20 a 0.16 m/s) y el origen del sesgo. |
| Cuadrado final | Prueba 3opt, cierre de 0,27 cm por /odom y 14,8 cm por /sim/pose | 3B, media de 18,2 cm de cinta sintética | El único cambio de ganancia tras el giro corto fue bajar Kp de yaw de 10 a 5; también se aplicaron otros límites y se retiró el sesgo inyectado. |

## Archivos

- [csv/hoja_resultados.csv](csv/hoja_resultados.csv) reúne las 30 pruebas. Las columnas `csv_prueba` y `csv_cierre` referencian las series del nodo y la cinta virtual, respectivamente. En los cuadrados el cierre es la distancia inicio–fin y el yaw acumulado sigue los cuatro giros. `cierre_real_cm` usa /sim/pose en el simulador y cinta sintética en el escenario físico. Para recta y giro se usan columnas de error específicas. Un asentamiento vacío indica que el error no permaneció en banda de ±2 cm o ±2° hasta el final del registro usado. Los ceros en `FIN` se excluyeron para la recta.
- [plots/](plots/) contiene los PNG de `09_plot_csv.py` (versión del curso que filtra `FIN` en recta, con origen en el título) y `cierre_real.py` sobre las series guardadas. La teleoperación A usa únicamente la serie pareada de `cierre_real.py`. La gráfica de cinta sintética compara distancias de cierre.
- [bags/](bags/) contiene cuatro carpetas ROS 2 Humble, SQLite3 y CDR. Sus tres tópicos son `/odom`, `/cmd_vel` y `/tf`, derivados a 30 Hz de los CSV sintéticos del cuadrado. La época de referencia es ficticia. Se inspeccionan con `ros2 bag info bags/rc1_real_3b_run1`.
- [sim_log.txt](sim_log.txt), [rqt_graph.png](rqt_graph.png) e [informe.pdf](informe.pdf) documentan el entorno, la evidencia gráfica y la conclusión en una página.

Tag local de la entrega completa: `v1.0-rc1`.
