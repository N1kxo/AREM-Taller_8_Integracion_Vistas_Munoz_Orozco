# Registro de Trabajo en Clase - Taller 8 (Integración de Vistas) · Parte 1: FarmApp

## Fecha de la sesión
Viernes 30 de Octubre de 2026

## Integrantes presentes
- Nicolás Esteban Muñoz Sendoya
- Juan David Orozco Rodríguez

## Actividades realizadas en clase

Se siguió la metodología de 5 pasos de la [guía paso a paso](guia_paso_a_paso_integracion.md) sobre el caso base FarmApp, en draw.io (`tablero-farmapp.drawio`, una página por paso):

| Página del .drawio | Paso de la guía | Qué muestra |
|---|---|---|
| 1 · Capas sueltas | Pasos 1 y 2 | Las 5 capas y los 17 elementos de FarmApp, sin conectar |
| 2 · Hilo Compra Online | Paso 3 | Hilo de punta a punta (el de la guía) |
| 3 · Hilo Prescripción | Paso 3 | Mismo método aplicado al segundo proceso |
| 4 · Hilo Despacho | Paso 3 | Mismo método aplicado al tercer proceso |
| 5 · Tablero integrado | Paso 5 | Los 3 hilos juntos; ningún elemento huérfano |

Decisiones de modelado:
- El hilo de **Compra Online** se dejó igual que en la guía. Los hilos de **Prescripción** y **Despacho** los trazamos nosotros (la guía solo dice que "el mismo método se repite") para que Sistema POS, CRM, Servidores Regionales y Control de acceso por rol dejaran de quedar huérfanos.
- Las 4 entidades de información se conectan a la **Base de Datos Replicada** ("se persisten en"), porque la guía deja esa relación sin trazar y así cada entidad tiene conexión hacia arriba y hacia abajo.
- Los controles de seguridad se dibujan con línea punteada ("protege") y se conectan desde la aplicación que maneja el dato o el acceso.
- Colores: naranja = solo Compra Online, verde azulado = solo Prescripción, magenta = solo Despacho, gris = compartido por 2 o más hilos.

**Supuestos** (el enunciado de FarmApp no los especifica; en un cliente real saldrían de los talleres indicados):
- CRM y Plataforma E-commerce corren sobre la Nube Híbrida (Taller 4).
- El Sistema POS opera sobre Servidores Regionales para seguir vendiendo si falla la conexión a la nube (Taller 4).
- Hay datos de salud en las fórmulas médicas, por eso Prescripción exige cifrado y control de acceso por rol (Taller 5).

### Narrativa de coherencia (Paso 4)

**Compra Online.** Se apoya en la Plataforma E-commerce y en la App Móvil, priorizadas porque la mayoría de las compras ya ocurren desde dispositivos móviles. Ambas comparten las entidades Producto, Cliente, Pedido y Descuento para evitar inconsistencias de inventario entre canales, y corren sobre la Nube Híbrida con Base de Datos Replicada para tolerar picos de demanda en campañas promocionales (decisión del Taller 4). El cifrado de datos personales y el monitoreo de fraude se aplican aquí porque el proceso involucra pago electrónico, el activo más sensible (Taller 5).

**Prescripción.** La fórmula médica entra por tres puertas, por eso el proceso se apoya en tres aplicaciones: la Plataforma E-commerce (carga de la fórmula en línea), el Sistema POS (validación presencial en el mostrador) y el CRM (historial de fórmulas del cliente, para no pedirlas de nuevo). Todas consultan Cliente y Producto, porque hay que saber a quién pertenece la fórmula y si el medicamento la exige. El POS corre sobre Servidores Regionales porque validar una fórmula en el mostrador no puede depender de la conexión a la nube. La fórmula es dato personal sensible, así que el CRM y la Plataforma E-commerce se protegen con cifrado de datos personales, y la validación se restringe con control de acceso por rol (solo personal autorizado, p. ej. el químico farmacéutico, puede validar o consultar fórmulas).

**Despacho.** El pedido sale desde la tienda, por eso el Sistema POS (alistamiento e inventario por sede) es el punto central; la Plataforma E-commerce aporta el rastreo de la entrega y el CRM las notificaciones y el seguimiento al cliente. Las entidades clave son Pedido (estado), Producto (inventario por sede) y Cliente (dirección y contacto). El POS vuelve a apoyarse en Servidores Regionales para que el inventario de cada sede se mantenga aunque falle la nube, y la Base de Datos Replicada sincroniza sede y nube. El control de acceso por rol define quién puede cambiar el estado de un pedido (cajero, repartidor, administrador) y el cifrado protege dirección y datos de contacto.

### Verificación de elementos huérfanos (Paso 5)

| Elemento | Capa | ¿Conectado hacia arriba? | ¿Conectado hacia abajo? |
|---|---|---|---|
| Compra Online | Negocio | — (capa más alta) | App Móvil, Plataforma E-commerce |
| Prescripción | Negocio | — (capa más alta) | CRM, Plataforma E-commerce, Sistema POS |
| Despacho | Negocio | — (capa más alta) | CRM, Plataforma E-commerce, Sistema POS |
| App Móvil | Aplicaciones | Compra Online | Nube Híbrida |
| Plataforma E-commerce | Aplicaciones | Compra Online, Despacho, Prescripción | Cifrado de datos personales, Cliente, Descuento, Monitoreo de fraude, Nube Híbrida, Pedido, Producto |
| CRM | Aplicaciones | Despacho, Prescripción | Cifrado de datos personales, Cliente, Control de acceso por rol, Nube Híbrida |
| Sistema POS | Aplicaciones | Despacho, Prescripción | Control de acceso por rol, Pedido, Producto, Servidores Regionales |
| Descuento | Información | Plataforma E-commerce | Base de Datos Replicada |
| Cliente | Información | CRM, Plataforma E-commerce | Base de Datos Replicada |
| Pedido | Información | Plataforma E-commerce, Sistema POS | Base de Datos Replicada |
| Producto | Información | Plataforma E-commerce, Sistema POS | Base de Datos Replicada |
| Nube Híbrida | Infraestructura | App Móvil, CRM, Plataforma E-commerce | Base de Datos Replicada |
| Base de Datos Replicada | Infraestructura | Cliente, Descuento, Pedido, Producto | — (es el destino final de datos; recibe de Nube, Servidores Regionales y las 4 entidades) |
| Servidores Regionales | Infraestructura | Sistema POS | Base de Datos Replicada |
| Monitoreo de fraude | Seguridad | Plataforma E-commerce | — (capa más baja) |
| Cifrado de datos personales | Seguridad | CRM, Plataforma E-commerce | — (capa más baja) |
| Control de acceso por rol | Seguridad | CRM, Sistema POS | — (capa más baja) |

Resultado: los 17 elementos tienen al menos una conexión hacia arriba y una hacia abajo (salvo las capas extremas). Descuento queda conectado solo desde Compra Online; si el CRM maneja promociones personalizadas, faltaría la relación CRM → Descuento (ver preguntas).

## Boceto inicial del modelo

![Tablero integrado de FarmApp](tablero-farmapp-integrado.png)

Archivo editable: [`tablero-farmapp.drawio`](tablero-farmapp.drawio) (abrir en draw.io; 5 páginas).

## Preguntas para el docente
1. Para el caso base, ¿basta con un hilo completo de punta a punta o se espera ver los tres (Compra, Prescripción, Despacho)?
2. La guía deja sin trazar Información → Infraestructura. ¿Está bien conectar las 4 entidades a la Base de Datos Replicada, o se asume implícito?
3. Los supuestos (CRM sobre Nube, POS sobre Servidores Regionales): ¿se aceptan en el caso base o deben sustentarse solo con lo definido en los talleres?
4. ¿Los controles de seguridad deben conectarse solo desde Aplicaciones, o también desde Infraestructura e Información (p. ej. cifrado sobre la base de datos)?
5. En la Parte 2 (cliente real): ¿se prefiere el tablero con formas libres o con notación ArchiMate? ¿Cómo se espera marcar lo NUEVO (TO-BE) frente al AS-IS?
6. El CRM maneja promociones personalizadas según el enunciado: ¿conviene trazar CRM → Descuento, o se deja fuera del hilo?

## Retroalimentación del docente
_(completar en clase)_

## Tareas definidas para complementar el taller

| Tarea asignada | Responsable | Fecha estimada |
|----------------|-------------|----------------|
| Parte 1: revisión final del tablero y las notas con el docente | Nicolás Muñoz | Clase |
| Parte 2: `entrega/tablero-integrado-cliente.drawio` (AS-IS Talleres 1–6 + TO-BE Taller 7, marcado NUEVO) | Juan David Orozco | _por definir_ |
| Parte 2: `entrega/informe.md` y `entrega/referencias.md` | Juan David Orozco | _por definir_ |

---

_Este documento resume el trabajo colaborativo del Taller 8 en el curso AREM - Universidad de La Sabana._
