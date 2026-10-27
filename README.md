# Crash of the Titans (PSP) — port nativo a PC, sin emulador

Primera publicación mía en GitHub: pidan disculpas por el desorden y por no saber todavía cómo organizar bien las cosas acá (etiquetas, estructura, formato de las capturas). Si alguien sabe cómo debería armarse un hilo así, me avisa y lo ordeno.

## Qué estoy haciendo

Un **port nativo a PC** de *Crash of the Titans* (PSP, ULES00915). No es un emulador y tampoco es un juego nuevo hecho desde cero: es el código del juego original corriendo en mi máquina sin capa de emulación.

## Método

Lo que hay hasta ahora, en orden:

1. **Recompilación anticipada (AOT) del binario original.** Tomo el `BOOT.BIN` de MIPS de PSP y lo traduzco a C++ (unos 90 objetos compilados por separado). No reescribo la lógica del juego a mano: el comportamiento viene del binario de Vivendi/Vicarious, decompilado y vuelto a compilar para x86-64.
2. **Reimplementación de los servicios de PSP que el binario llama**, en el host nativo: kernel (hilos, semáforos, colas de eventos, flags), `sceDisplay` con su latch de framebuffer, y las listas de comandos del GE (el GPU de PSP).
3. **Rasterizador por software propio** que emula el pipeline del GE (blend, texturas, scissor) sobre un framebuffer en memoria, y presentación en ventana con OpenGL/GDI.
4. **Assets originales** de la ISO, sin reemplazar ni regenerar nada.

La vara que me puse es dura a propósito: en modo fiel, el cuadro del título tiene que dar el **mismo hash** que el de la implementación de referencia (PPSSPP con la misma ISO). Si un cambio mueve un píxel, se descarta.

## Estado actual (honesto)

Ya dibuja imagen real del juego y ya pasó del título al menú, pero la cadencia todavía es de **~1 cuadro por segundo** en esta máquina. Es decir: el camino gráfico está abierto, la velocidad no.

Las capturas son del port, volcadas desde el framebuffer nativo a **480x272** (resolución interna real de PSP); las agrandé a 960x544 con vecino más cercano para que se lean, sin suavizar ni inventar detalle.

![Título con PULSA EL BOTÓN START](capturas/01_titulo_start.png)

![Menú principal: Nueva Partida / Cargar Partida / Multijugador / Créditos](capturas/02_menu_principal.png)

![Primer cuadro rasterizado (fondo del título)](capturas/03_primer_cuadro_raster.png)

## Objetivos

Lo que quiero llegar a conseguir, en este orden:

- **Rendimiento primero en CPU**, porque la meta es que se mueva bien en hardware humilde: 60 fps sostenidos con escalado 2x (960x544) y después 3x. Hoy mido unos 9,4 Mpx/s de relleno y 2x@60 pide ~31 Mpx/s, así que el margen está en el rasterizador y en la cadencia de presentación, no en la suerte.
- **Escalera de fps reales, sin trampa**: 60 → 120 → 240 como opciones. Nada de frame-skip ni de "turbo" (acelerar el reloj del juego como pasa en PSP): la métrica de control es que el reloj del juego avance 1,000 s por cada segundo real, y que los cuadros presentados sean cuadros distintos.
- **Escalera de calidad hasta 4K** (720p, 1080p, 2K, 4K) como **opción dejada lista** para cuando tenga una gráfica más potente, no para esta máquina.
- **Entrada nativa**: mando de Xbox, teclado y mouse los tres, sin mapeos roñosos.
- **Cámara libre** con mouse o con el stick derecho, en opción. Esto la PSP no lo tenía; lo quiero como plus opcional, apagado por defecto para no romper la fidelidad.
- Y el paso que falta antes de todo lo vistoso: **que se pueda jugar un nivel entero**.

## Hardware en uso

| | |
|---|---|
| CPU | AMD Ryzen 5 5500 (6 núcleos / 12 hilos) |
| RAM | 16 GB |
| GPU | NVIDIA GeForce GT 440 1 GB |

La GT 440 no tiene CUDA ni soporte moderno, así que todo el trabajo pesado hoy está del lado de la CPU. La referencia que quiero superar es PPSSPP en esta misma máquina: 50-60 fps estables a 2x.

## Próximos días

Voy a ir subiendo más información: la autopsia del sobre-dibujo (por qué se rellenan ~72 pantallas por cuadro), el histograma de áreas de primitivas, la cadencia de listas del GE por cuadro presentado, y las medicaciones de fps con la métrica anti-aceleración. También más capturas cuando haya gameplay.

Preguntas, correcciones y golpes de realidad, bienvenidos. Si alguien ya hizo un port AOT de un juego de PSP (o conoce material sobre el GE y su pipeline de blend), me interesa muchísimo leerlo.

