# Tarea de Programación – Laboratorio 1

> Desarrollo del programa `psinfo` con uso reflexivo de herramientas de IA generativa

## 1. Introducción

En una empresa de desarrollo de software de sistema se está realizando un proyecto de personalización de un sistema operativo Linux (Ubuntu). Se ha propuesto incluir nuevos comandos en el intérprete de línea de comandos. A usted se le asigna la tarea de programar un nuevo comando llamado `psinfo`, que permita obtener información importante acerca de un determinado proceso del sistema operativo.

En esta versión del laboratorio, se autoriza el uso de herramientas generativas de código (por ejemplo ChatGPT, GitHub Copilot, etc.), con la condición de que cada estudiante documente claramente:
1. Las partes del código que fueron generadas por la herramienta.
2. La forma como se ajustaron o corrigieron las respuestas generadas.
3. El aprendizaje o hallazgos que surgieron al usar estas herramientas.
   
El objetivo es aprender tanto el uso de C y de los conceptos de procesos en Linux, como la correcta integración y validación de lo que producen las herramientas de IA.


## 2. Contextualización: Información de procesos en Linux

### 2.1. Sobre los procesos

Como se ha mencionado en la teoría, un proceso en Unix/Linux es una instancia en ejecución de un programa, con datos asociados (archivos abiertos, memoria asignada, prioridad, estado, etc.). En Ubuntu, se pueden obtener datos de los procesos en ejecución mediante:
* El System Monitor (interfaz gráfica).
* La consola, utilizando herramientas como `ps`, `top` o `jobs`.
  
### 2.2. Procesos y consola

Los comandos más utilizados para listar y monitorear procesos desde la consola incluyen:
* **`ps`**: Lista procesos en ejecución.
  * **`ps -ux`**: lista los procesos del usuario actual.
  * **`ps -aux`**: lista todos los procesos del sistema.
* **top**: Muestra procesos más activos, ordenados por uso de CPU.
* **jobs**: Muestra procesos en background en el shell actual.

### 2.3. Practicando los comandos anteriores

1. Ejecute `ps` y analice la salida.
2. Ejecute `jobs`. ¿Qué muestra?
3. Lance un proceso en background (por ejemplo `gedit &`) y observe su `PID`.
4. Ejecute `ps l` y revise campos como `PID`, `VSZ`, `RSS`, `STAT`.
5. Compare la información con la que brinda el System Monitor.
6. Cierre `gedit` y observe cómo se ve en `ps` y en la terminal.

### 2.4. Sobre el sistema de archivos `procfs`

En la carpeta `/proc`, el kernel expone información de los procesos. Cada proceso se representa como un subdirectorio /proc/[pid]/ con archivos como status, stat, cmdline, etc.

#### Actividad

* Explore `/proc`, verifique qué significan los subdirectorios numéricos (`PID`).
* Revise los archivos `status` y `stat` del proceso `gedit`. Compare la información y analice cuál es más práctico de leer para esta práctica.
  
## 3. Descripción del programa a desarrollar: `psinfo`

Deberá desarrollar un programa en C, con el objetivo de replicar (de forma simplificada) algunas funciones de comandos de monitoreo de procesos. Usará el pseudo-sistema de archivos `/proc` para obtener información.

El desarrollo se realizará por etapas, cada una generando una versión funcional. Al final, su programa debe:
* Recibir PIDs como parámetros.
* Leer la información de cada PID desde `/proc/[pid]/status` (u otros archivos en `/proc` que sean pertinentes).
* Mostrar la información requerida por pantalla y, según la etapa, también generar reportes a archivos.

### 3.1. Etapa 1

Desarrolle un programa básico `psinfo` que:
1. Reciba un solo identificador de proceso (`PID`) como argumento en línea de comandos.
2. Muestre en pantalla:
   * Nombre del proceso (`Name`).
   * Estado del proceso (`State`).
   * Tamaño total de la imagen de memoria.
   * Tamaño de la sección de memoria TEXT (`VmExe`).
   * Tamaño de la sección de memoria DATA (`VmData`).
   * Tamaño de la sección de memoria STACK (`VmStk`).
   * Número de cambios de contexto realizados:
     * Voluntarios (`voluntary_ctxt_switches`)
     * No voluntarios (`nonvoluntary_ctxt_switches`)
       
#### Ejemplo

A continuación se muestra un ejemplo de uso:

```bash
$ ./psinfo 10898

Nombre del proceso: gedit
Estado: S (sleeping)
Tamaño total de la imagen de memoria: 715000 KB
Tamaño de la memoria TEXT: 10000 KB
Tamaño de la memoria DATA: 24200 KB
Tamaño de la memoria STACK: 21000 KB
Número de cambios de contexto (voluntarios - no voluntarios): 17536 - 189
```

### 3.2. Etapa 2

Agregue la opción `-l` (lista múltiple). Si el usuario ingresa varios `PIDs`, por ejemplo:

```bash
$ psinfo -l 10898 1342 2341
```

El programa debera cumplir con los siguientes requisitos:
* Almacenar la información de cada PID en estructuras en memoria, antes de reportar el resultado en pantalla.
* Reportar en pantalla los datos de cada uno, por ejemplo:

A continuación se muestra una posible plantilla para la salida asociada al comando anterior:

```bash
-- Información recolectada!!!
Pid: 10898
Nombre del proceso: gedit
...
Pid: 1342
Nombre del proceso: chrome
...
Pid: 2341
Nombre del proceso: bash
...
```

### 3.3. Etapa 3

Implemente la opción `-r` (reporte a archivo). De forma similar a la opción `-l`, el programa debe:
1. Tomar varios PIDs.
2. Leer la información de cada proceso.
3. Generar un archivo de salida llamado: `psinfo-report-[pid1]-[pid2]-...[pidN].info` que contenga la misma información.
   
A continuación se muestra un ejemplo de uso para ilustrar el resultado de la implementación anteriormente descrita:

```bash
$ psinfo -r 10898 1342
Archivo de salida generado: psinfo-report-10898-1342.info
```

### 3.4. Etapa 4

El programa debe ser capaz de gestionar los siguientes casos de uso incorrecto:
* Parámetros inválidos o ausencia de PIDs.
* Procesos inexistentes.
* Error en la lectura de archivos de `/proc`.
  
Si detecta un error, el programa debe:
1. Imprimir un mensaje indicando el problema.
2. Mostrar un resumen de la sintaxis correcta (por ejemplo, `Uso: psinfo [-l|-r] <listado PIDs>`).
3. Finalizar con un código de error distinto del de ejecución normal (por ejemplo, `return 1;`).

## 4. Consideraciones generales

1. **Desarrollo por etapas**: Mantenga siempre una versión funcional (use control de versiones, como `git`).
2. **Buenas prácticas de programación**:
   * Código modular (multiarchivo).
   * Funciones claras.
   * Documentación apropiada (comentarios).
3. **Lectura y validación** de la información que genera `/proc/[pid]`:
   * Tenga en cuenta que algunos campos podrían no existir si el proceso ya terminó.
   * Sea robusto ante la posibilidad de que un archivo no se pueda abrir.
4. **Eficiencia**: Aunque no es el eje central de la práctica, procure un manejo adecuado de memoria (uso de *structs* y *punteros* cuando sea necesario).

## 5. Uso reflexivo de herramientas generativas

Para fomentar la comprensión y evitar un uso superficial de herramientas de IA, deberá incluir en la entrega lo siguiente:
1. **Reporte de uso de IA generativa**
   * Explique qué prompts o indicaciones usó con la(s) herramienta(s) (por ejemplo, ChatGPT, GitHub Copilot).
   * Mencione qué porciones de código provinieron de la IA.
   * Describa qué modificaciones realizó tras analizar y probar dicho código.
   * Incluya cualquier corrección de errores, advertencias o inconsistencias que haya surgido en el proceso.
2. **Reflexión técnica**
   * Describa por qué usó la IA en determinadas partes (por ejemplo, generación de una función de lectura de archivos, formateo de strings, etc.).
   * Explique qué aprendió al comparar el código generado con la documentación oficial o con sus propios conocimientos.
3. **Pruebas y debugging**
   * Documente al menos tres casos de prueba:
     * `PID` válido de un proceso en ejecución.
     * `PID` inexistente o muerto.
     * Uso de la opción `-l` o `-r` con múltiples `PIDs`.
   * Incluya capturas o transcripciones de la consola donde se vea el comportamiento de su programa.
   * Si surgieron errores (por ejemplo, segmentation fault, datos incorrectos), explique el proceso de debugging y la corrección realizada.
  4. **Sección "a mano"**
     * Al menos una de las secciones del código (por ejemplo, la que realiza la lectura detallada de `/proc/[pid]/status`) deberá ser desarrollada sin asistencia de la IA.
     * Explique en el reporte **cómo** comprobó y validó esa porción de código.

## 6. Recomendaciones de trabajo y ética

* Evite la dependencia ciega en la herramienta. Siempre revise la veracidad y pertinencia del código generado.
* Cite adecuadamente cualquier fragmento de ejemplo o referencia que use.
  
## 7. Entregables y evaluación

1. **Código fuente (multiarchivo) de `psinfo`**: Archivos de codigo fuente y `Makefile` (u otro mecanismo de compilación) que permita compilar el proyecto con un simple comando: `make`
2. **Informe escrito**: que debe incluir:
   * Estructura general del programa (funciones y archivos).
   * Reporte de uso de IA (prompts, porciones de código generadas, modificaciones realizadas).
   * Reflexión técnica (lecciones aprendidas, comparaciones, etc.).
   * Pruebas y debugging (casos de prueba, logs de corrección).
   * Conclusiones y posibles mejoras.
3. **Rúbrica de evaluación**:
   * **Funcionalidad (30%)**: El programa compila y corre adecuadamente, respondiendo a las etapas requeridas.
   * **Correctitud técnica (20%)**: Manejo correcto de archivos y estructuras en C, control de errores, lectura de `/proc`.
   * **Reporte de uso de IA (20%)**: Claridad en la explicación de cómo se usó la herramienta generativa. Diferenciación entre código generado y propio.
   * **Reflexión y debugging (20%)**: Calidad de la documentación de errores, pruebas y soluciones.
   * **Buenas prácticas y presentación (10%)**: Estructura modular, nomenclatura, comentarios, formato de entrega.

### 7.1. Formato final de entrega

* Un ZIP o carpeta con:
  * Código fuente (`*.c`, `*.h`).
  * `Makefile`.
  * Informe (`.pdf`).
* **Opcional**: Repositorio en `GitHub/Bitbucket` con historial de commits (si usa control de versiones).

### 7.2. Plazos y dudas

* La fecha límite de entrega y los canales de consulta (foros, Moodle, etc.) se encuentran en la plataforma oficial del curso.
* Cualquier duda adicional, contacte al equipo docente en horas de consulta o vía el foro del curso.

## 8. Nota final

Este laboratorio tiene como fin consolidar los conceptos de manejo de procesos en Linux, aprender y ejercitar la programación en C, y desarrollar competencias en la utilización crítica y efectiva de herramientas de IA generativa. ¡Éxitos en el desarrollo!

