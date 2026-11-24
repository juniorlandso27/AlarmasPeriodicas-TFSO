# Alarmas periódicas por tiempo de CPU en xv6-riscv

Trabajo final del curso de Sistemas Operativos.

## Descripción del proyecto

Este proyecto amplía xv6-riscv para que un proceso pueda registrar una función que se ejecute periódicamente según los ticks de CPU que acumula en modo usuario, al completar el intervalo configurado, el kernel guarda el contexto del proceso y ejecuta la función registrada. Cuando esta llama a `sigreturn()`, el proceso retoma la ejecución desde el punto donde fue interrumpido.

Escogimos este tema porque un programa que realiza un cálculo largo puede necesitar mostrar su avance mientras sigue trabajando, si el aviso depende del tiempo de CPU utilizado, los periodos en los que el proceso duerme o espera su turno quedan fuera del conteo, y la implementación nos permitió trabajar con interrupciones del temporizador, llamadas al sistema y restauración de registros en un mismo proyecto.

## Requisitos

- Linux o Debian/Ubuntu en WSL.
- Git, GNU Make y herramientas de compilación para el equipo anfitrión.
- Compilador y utilidades para RISC-V de 64 bits.
- QEMU con el emulador `qemu-system-riscv64`.
- Perl y `bc`, usados durante la generación de archivos y la compilación.

## Instalación

En Debian o Ubuntu, instala las dependencias:

```bash
sudo apt update
sudo apt install git build-essential gcc-riscv64-linux-gnu \
  binutils-riscv64-linux-gnu qemu-system-misc perl bc
```

Luego clona el repositorio:

```bash
git clone https://github.com/juniorlandso27/AlarmasPeriodicas-TFSO.git
cd AlarmasPeriodicas-TFSO
```

## Compilación y ejecución

Desde la carpeta del proyecto, en la terminal de Linux o WSL, compila e inicia xv6 con una CPU virtual:

```bash
make CPUS=1 qemu
```

Cuando aparezca el prompt de xv6, ejecuta estos comandos **dentro de QEMU**:

```text
alarma_demo
alarmas_test
```

- `alarma_demo` registra una alarma periódica y muestra `Alarmas recibidas: 3`.
- `alarmas_test` ejecuta las ocho pruebas y muestra el resultado de cada una.

Para salir de QEMU, pulsa **Ctrl+A** y luego **X**.

## Implementación

| Archivo | Cambio principal |
| --- | --- |
| `kernel/proc.h` y `kernel/proc.c` | Almacenan el intervalo, el contador, la dirección de la función y el contexto guardado de cada proceso. El hijo creado con `fork()` empieza con su temporizador desactivado. |
| `kernel/trap.c` | Cuenta los ticks en modo usuario y redirige la ejecución a la función registrada cuando se cumple el intervalo. Evita otra invocación mientras la función sigue activa. |
| `kernel/sysproc.c` | Implementa `sigalarm()` para configurar o desactivar la alarma y `sigreturn()` para restaurar el contexto. |
| `kernel/exec.c` | Descarta el estado de la alarma anterior cuando `exec()` reemplaza el programa. |
| `kernel/syscall.h`, `kernel/syscall.c`, `user/user.h` y `user/usys.pl` | Registran las llamadas al sistema y permiten invocarlas desde programas de usuario. |
| `Makefile`, `user/alarma_demo.c` y `user/alarmas_test.c` | Incorporan la demostración y el programa de pruebas a la imagen de xv6. |

## Pruebas realizadas

Las ocho pruebas se ejecutaron en QEMU con una CPU virtual; La salida final de `alarmas_test` fue `Pruebas aprobadas: 8/8`.

| Prueba | Qué se hizo | Resultado observado |
| --- | --- | --- |
| P1. Periodicidad | Configurar una alarma de 5 ticks y mantener una carga de CPU. | `OK`. Se recibieron al menos tres alarmas. |
| P2. Suspensión | Configurar un intervalo de 10 ticks y suspender el proceso durante 25 ticks. | `OK`. El aviso se recibió después de que el proceso volvió a usar la CPU. |
| P3. Restauración de `a0` | Cargar el valor `123` en `a0` y esperar una interrupción por alarma. | `OK`. `a0` conservó el valor `123` después de `sigreturn()`. |
| P4. Independencia con `fork()` | Configurar un intervalo en el padre, crear un hijo y registrar un intervalo distinto en el hijo. | `OK`. El hijo empezó con la alarma desactivada y, tras configurarla, ambos procesos recibieron sus propios avisos. |
| P5. Desactivación | Recibir una alarma, llamar a `sigalarm(0, 0)` y continuar usando la CPU. | `OK`. El número de alarmas se mantuvo fijo. |
| P6. Sin anidamiento | Mantener activa la función de alarma durante más tiempo que el intervalo configurado. | `OK`. La función terminó sin ser invocada nuevamente mientras seguía activa. |
| P7. Llamadas inválidas | Pasar un intervalo negativo a `sigalarm()` y llamar a `sigreturn()` fuera de una alarma. | `OK`. Ambas llamadas devolvieron `-1`. |
| P8. Reinicio con `exec()` | Ejecutar `exec()` desde la función de alarma para volver a cargar el programa. | `OK`. El programa nuevo no recibió avisos del temporizador anterior y `sigreturn()` devolvió `-1`. |

La prueba de periodicidad comprobó que los avisos se repiten. Queda pendiente registrar el contador del kernel en cada entrega para verificar el tick exacto.

## Evidencias

<img width="327" height="278" alt="image" src="https://github.com/user-attachments/assets/bc89b1c4-71dd-4ff7-b086-e895d6289e12" />
<img width="247" height="340" alt="image" src="https://github.com/user-attachments/assets/913e24ff-a62a-4987-926c-9f969e8f3526" />
<img width="229" height="134" alt="image" src="https://github.com/user-attachments/assets/54b7b0c4-66a0-4295-b6a1-cbc5c6f6b73a" />


## Comparación con Linux

En Linux, `setitimer()` con `ITIMER_VIRTUAL` usa el tiempo de CPU en modo usuario y genera la señal `SIGVTALRM` cuando vence. En este proyecto, xv6 usa ticks y dirige la ejecución a la función registrada mediante `sigalarm()`.

## Historial de aportes

La participación de cada integrante se puede consultar en la pestaña **Commits** del repositorio. Para revisar los autores y los cambios desde una copia local:

```bash
git log --format="%h %an - %s"
```

## Referencias

- [Repositorio original de xv6-riscv del MIT](https://github.com/mit-pdos/xv6-riscv)
- [Laboratorio Traps del MIT](https://pdos.csail.mit.edu/6.1810/2024/labs/traps.html)
- [Manual de Linux sobre setitimer()](https://man7.org/linux/man-pages/man2/setitimer.2.html)
## Publicación en LinkedIn
- [Ver publicación del proyecto en LinkedIn](https://www.linkedin.com/posts/jhon-elvis-curi-sanchez-062a1b451_sistemasoperativos-xv6-github-activity-7531005431633063937-T8g0)

