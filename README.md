# ESCONDETE DEL SAT
### Un videojuego de supervivencia fiscal en primera persona

---

> *"El SAT no descansa. Tu tampoco puedes."*
> 
> Un juego de navegador en 3D donde interpretas a un contribuyente en apuros que debe sobrevivir el mayor tiempo posible evadiendo a los agentes del SAT, lavando dinero en efectivo, y usando la impunidad politica a su favor.

---

## Descripcion General

**Escondete del SAT** es un juego de supervivencia en primera persona desarrollado como una pagina HTML autocontenida. No requiere instalacion, servidor, ni dependencias externas. Funciona directamente en el navegador con tecnologia WebGL (Three.js) y logica reactiva (Vue 3). La estetica retro con fuentes pixeladas, scanlines y efectos neon en verde terminal rinden homenaje a los arcades de los 90.

El objetivo es acumular la mayor puntuacion posible antes de ser "auditado" por los agentes del SAT o quedar sin dinero.

---

## Tabla de Contenido

| Seccion | Descripcion |
|---|---|
| [Tecnologias](#tecnologias) | Stack tecnico utilizado |
| [Mecanicas de Juego](#mecanicas-de-juego) | Reglas y sistemas centrales |
| [Controles](#controles) | Teclado y raton |
| [HUD e Interfaz](#hud-e-interfaz) | Elementos visuales en pantalla |
| [Economía del Juego](#economia-del-juego) | Dinero, efectivo y lavado |
| [Enemigos](#enemigos) | Tipos de amenazas |
| [Armas](#armas) | Arsenal disponible |
| [Items y Mejoras](#items-y-mejoras) | Potenciadores en la tienda |
| [Ubicaciones del Mapa](#ubicaciones-del-mapa) | Puntos clave del mundo |
| [Sistema de Puntuacion](#sistema-de-puntuacion) | Como se calcula el score |
| [Condiciones de Derrota](#condiciones-de-derrota) | Causas del Game Over |
| [Configuracion](#configuracion) | Ajustes disponibles |
| [Arquitectura Tecnica](#arquitectura-tecnica) | Como esta construido |

---

## Tecnologias

| Libreria | Version | Uso |
|---|---|---|
| Three.js | r128 | Motor 3D / WebGL |
| Vue 3 | Global CDN | Interfaz reactiva (HUD, menus, tienda) |
| Web Audio API | Nativa | Efectos de sonido procedurales |
| Font Awesome | 6.4.0 | Iconografia de interfaz |
| Google Fonts | CDN | Tipografias retro (Press Start 2P, VT323, Oswald) |

El juego no utiliza ningun backend, base de datos, ni almacenamiento local. Todo el estado reside en memoria durante la sesion.

---

## Mecanicas de Juego

### Ciclo Principal

El bucle de juego opera a traves de `requestAnimationFrame` con delta time controlado (maximo 0.05 segundos por frame para evitar saltos de fisica). Cada iteracion ejecuta:

1. Movimiento del jugador y camara
2. Actualizacion de IA de enemigos
3. Deteccion de proximidad a objetos interactuables
4. Verificacion de zona IVA
5. Spawning dinamico de enemigos y efectivo
6. Cobro periodico de impuestos
7. Actualizacion del minimapa

### Fisica del Jugador

El jugador se mueve en un mundo de 120x120 unidades (±60 en X y Z). La fisica de gravedad es simple:

- Gravedad: `20 unidades/s²`
- Impulso de salto: `8 unidades/s` hacia arriba
- Colision con el suelo en `Y = 0`
- El jugador no puede salir de los limites del mundo

### Interaccion

Presionando `E` cuando el jugador esta cerca de un objeto interactuable se activa una accion contextual. La proximidad de deteccion varia por tipo de objeto.

---

## Controles

| Accion | Tecla Predeterminada |
|---|---|
| Mover adelante | W / Flecha Arriba |
| Mover atras | S / Flecha Abajo |
| Mover izquierda | A / Flecha Izquierda |
| Mover derecha | D / Flecha Derecha |
| Saltar | Espacio |
| Interactuar | E |
| Ataque cuerpo a cuerpo | F |
| Disparar arma equipada | Clic izquierdo |
| Seleccionar slot de arma | Teclas 1 a 5 |
| Pausar / Reanudar | Escape |

> El movimiento de camara requiere que el puntero del raton este bloqueado en el canvas. El juego solicita el bloqueo automaticamente al hacer clic sobre la pantalla de juego.

Los controles son reconfigurables desde el menu de Configuracion. La sensibilidad del raton se ajusta en una escala de 1 a 10 (valor por defecto: 5).

---

## HUD e Interfaz

La interfaz se superpone en tiempo real sobre el renderizado 3D y esta compuesta por los siguientes elementos:

| Elemento | Ubicacion | Descripcion |
|---|---|---|
| Barra de Tarjeta (dinero digital) | Arriba izquierda | Barra de color que refleja el saldo en cuenta. Verde = bien, Amarillo = precaucion, Rojo = critico |
| Contador de efectivo | Arriba derecha | Muestra el efectivo en inventario. Maximo: $50,000 MXN |
| Tiempo evadido | Centro superior | Temporizador en formato MM:SS |
| Puntuacion | Centro superior | Score acumulado |
| Inventario de armas | Barra inferior central | 5 slots de armas con icono, numero de slot y balas restantes |
| Minimapa | Abajo derecha | Canvas 130x130 px con vision de pajaro del entorno |
| Zona IVA | Pantalla completa | Overlay rojo semitransparente al entrar a la zona de lavado |
| Indicador de ocultamiento | Centro | Aparece cuando el jugador esta oculto en un edificio PRI |
| Notificaciones | Centro superior | Mensajes emergentes con colores segun tipo (info, exito, peligro, advertencia, curioso) |
| Numeros flotantes | Centro pantalla | Indicadores visuales de ganancias y perdidas de dinero |
| Mira | Centro exacto | Cruceta verde para apuntar |
| Hint de interaccion | Centro inferior | Indica que accion realizar al acercarse a un objeto |

---

## Economia del Juego

El juego maneja dos tipos de dinero con funciones completamente distintas:

### Dinero Digital (Tarjeta)

- Valor inicial: `$20,000 MXN`
- Valor maximo: `$20,000 MXN`
- Se pierde si el SAT te atrapa o por cobros periodicos de impuesto
- Si llega a cero: Game Over

### Efectivo (Inventario)

- Capacidad maxima: `$50,000 MXN`
- Se recoge del suelo interactuando con billetes dispersos en el mapa
- Cantidades disponibles al recoger: $100, $200, $500, $1,000, $2,000 o $5,000
- El efectivo solo sirve para comprar en la tienda o lavarlo en la lavanderia

### Lavado de Dinero

Al acercarte al edificio "Lavado de Dinero" e interactuar, el efectivo de tu inventario se convierte en saldo en tarjeta. Si permaneces mas de 5 segundos dentro de la zona sin interactuar, el IVA comenzara a cobrarte `$1,000 por segundo` directamente de tu tarjeta.

### Cobro Periodico de Impuestos

Cada 30 segundos de partida, el sistema cobra automaticamente `$200 MXN` del saldo en tarjeta. Este monto se reduce al 70% si tienes el Contador Corrupto activo.

---

## Enemigos

### Agente del SAT

El enemigo principal. Patrulla el mapa y persigue al jugador cuando lo detecta.

| Parametro | Valor |
|---|---|
| Vida | 3 puntos de daño |
| Velocidad base | Baja (patrulla) |
| Velocidad persecucion | Mayor que base |
| Rango de deteccion | Depende del estado |
| Daño al jugador | Reduce saldo de tarjeta |
| Estado al ser noqueado | Aturdido 10 segundos, luego regresa a patrulla |
| Cantidad inicial | 4 agentes |
| Cantidad maxima simultanea | 12 agentes |

**Estados de IA del SAT:**

| Estado | Comportamiento |
|---|---|
| `patrol` | Deambula aleatoriamente por el mapa |
| `chase` | Persigue al jugador en linea recta |
| `stun` | Inmovilizado temporalmente tras ser noqueado |

El jugador es invisible para los agentes SAT cuando esta oculto en un edificio PRI (inmunidad politica).

Cada 30 segundos (reduciendo hasta 10 segundos conforme avanza la partida), aparece un nuevo agente SAT con una notificacion de alerta.

### Asaltante

Enemigo secundario con comportamiento de deambulacion aleatoria. Puede robar dinero del inventario del jugador si se acerca demasiado.

| Parametro | Valor |
|---|---|
| Vida | 2 puntos de daño |
| Arma | Cuchillo (sin deduccion fiscal) |
| Cantidad maxima | 2 asaltantes simultaneos |
| Intervalo de spawn | Cada 45 segundos |
| Daño | Roba efectivo del inventario |

El Casco de Obrero reduce el daño de asaltantes un 50%.

---

## Armas

Las armas se adquieren en la tienda "Pirateria y Asociados" usando efectivo. Cada compra incluye IVA del 16%.

| Arma | Precio base | Daño | Municion inicial | Modo de disparo |
|---|---|---|---|---|
| Pistola de Agua | $5,000 | 1 HP | 10 balas | Un proyectil por clic |
| Escopeta Pirata | $9,000 | 3 HP | 6 cartuchos | 3 proyectiles con dispersion |
| TEC-9 Pirata | $14,000 | 1 HP | 30 balas | Rafaga de 3 balas por clic |
| Lanza-RFC | $20,000 | 9 HP | 3 cohetes | Un cohete por clic |

El sistema de disparo usa **raycasting desde la camara** con rango maximo de 50 unidades. La escopeta aplica dispersion adicional a los proyectiles secundarios. Al impactar, aparece una chispa 3D en el punto de contacto.

### Municion Recargable

| Recarga | Precio base | Cantidad |
|---|---|---|
| Balas Pistola | $1,200 | +15 balas |
| Cartuchos Escopeta | $1,800 | +6 cartuchos |
| Cargador TEC-9 | $2,500 | +30 balas |
| Cohete Extra | $6,000 | +1 cohete |

La municion solo puede comprarse si ya se posee el arma correspondiente.

---

## Items y Mejoras

Items de una sola compra que otorgan ventajas pasivas permanentes durante la partida:

| Item | Precio base | Efecto |
|---|---|---|
| Contador Corrupto | $12,000 | Reduce todos los impuestos cobrados en un 30% |
| Casco de Obrero | $3,000 | Reduce el daño de asaltantes en un 50% |
| Bicicleta Cleta | $6,000 | Aumenta la velocidad de movimiento en un 40% |
| Celular Espejo | $4,000 | Activa el radar SAT en el minimapa |
| Escudo Fiscal | $15,000 | Absorbe un golpe del SAT (uso unico) |

Todos los precios de la tienda tienen un IVA automatico del 16% sobre el valor base.

---

## Ubicaciones del Mapa

El mundo tiene dimensiones de 120x120 unidades. Las siguientes ubicaciones son interactuables:

| Lugar | Coordenadas (aprox.) | Funcion |
|---|---|---|
| Lavado de Dinero | (-18, -5) | Convierte efectivo en saldo de tarjeta. Riesgo de IVA si te quedas |
| Sede PRI Local | (15, -12) | Escondite con inmunidad politica contra el SAT |
| Sindicato Nacional | (-12, 18) | Segundo escondite con inmunidad politica |
| Pirateria y Asociados (Tienda) | (18, 15) | Compra armas, municion e items |

El resto del mapa contiene edificios decorativos, parches de cesped, bardas perimetrales y postes de alumbrado con luz dinamica.

---

## Sistema de Puntuacion

La puntuacion se incrementa al lavar dinero en la lavanderia:

```
Score += floor(monto_lavado / 100)
```

Ademas, el juego registra las siguientes estadisticas al terminar la partida:

| Estadistica | Descripcion |
|---|---|
| Tiempo sobrevivido | Tiempo total antes del Game Over |
| Puntuacion final | Score acumulado por lavado |
| Efectivo acumulado | Total de efectivo recogido del suelo |
| Agentes SAT evadidos | Cantidad de agentes noqueados |
| Veces robado | Cantidad de robos exitosos de asaltantes |
| IVA pagado total | Suma de todos los impuestos cobrados |

El mejor tiempo de la sesion se guarda en memoria y se muestra si se supera en la partida siguiente.

---

## Condiciones de Derrota

El juego termina ("AUDITADO") cuando se cumple alguna de las siguientes condiciones:

| Condicion | Mensaje |
|---|---|
| Saldo en tarjeta llega a cero por agente SAT | Mensaje especifico segun la logica de dano |
| Saldo en tarjeta llega a cero por IVA de lavanderia | "El IVA de la lavanderia te dejo sin dinero. Paradojas de la vida." |
| Saldo llega a cero por cobro periodico de impuestos | Game Over automatico |

---

## Configuracion

Accesible desde el menu principal y desde la pantalla de pausa:

| Ajuste | Tipo | Rango |
|---|---|---|
| Idioma | Selector | Espanol |
| Volumen de musica | Slider | 0 a 100 |
| Volumen de efectos | Slider | 0 a 100 |
| Brillo | Slider | 0 a 100 |
| Sensibilidad del raton | Slider | 1 a 10 |
| Controles | Reconfigurables | 9 acciones asignables |

---

## Arquitectura Tecnica

El juego esta contenido en un unico archivo `.html` con todo el codigo CSS, HTML y JavaScript embebido. No existen archivos separados ni proceso de compilacion.

### Patron de Arquitectura

```
HTML (estructura DOM)
 ├── CSS (estilos, animaciones, variables CSS)
 └── JavaScript (logica de juego)
      ├── Vue 3 Setup (estado reactivo, computed, lifecycle)
      │    ├── Estado del juego (state, playerMoney, survivalTime, score...)
      │    ├── Sistema de notificaciones y numeros flotantes
      │    ├── Logica de tienda y compras
      │    └── Handlers de eventos (teclado, raton, raton bloqueado)
      └── Three.js (motor 3D)
           ├── Escena, camara perspectiva, renderer WebGL
           ├── Texturas procedurales (canvas 2D → THREE.CanvasTexture)
           ├── Modelos geometricos de bajo poligonaje
           ├── Sistema de luces (ambiental, direccional, puntual, hemisferica)
           ├── IA de enemigos (maquina de estados: patrol / chase / stun)
           ├── Raycasting para disparo
           └── Minimapa (canvas 2D separado)
```

### Audio

Los efectos de sonido son completamente procedurales mediante la **Web Audio API**. No se carga ningun archivo de audio externo. Los sonidos se generan con osciladores de tipo `sine`, `square` y `sawtooth` con envolventes de amplitud.

| Evento | Frecuencia / Tipo |
|---|---|
| Recoger efectivo | 880Hz + 1100Hz, sine |
| Alerta SAT | 300Hz + 200Hz, square |
| Victoria (lavado) | Arpegio 440-550-660-880Hz, sine |
| Impacto | 150Hz, sawtooth |
| Disparo fallido | 180Hz, sine |
| Salto | 300Hz, sine |

### Texturas Procedurales

Todas las texturas son generadas en tiempo de ejecucion con Canvas 2D API:

| Textura | Descripcion |
|---|---|
| `ground` | Asfalto oscuro con grietas aleatorias |
| `wall` | Patron de ladrillos con variacion de color |
| `grass` | Parches de cesped con tonos HSL variables |
| `roof` | Tejas con filas alternadas |
| `pri` | Colores rojo/blanco/verde con simbolo circular dorado |
| `money` | Billetes con patron de pixeles aleatorio |

---

## Como Ejecutar

1. Descargar el archivo `EscondeteDelSat.html`
2. Abrirlo en cualquier navegador moderno con soporte WebGL (Chrome, Firefox, Edge)
3. Hacer clic en **INICIAR PARTIDA**
4. Hacer clic en el canvas para bloquear el puntero y comenzar a jugar

No se requiere conexion a internet una vez que la pagina ha cargado. Las fuentes y librerias se obtienen de CDNs publicos al momento de carga inicial.

---

## Creditos y Contexto

Desarrollado como proyecto individual de entretenimiento satirico. El juego es una parodia de la burocracia fiscal mexicana y no pretende promover ni ensenar evasion de impuestos real. Todos los personajes, instituciones y eventos son ficticios y tienen fines exclusivamente humoristicos.

---

*Escondete del SAT — Un juego de navegador. Un archivo. Cero facturas CFDI.*
