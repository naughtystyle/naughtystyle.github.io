---
layout: post
title: Microservicios
permalink: /microservicios/
tags: ["Software Architecture"]
---

# Microservicios

> El siguiente artículo es una traducción del original <a target="blank" href="https://martinfowler.com/articles/microservices.html">Microservices</a><br>
  escrito por Martin Fowler y James Lewis, realizada el 22 de Mayo de 2018

## Una definición de este nuevo término arquitectural


El término "Arquitectura de Microservicios" ha surgido en los últimos años para describir una forma particular de diseñar aplicaciones de software como conjuntos de servicios de despliegue independiente. Si bien no existe una definición precisa de este estilo arquitectónico, existen ciertas características comunes en torno a la organización acerca de la capacidad operativa, la distribución automatizada, la inteligencia en los endpoints y el control descentralizado de lenguajes y datos.

<hr />

#### Contenido

<ul class="toc">
  <li><a href="#caracteristicas">Características de una arquitectura de microservicios</a></li>
  <li><a href="#componentizacion">Componentización mediante servicios</a></li>
  <li><a href="#organizacion">Organizado en torno a las capacidades de negocio</a></li>
  <li><a href="#productos_no_proyectos">Productos no Proyectos</a></li>
  <li><a href="#endpoints_conexiones">Endpoints inteligentes y conexiones simples</a></li>
  <li><a href="#gobernanza_decentralizada">Gobernanza descentralizada</a></li>
  <li><a href="#administracion_datos">Administración de datos descentralizada</a></li>
  <li><a href="#automatizacion_infraestructura">Automatización de Infraestructura</a></li>
  <li><a href="#diseno_fallas">Diseño para fallas</a></li>
  <li><a href="#diseno_evolutivo">Diseño Evolutivo</a></li>
  <li><a href="#microservicios_futuro">¿Son los microservicios el futuro?</a></li>
</ul>

<br />

#### Barras Laterales

<ul class="toc">
  <li><a href="#tamano_microservicio">¿Qué tamaño tiene un microservicio?</a></li>
  <li><a href="#microservicios_soa">Microservicios y SOA</a></li>
  <li><a href="#muchos_lenguajes_opciones">Muchos lenguajes, muchas opciones</a></li>
  <li><a href="#estandares">Estándares probados y estándares aplicados</a></li>
  <li><a href="#hacer_lo_correcto">Facilitar el hacer lo correcto</a></li>
  <li><a href="#circuit_breaker_produccion">El Circuit Breaker y el código listo para producción</a></li>
  <li><a href="#llamadas_sincronas_daninas">Llamadas síncronas consideradas dañinas</a></li>
</ul>

<hr />

"Microservicios" - otro nuevo término en las concurridas calles de la arquitectura de software. Aunque nuestra inclinación natural es pasar por alto estas cosas con una mirada despectiva, este trozo de terminología describe un estilo de sistemas de software que encontramos cada vez más atractivos. Hemos visto muchos proyectos usar este estilo en los últimos años y los resultados hasta ahora han sido positivos, hasta el punto de que para muchos de nuestros colegas se está convirtiendo en el estilo por defecto para la creación de aplicaciones empresariales. Lamentablemente, sin embargo, no hay mucha información que describa el concepto de microservicio y cómo implementarlo.

En resumen, el estilo de arquitectura de microservicios<a href="#nota_1">[1]</a> es un enfoque para desarrollar una única aplicación como un conjunto de pequeños servicios, cada uno de los cuales se ejecuta en su propio proceso y se comunica con mecanismos ligeros, a menudo una API de recursos HTTP. Estos servicios se construyen en torno a las capacidades operativas y pueden desplegarse de forma independiente mediante maquinaria de despliegue completamente automatizada. Existe un mínimo de manejo centralizado de estos servicios, el cual puede ser escrito en diferentes lenguajes de programación y usar diferentes tecnologías de almacenamiento de datos.

Para empezar a explicar el estilo de los microservicios es conveniente compararlo con el estilo monolítico: una aplicación monolítica construida como una sola unidad. Las aplicaciones empresariales suelen constar de tres partes principales: una interfaz de usuario del lado del cliente (que consta de páginas HTML y javascript que se ejecutan en un navegador en el equipo del usuario), una base de datos (que consta de muchas tablas insertadas en un sistema de base de datos común, por lo general relacional) y una aplicación del lado del servidor. La aplicación del lado del servidor manejará las peticiones HTTP, ejecutará la lógica del dominio, recuperará y actualizará los datos de la base de datos, y seleccionará y completará las vistas HTML que se enviarán al navegador. Esta aplicación del lado del servidor es un monolito - un único ejecutable lógico<a href="#nota_2">[2]</a>. Cualquier cambio en el sistema implica el despliegue de una nueva versión de la aplicación del lado del servidor.

Un servidor monolítico de este tipo es una forma natural de abordar la construcción de un sistema de esta clase. Toda su lógica para manejar una solicitud se ejecuta en un solo proceso, permitiéndole usar las características básicas de su lenguaje para dividir la aplicación en clases, funciones y espacios de nombres (namespaces). Con un poco de cuidado, puede ejecutar y probar la aplicación en el portátil de un desarrollador y utilizar un canal de despliegue para asegurarse de que los cambios se probaron e incorporaron correctamente en producción. Se puede escalar horizontalmente el monolito ejecutando muchas instancias detrás de un balanceador de carga (load-balancer).

Las aplicaciones monolíticas pueden tener éxito, pero cada vez son más las personas que sienten frustraciones con las mismas, especialmente a medida que se despliegan más aplicaciones en la nube. Los ciclos de cambio están unidos - un cambio hecho en una pequeña parte de la aplicación, requiere que todo el monolito sea desplegado nuevamente. Con el tiempo, a menudo es difícil mantener una buena estructura modular, lo que hace más difícil mantener los cambios que sólo deberían afectar a un módulo dentro de ese módulo. El escalado requiere escalar toda la aplicación en lugar de partes de ésta que requieren mayores recursos.

<img src="/images/microservices/sketch.png" alt="Monolitos & Microservicios" class="center">

_Figura 1: Monolitos y Microservicios_

Estas frustraciones han llevado al estilo arquitectónico del microservicio: construir aplicaciones como conjuntos de servicios. Además del hecho de que los servicios son desplegables y escalables de forma independiente, cada servicio también proporciona un límite de módulo sólido, permitiendo incluso que se escriban diferentes servicios en diferentes lenguajes de programación. También pueden ser administrados por diferentes equipos.

No afirmamos que el estilo de microservicio sea novedoso o innovador, sus raíces se remontan al menos a los principios de diseño de Unix. Pero sí creemos que no hay suficiente gente que considere una arquitectura de microservicio y que muchos desarrollos de software estarían mejor si la usaran.

<a id="caracteristicas"></a>
## Características de una Arquitectura de Microservicios

No podemos decir que exista una definición formal del estilo arquitectónico de los microservicios, pero podemos intentar describir lo que vemos como características comunes para las arquitecturas que se ajustan a la categoría. Al igual que con cualquier definición que describa atributos comunes, no todas las arquitecturas de microservicios tienen todas las características, pero esperamos que la mayoría de estas exhiban la mayoría de los atributos. Si bien nosotros, los autores, hemos sido miembros activos de esta comunidad bastante dispersa, nuestra intención es intentar una descripción de lo que vemos en nuestro propio trabajo y en esfuerzos similares por parte de equipos que conocemos. En particular, no estamos estableciendo ninguna definición a la que ajustarse.

<a id="componentizacion"></a>
### Componentización mediante servicios

Durante todo el tiempo que hemos estado involucrados en la industria del software, ha habido un deseo de construir sistemas mediante la integración de componentes, mucho en la forma en que vemos que las cosas se hacen en el mundo físico. Durante las últimas dos décadas hemos visto un progreso considerable con grandes compendios de librerías comunes que forman parte de la mayoría de las plataformas lingüísticas.

Cuando hablamos de componentes nos encontramos con la difícil definición de lo que hace a un **componente**. Nuestra definición es que un componente es una unidad de software que es independientemente reemplazable y actualizable.

Las arquitecturas de microservicios utilizarán librerías, pero su principal forma de componer su propio software es dividiéndolo en servicios. Definimos las **librerías** como componentes que se enlazan a un programa y se llaman mediante llamadas a funciones en memoria, mientras que los **servicios** son componentes fuera de proceso que se comunican con un mecanismo como una solicitud de servicio web o una llamada a un procedimiento remoto. (Este es un concepto diferente al de un objeto de servicio en muchos programas OO<a href="#nota_3">[3]</a>.

Una razón principal para utilizar los servicios como componentes (en lugar de librerías) es que los servicios se pueden implementar de forma independiente. Si tiene una aplicación<a href="#nota_4">[4]</a> que consiste en múltiples librerías en un solo proceso, un cambio en cualquier componente resulta en tener que volver a desplegar toda la aplicación. Pero si esa aplicación se descompone en múltiples servicios, puede esperar que muchos cambios en un solo servicio sólo requieran la redistribución de ese servicio. Eso no es un hecho, algunos cambios cambiarán las interfaces de servicio resultando en cierta coordinación, pero el objetivo de una buena arquitectura de microservicio es minimizarlas a través de límites de servicio cohesivos y mecanismos de evolución en los contratos de servicio.

Otra consecuencia del uso de servicios como componentes es una interfaz de componentes más explícita. La mayoría de los lenguajes no tienen un buen mecanismo para definir una Interfaz Publicada <a target="blank" href="https://martinfowler.com/bliki/PublishedInterface.html">(Published Interface)</a> explícita. A menudo es sólo la documentación y la disciplina lo que evita que los clientes rompan el encapsulado de un componente, lo que conduce a un acoplamiento demasiado estrecho entre los componentes. Los servicios hacen más fácil evitarlo utilizando mecanismos explícitos de llamada remota.

Usar servicios como este tiene sus desventajas. Las llamadas remotas son más costosas que las llamadas en proceso y, por lo tanto, las API remotas deben ser más granulares, lo que a menudo es más incómodo de usar. Si necesita cambiar la asignación de responsabilidades entre componentes, tales movimientos de comportamiento son más difíciles de hacer cuando está cruzando los límites del proceso.

En una primera aproximación, podemos observar que los servicios mapean los procesos en tiempo de ejecución, pero eso es sólo una primera aproximación. Un servicio puede consistir en múltiples procesos que siempre se desarrollarán y desplegarán juntos, como un proceso de aplicación y una base de datos que sólo es utilizada por ese servicio.

<a id="organizacion"></a>
## Organizado en torno a las areas de negocio

Cuando se busca dividir una aplicación grande en partes, a menudo se centra en la capa de tecnología, lo que conduce a dividir en equipos de interfaz de usuario, equipos de lógica del lado del servidor y equipos de bases de datos. Cuando los equipos se separan en este sentido, incluso los cambios más sencillos pueden llevar a que un proyecto entre equipos requiera tiempo y aprobación presupuestaria. Un equipo inteligente optimizará en torno a esto y se inclinará por el menor de los dos males: sólo tiene que forzar la lógica en cualquier aplicación a la que tenga acceso. Lógica en todas partes, en otras palabras. Este es un ejemplo de la Ley de Conway<a href="#nota_5">[5]</a> en acción.

> Cualquier organización que diseñe un sistema (definido ampliamente) producirá un diseño cuya estructura es una copia de la estructura de comunicación de la organización.
-- Melvyn Conway, 1967

<img src="/images/microservices/conways_law.png" alt="Ley de Conway" class="center">

_Figura 2: Ley de Conway en acción_

El enfoque de la división desde el punto de vista de los microservicios es diferente, ya que se divide en servicios organizados en base a la **capacidad empresarial**. Tales servicios requieren una amplia implementación de software para esa área de negocio, incluyendo interfaz de usuario, almacenamiento persistente y cualquier colaboración externa. En consecuencia, los equipos son multifuncionales, incluyendo toda la gama de habilidades necesarias para el desarrollo: experiencia del usuario, base de datos y administración de proyectos.

<img src="/images/microservices/prefer_functional_staff_organisation.png" alt="Organizacion de Equipos" class="center">

_Figura 3: Limites del servicio reafirmado por los limites del equipo_

Una empresa organizada de esta manera es <a target="blank" href="http://www.comparethemarket.com">www.comparethemarket.com</a>. Los equipos multifuncionales son responsables de la construcción y operación de cada producto y cada producto se divide en una serie de servicios individuales que se comunican a través de un bus de mensajes (message bus).

Las aplicaciones monolíticas de gran tamaño también pueden modularizarse en función de las capacidades empresariales, aunque no es el caso común. Ciertamente, nos gustaría instar a un gran equipo de construcción de una aplicación monolítica para dividirse a sí mismo a lo largo de las líneas de negocio. El principal problema que hemos visto aquí es que tienden a organizarse en demasiados contextos. Si el monolito se extiende a lo largo de muchos de estos límites modulares puede ser difícil para los miembros individuales de un equipo encajarlos en su memoria a corto plazo. Además, vemos que las líneas modulares requieren una gran disciplina para su aplicación. La separación necesariamente más explícita que requieren los componentes de servicio hace más fácil mantener claros los límites del equipo.

<a id="tamano_microservicio"></a>
<div class="side-note">
### ¿Qué tamaño tiene un microservicio?

Aunque "microservicio" se ha convertido en un nombre popular para este estilo arquitectónico, su nombre lleva a un desafortunado enfoque en el tamaño del servicio, y argumentos sobre lo que constituye "micro". En nuestras conversaciones con los practicantes de microservicios, vemos una variedad de tamaños de servicios. Los tamaños más grandes reportados siguen la noción de Amazon del Equipo de Dos Pizzas (es decir, todo el equipo puede ser alimentado con dos pizzas), lo que significa no más de una docena de personas. En la escala de menor tamaño hemos visto configuraciones en las que un equipo de media docena soportaría media docena de servicios.

Esto lleva a la pregunta de si hay diferencias suficientemente grandes dentro de este rango de tamaño como para que los tamaños de servicio por docena de personas y de servicio por persona no se agrupen bajo una etiqueta de microservicios. Por el momento pensamos que es mejor agruparlas, pero es posible que cambiemos de opinión a medida que exploremos más este estilo.
</div>


<a id="productos_no_proyectos"></a>
## Productos no Proyectos

La mayoría de los esfuerzos de desarrollo de aplicaciones que vemos utilizan un modelo de proyecto: donde el objetivo es entregar alguna pieza de software que luego se considera completada. Una vez finalizado, el software se entrega a una organización de mantenimiento y el equipo del proyecto que lo construyó se disuelve.

Los proponentes de los microservicios tienden a evitar este modelo, prefiriendo en cambio la noción de que un equipo debe poseer un producto durante toda su vida útil. Una inspiración común para esto es la noción de Amazon de <a target="blank" href="https://queue.acm.org/detail.cfm?id=1142065">"tú lo construyes, tú lo diriges"</a>, donde un equipo de desarrollo asume toda la responsabilidad del software en producción. Esto pone a los desarrolladores en contacto diario con el comportamiento de su software en la producción y aumenta el contacto con sus usuarios, ya que tienen que asumir al menos parte de la carga de soporte.

La mentalidad de producto, enlaza con la vinculación con las capacidades empresariales. En lugar de considerar el software como un conjunto de funcionalidades a completar, existe una relación continua en la que la cuestión es cómo puede el software ayudar a sus usuarios a mejorar la capacidad de negocio.

No hay razón por la que no se pueda adoptar este mismo enfoque con las aplicaciones monolíticas, pero la menor granularidad de los servicios puede facilitar la creación de relaciones personales entre los desarrolladores de servicios y sus usuarios.

<a id="endpoints_conexiones"></a>
## Puntos Finales (Endpoints) inteligentes y conexiones simples

Al construir estructuras de comunicación entre los diferentes procesos, hemos visto muchos productos y enfoques que enfatizan la importancia de poner inteligencia significativa en el mecanismo de comunicación en sí. Un buen ejemplo de esto es el Enterprise Service Bus (ESB), donde los productos ESB a menudo incluyen instalaciones sofisticadas para el enrutamiento de mensajes, coreografía, transformación y aplicación de reglas de negocio.

La comunidad de microservicios favorece un enfoque alternativo: puntos finales (endpoints) inteligentes y conexiones simples. Las aplicaciones construidas a partir de microservicios tienen como objetivo ser tan desacopladas y cohesivas como sea posible - poseen su propia lógica de dominio y actúan más como filtros en el sentido clásico de Unix - recibiendo una petición, aplicando la lógica apropiada y produciendo una respuesta. Estas son coreografiadas usando protocolos simples tipo REST en lugar de protocolos complejos como WS-Choreography o BPEL u orquestación por una herramienta central.

Los dos protocolos utilizados más comúnmente son HTTP con recursos API y mensajería ligera<a href="#nota_8">[8]</a>. La mejor expresión de la primera es

>Sea de la web, no detrás de la web
<a target="blank" href="https://www.amazon.com/gp/product/0596805829?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0596805829">-- Ian Robinson</a>

Los equipos de microservicios utilizan los principios y protocolos sobre los que se construye la World Wide Web (y en gran medida, Unix). Los recursos utilizados a menudo pueden almacenarse en caché con muy poco esfuerzo por parte de los desarrolladores o del personal de operaciones.

El segundo enfoque de uso común es la mensajería a través de un bus de mensajes ligero. La infraestructura elegida es típicamente simple (simple ya que actua sólo como un enrutador de mensajes) - las implementaciones simples como RabbitMQ o ZeroMQ no hacen mucho más que proveer una estructura asíncrona confiable - la parte inteligente todavía vive en los puntos finales que están produciendo y consumiendo mensajes; en los servicios.

En un monolito, los componentes se ejecutan durante el proceso y la comunicación entre ellos se realiza mediante la invocación del método o la llamada a la función. El mayor problema para transformar un monolito en microservicios radica en cambiar el patrón de comunicación. Una conversión ingenua de las llamadas de método en memoria a RPC conduce a comunicaciones de conversación que no funcionan bien. En su lugar, debe sustituir la comunicación hacia un enfoque menos granular.

<a id="microservicios_soa"></a>
<div class="side-note">
### Microservicios y SOA

Cuando hablamos de microservicios, una pregunta común es si se trata de una Arquitectura Orientada a Servicios (SOA)
como la que vimos hace una década. Hay mérito en este punto, porque el estilo de microservicio es muy similar a lo que algunos defensores de
SOA han estado a favor. El problema, sin embargo, es que SOA significa demasiadas cosas diferentes, y que la mayoría de las veces que nos
encontramos con algo llamado "SOA" es significativamente diferente al estilo que estamos describiendo aquí, usualmente debido a un enfoque en
ESBs usados para integrar aplicaciones monolíticas.

En particular, hemos visto tantas implementaciones fallidas de la orientación a servicios - desde la tendencia a ocultar la complejidad en los
ESB<a href="#nota_6">[6]</a>, a iniciativas plurianuales fallidas que cuestan millones y no aportan ningún valor, a modelos de gobierno centralizado que inhiben
activamente el cambio, que a veces es difícil ver más allá de estos problemas.

Ciertamente, muchas de las técnicas que se utilizan en la comunidad de microservicios han crecido a partir de las experiencias de los
desarrolladores que integran servicios en grandes organizaciones. El patrón de Lector Tolerante (Tolerant Reader) es un ejemplo de esto. Los esfuerzos por
utilizar la web han contribuido, el uso de protocolos sencillos es otro enfoque derivado de estas experiencias - una reacción lejos de los
estándares centrales que han alcanzado una complejidad que es, francamente, impresionante. (Cada vez que usted necesita una ontología para
manejar sus ontologías, se da cuenta de que está en serios problemas.)

Esta manifestación común de SOA ha llevado a algunos defensores de los microservicios a rechazar la etiqueta de SOA por completo, aunque otros
consideran que los microservicios son una forma de SOA<a href="#nota_7">[7]</a>, tal vez la orientación a servicios bien hecha. De cualquier manera, el hecho de que
SOA signifique cosas tan diferentes significa que es valioso tener un término que defina más claramente este estilo arquitectónico.
</div>

<a id="gobernanza_decentralizada"></a>
## Gobernanza Descentralizada

Una de las consecuencias de la gobernanza centralizada es la tendencia a la normalización en plataformas tecnológicas concretas. La experiencia demuestra que este enfoque es restrictivo - no todo problema es un clavo y no toda solución un martillo. Preferimos utilizar la herramienta adecuada para el trabajo y aunque las aplicaciones monolíticas pueden aprovechar los diferentes lenguajes hasta cierto punto, no es tan común.

Al dividir los componentes del monolito en servicios, podemos elegir a la hora de construir cada uno de ellos. ¿Desea utilizar Node.js para poner de pie una página de reportes simple? Vamos. ¿C++ para un componente particularmente nudoso y casi en tiempo real? Bien. ¿Desea cambiar a un tipo diferente de base de datos que mejor se adapte al comportamiento de lectura de un componente? Tenemos la tecnología para reconstruirlo.

Por supuesto, el hecho de que pueda hacer algo no significa que deba hacerlo, pero particionar su sistema de esta manera significa que tiene la opción.

Los equipos que crean microservicios también prefieren un enfoque diferente de las normas. En lugar de utilizar un conjunto de estándares definidos escritos en algún papel, prefieren la idea de producir herramientas útiles que otros desarrolladores puedan usar para resolver problemas similares a los que están enfrentando. Estas herramientas son usualmente recolectadas de implementaciones y compartidas con un grupo más amplio, algunas veces, pero no exclusivamente, usando un modelo interno de código abierto. Ahora que git y github se han convertido en el sistema de control de versiones por defecto, las prácticas de código abierto son cada vez más comunes en las empresas.

Netflix es un buen ejemplo de una organización que sigue esta filosofía. Compartir código útil y, sobre todo, probado en el campo de batalla, ya que las librerías animan a otros desarrolladores a resolver problemas similares de forma similar, pero deja la puerta abierta para elegir un enfoque diferente si es necesario. Las librerías compartidas tienden a centrarse en problemas comunes de almacenamiento de datos, comunicación entre procesos y, como veremos más adelante, automatización de la infraestructura.

Para la comunidad de microservicios, los excesos son particularmente poco atractivos. Eso no quiere decir que la comunidad no valore los contratos de servicio. Todo lo contrario, ya que suelen ser muchos más. Es sólo que están buscando diferentes formas de manejar esos contratos. Patrones como <a target="blank" href="https://martinfowler.com/bliki/TolerantReader.html">Tolerant Reader</a> y <a target="blank" href="https://martinfowler.com/articles/consumerDrivenContracts.html">Consumer-Driven Contracts</a> se aplican a menudo a los microservicios. Estos contratos de servicios de ayuda evolucionan de forma independiente. La ejecución de contratos impulsados por el consumidor como parte de su construcción aumenta la confianza y proporciona una rápida retroalimentación sobre si sus servicios están funcionando o no. De hecho, conocemos un equipo en Australia que impulsa la construcción de nuevos servicios con contratos orientados al consumidor. Utilizan herramientas sencillas que les permiten definir el contrato de un servicio. Esto se convierte en parte del build automatizado antes incluso de que se escriba el código para el nuevo servicio. El servicio se construye entonces sólo hasta el punto en que satisface el contrato - un enfoque elegante para evitar el dilema "YAGNI"<a href="#nota_9">[9]</a> cuando se construye nuevo software. Estas técnicas y las herramientas que crecen en torno a ellas, limitan la necesidad de una administración central de contratos al disminuir el acoplamiento temporal entre servicios.

Tal vez el apogeo de la gobernanza descentralizada sea el espíritu de constrúyala / diríjala popularizado por Amazon. Los equipos son responsables de todos los aspectos del software que construyen, incluyendo la operación del software 24/7. La devolución de este nivel de responsabilidad no es la norma, pero vemos que cada vez más empresas trasladan la responsabilidad a los equipos de desarrollo. Netflix es otra organización que ha adoptado este ethos<a href="#nota_11">[11]</a>. Ser despertado a las 3 de la madrugada todas las noches por su localizador es sin duda un poderoso incentivo para centrarse en la calidad a la hora de escribir su código. Estas ideas están lo más lejos posible del modelo tradicional de gobierno centralizado.

<a id="muchos_lenguajes_opciones"></a>
<div class="side-note">
### Muchos lenguajes, muchas opciones

El crecimiento de la JVM como plataforma es sólo el último ejemplo de mezcla de lenguajes dentro de una plataforma común. Ha sido una práctica común la renuncia a un lenguaje de mayor nivel para aprovechar las abstracciones de mayor nivel durante décadas. Como es utilizar lenguajes de bajo nivel y escribir código sensible al performance en un nivel inferior. Sin embargo, muchos monolitos no necesitan este nivel de optimización del performance, ni son comunes (para nuestra consternación) las abstracciones DSL y de nivel superior. En cambio, los monolitos suelen ser monolingües y la tendencia es limitar el número de tecnologías en uso<a href="#nota_10">[10]</a>.
</div>

<a id="administracion_datos"></a>
## Administración Descentralizada de Datos

La descentralización de la administración de datos se presenta de varias maneras diferentes. En el nivel más abstracto, significa que el modelo conceptual del mundo diferirá entre sistemas. Este es un problema común cuando se integra en una gran empresa, la vista del departamento de ventas hacia un cliente será diferente de la vista de soporte técnico. Algunas cosas que se llaman clientes del punto de vista de ventas pueden no aparecer en absoluto de lado de soporte. Aquellos que puedan tener atributos diferentes y (peor aun) atributos comunes con semántica sutilmente diferente.

Este problema es común entre aplicaciones, pero también puede ocurrir dentro de las aplicaciones, especialmente cuando esa aplicación está dividida en componentes separados. Una forma útil de pensar sobre esto es la noción de Contexto Limitado <a target="blank" href="https://martinfowler.com/bliki/BoundedContext.html">(Bounded Context)</a> de Diseño Dirigido por Dominio. DDD divide un dominio complejo en múltiples contextos limitados y mapea las relaciones entre ellos. Este proceso es útil tanto para arquitecturas monolíticas como de microservicio, pero existe una correlación natural entre los límites de servicio y contexto que ayuda a clarificar, y como describimos en la sección sobre capacidades de negocio, reforzar las separaciones.

Además de descentralizar las decisiones sobre modelos conceptuales, los microservicios también descentralizan las decisiones de almacenamiento de datos. Mientras que las aplicaciones monolíticas prefieren una única base de datos lógica para los datos persistentes, las empresas a menudo prefieren una única base de datos para toda una gama de aplicaciones - muchas de estas decisiones se toman a través de los modelos comerciales de los proveedores en torno a la concesión de licencias. Los microservicios prefieren dejar que cada servicio administre su propia base de datos, ya sea diferentes instancias de la misma tecnología de base de datos, o sistemas de base de datos completamente diferentes - un enfoque llamado Persistencia Políglota <a target="blank" href="https://martinfowler.com/bliki/PolyglotPersistence.html">Polyglot Persistence</a>. Se puede utilizar la persistencia políglota en un monolito, pero aparece más frecuentemente con microservicios.

<img src="/images/microservices/decentralised_data.png" alt="Datos Descentralizados" class="center">

_Figura 4: Datos Centralizados/Decentralizados_

La responsabilidad de la descentralización de los datos en todos los microservicios tiene consecuencias para la gestión de las actualizaciones. El enfoque común para hacer frente a las actualizaciones ha consistido en utilizar las transacciones para garantizar la coherencia al actualizar múltiples recursos. Este enfoque se utiliza a menudo dentro de los monolitos.

El uso de transacciones de este tipo ayuda a la coherencia, pero impone un acoplamiento temporal significativo, lo que resulta problemático en múltiples servicios. Las transacciones distribuidas son notoriamente difíciles de implementar y, en consecuencia, las arquitecturas de microservicios hacen hincapié en la coordinación sin transacciones entre servicios, con el reconocimiento explícito de que la coherencia sólo puede ser una coherencia eventual y que los problemas se resuelven mediante operaciones que compensan esto.

Decidir manejar las inconsistencias de esta manera es un nuevo desafío para muchos equipos de desarrollo, pero es uno que a menudo se ajusta a las prácticas comerciales. A menudo las empresas manejan un grado de inconsistencia con el fin de responder rápidamente a la demanda, mientras que tienen algún tipo de proceso de inversión para hacer frente a los errores. El trueque vale la pena siempre y cuando el costo de corregir errores sea menor que el costo de perder negocios bajo una mayor consistencia.

<a id="estandares"></a>
<div class="side-note">
### Estándares probados y estándares aplicados
Es una especie de contradicción que los equipos de microservicios tiendan a evitar el tipo de estándares rígidos impuestos por los grupos de arquitectura empresarial, pero con gusto usarán e incluso evangelizarán el uso de estándares abiertos como HTTP, ATOM y otros microformatos.

La diferencia clave es cómo se desarrollan las normas y cómo se aplican. Los estándares gestionados por grupos como el IETF sólo se convierten en estándares cuando hay varias implementaciones de los mismos en el mundo y que a menudo crecen a partir de proyectos exitosos de código abierto.

Estos estándares son un mundo aparte de muchos en el mundo corporativo, que a menudo son desarrollados por grupos que tienen poca experiencia en programación o excesivamente influenciados por los proveedores.
</div>

<a id="automatizacion_infraestructura"></a>
## Automatización de Infraestructura

Las técnicas de automatización de infraestructuras han evolucionado enormemente en los últimos años: la evolución de la nube y de AWS en particular ha reducido la complejidad operativa de la construcción, el despliegue y la operación de microservicios.

Muchos de los productos o sistemas que se están construyendo con microservicios están siendo construidos por equipos con amplia experiencia en la <a target="blank" href="https://martinfowler.com/bliki/ContinuousDelivery.html">Entrega Continua (Continuous Delivery)</a> y su precursor, la <a target="blank" href="https://martinfowler.com/articles/continuousIntegration.html">Integración Continua (Continuous Integration)</a>. Los equipos que construyen software de esta manera hacen un uso extensivo de las técnicas de automatización de infraestructura. Esto se ilustra a continuación.

<img src="/images/microservices/basic_pipeline.png" alt="Build" class="center">

_Figura 5: Flujo básico de build_

Como este no es un artículo sobre Entrega Continua, pondremos atención solo a un par de características clave. Queremos la mayor confianza posible en que nuestro software funciona, por lo que realizamos muchos **tests automatizados**. La promoción del software de trabajo 'arriba' de la tubería significa que **automatizamos el despliegue** en cada nuevo entorno.

Una aplicación monolítica será construida, testeada y empujada a través de estos ambientes alegremente. Resulta que una vez que se ha invertido en automatizar el camino a la producción de un monolito, entonces desplegar más aplicaciones ya no parece tan aterrador. Recuerde, uno de los objetivos de la Entrega Continua es hacer que el despliegue sea aburrido, así que ya sea que se trate de una o tres aplicaciones, siempre y cuando sea aburrido no importa<a href="#nota_12">[12]</a>.

Otra área en la que vemos equipos que utilizan una amplia automatización de la infraestructura es en la administración de microservicios en la producción. En contraste con nuestra afirmación anterior de que mientras el despliegue sea aburrido no hay tanta diferencia entre monolitos y microservicios, el panorama operacional para cada uno puede ser notablemente diferente.

<img src="/images/microservices/micro_deployment.png" alt="Despliegue de Microservicios" class="center">

_Figura 6: El despliegue de modulos es a menudo diferente_

<a id="hacer_lo_correcto"></a>
<div class="side-note">
### Facilitar el hacer lo correcto

Un efecto secundario que hemos encontrado del aumento de la automatización como consecuencia de la entrega y despliegue continuos es la creación de herramientas útiles para ayudar a los desarrolladores y al personal de operaciones. Las herramientas para la creación de artefactos, la administración de bases de código, la creación de servicios simples o la adición de monitoreo y registro estándar son bastante comunes en la actualidad. El mejor ejemplo en la web es probablemente el <a target="blank" href="http://netflix.github.io/">conjunto de herramientas de código abierto de Netflix</a>, pero hay otros incluyendo <a target="blank" href="http://dropwizard.codahale.com/">Dropwizard</a> que hemos utilizado ampliamente.
</div>

<a id="diseno_fallas"></a>
## Diseño para fallas

Una consecuencia del uso de los servicios como componentes es que las aplicaciones deben diseñarse de manera que puedan tolerar el fallo de los servicios. Cualquier llamada de servicio puede fallar debido a la falta de disponibilidad del proveedor, el cliente tiene que responder a esto tan elegantemente como sea posible. Esto es una desventaja en comparación con un diseño monolítico, ya que introduce complejidad adicional para manejarlo. La consecuencia es que los equipos de microservicios reflexionan constantemente sobre cómo las fallas en los servicios afectan la experiencia del usuario. <a target="blank" href="https://github.com/Netflix/SimianArmy">Simian Army</a> de Netflix induce fallas en los servicios e incluso en los centros de datos durante la jornada laboral para probar tanto la resiliencia como la monitorización de la aplicación.

Este tipo de prueba automatizada en la producción sería suficiente para dar a la mayoría de los grupos de operación el tipo de escalofríos que normalmente preceden a una semana de ausencia del trabajo. Esto no quiere decir que los estilos arquitectónicos monolíticos no sean capaces de configuraciones de monitorización sofisticadas - es solamente menos común en nuestra experiencia.

Dado que los servicios pueden fallar en cualquier momento, es importante poder detectar los fallos rápidamente y, si es posible, restaurar el servicio automáticamente. Las aplicaciones de microservicios ponen mucho énfasis en la monitorización en tiempo real de la aplicación, comprobando tanto los elementos arquitectónicos (cuántas peticiones por segundo recibe la base de datos) como las métricas relevantes para el negocio (por ejemplo, cuántos pedidos por minuto se reciben). El monitoreo semántico puede proporcionar un sistema de alerta temprana de que algo va mal que desencadena que los equipos de desarrollo hagan un seguimiento e investiguen.

Esto es particularmente importante para una arquitectura de microservicios porque la preferencia de los microservicios por la coreografía y la <a target="blank" href="https://martinfowler.com/eaaDev/EventCollaboration.html">colaboración en eventos</a> conduce a un comportamiento emergente. Mientras que muchos expertos elogian el valor del surgimiento fortuito, la verdad es que el comportamiento emergente a veces puede ser algo malo. El monitoreo es vital para detectar rápidamente el mal comportamiento emergente y poder corregirlo.

Los monolitos pueden construirse para que sean tan transparentes como un microservicio; de hecho, deberían serlo. La diferencia es que es absolutamente necesario saber cuándo se desconectan los servicios que se ejecutan en procesos diferentes. Con las librerías dentro del mismo proceso, es menos probable que este tipo de transparencia sea útil.

Los equipos de microservicios esperarán ver configuraciones sofisticadas de monitoreo y registro para cada servicio individual, tales como cuadros estadísticos que muestren el estado ascendente/descendente y una variedad de métricas operativas y de negocios relevantes. Los detalles sobre el estado de los cortacircuitos (Circuit Breakers), el rendimiento de corriente y la latencia son otros ejemplos que encontramos a menudo en la naturaleza.

<a id="circuit_breaker_produccion"></a>
<div class="side-note">
### El Circuit Breaker y el código listo para producción

El <a target="blank" href="https://martinfowler.com/bliki/CircuitBreaker.html">Circuit Breaker</a> (disyuntor) aparece en el <a target="blank" href="https://www.amazon.com/gp/product/B00A32NXZO?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=B00A32NXZO">Release It!</a> junto con otros patrones como el Bulkhead y el Timeout. Implementados juntos, estos patrones son de crucial importancia a la hora de construir aplicaciones de comunicación. Esta <a target="blank" href="http://techblog.netflix.com/2012/02/fault-tolerance-in-high-volume.html">entrada de blog de Netflix</a> hace un gran trabajo explicando su aplicación.
</div>

<a id="diseno_evolutivo"></a>
## Diseño Evolutivo

Los practicantes de microservicios, por lo general, provienen de un contexto de diseño evolutivo y ven la descomposición de servicios como una herramienta más para permitir a los desarrolladores de aplicaciones controlar los cambios en sus aplicaciones sin ralentizar el cambio. El control del cambio no significa necesariamente la reducción del cambio - con las actitudes y herramientas correctas puede realizar cambios frecuentes, rápidos y bien controlados en el software.

Cada vez que intentas dividir un sistema de software en componentes, te encuentras con la decisión de cómo dividir las piezas - ¿cuáles son los principios sobre los que decidimos dividir nuestra aplicación? La propiedad clave de un componente es la noción de sustitución independiente y de capacidad de actualización<a href="#nota_13">[13]</a>. lo que implica que busquemos puntos en los que podamos imaginar la reescritura de un componente sin afectar a sus colaboradores. De hecho, muchos grupos de microservicios van más allá y esperan explícitamente que muchos servicios sean desechados en lugar de evolucionar a largo plazo.

El sitio web The Guardian es un buen ejemplo de una aplicación que fue diseñada y construida como un monolito, pero que ha estado evolucionando en una dirección de microservicio. El monolito sigue siendo el núcleo del sitio web, pero prefieren añadir nuevas características mediante la creación de microservicios que utilizan la API del monolito. Este enfoque es particularmente útil para características que son intrínsecamente temporales, como páginas especializadas para manejar un evento deportivo. Esta parte del sitio web se puede crear rápidamente utilizando lenguajes de desarrollo rápido y eliminar una vez finalizado el evento. Hemos visto enfoques similares en una institución financiera donde se añaden nuevos servicios para una oportunidad de mercado y se descartan después de unos meses o incluso semanas.

Este énfasis en la reemplazabilidad es un caso especial de un principio más general de diseño modular, que consiste en impulsar la modularidad a través del patrón de cambio<a href="#nota_14">[14]</a>. Con el fin de mantener las cosas que cambian al mismo tiempo en el mismo módulo. Las partes de un sistema que cambian raramente deberían estar en servicios diferentes a los que actualmente tienen muchos cambios. Si te encuentras cambiando repetidamente dos servicios juntos, eso es una señal de que deberían fusionarse.

La puesta en servicios de los componentes agrega una oportunidad para una planificación más granular en el planeamiento de un lanzamiento. Con un monolito cualquier cambio requiere una compilación y despliegue completo de toda la aplicación. Sin embargo, con los microservicios, sólo necesita redistribuir el servicio o servicios que modificó. Esto puede simplificar y acelerar el proceso de lanzamiento. La desventaja es que usted tiene que preocuparse por los cambios en un servicio que quiebra a sus consumidores. El enfoque tradicional de integración consiste en tratar de resolver este problema utilizando el versionado, pero la preferencia en el mundo de los microservicios es <a target="blank" href="https://martinfowler.com/articles/enterpriseREST.html#versioning">utilizar el versionado sólo como último recurso</a>. Podemos evitar una gran cantidad de versionado diseñando servicios que sean lo más tolerantes posible a los cambios en sus proveedores.

<a id="llamadas_sincronas_daninas"></a>
<div class="side-note">
### Llamadas síncronas consideradas dañinas

Cada vez que tenga varias llamadas síncronas entre servicios, se encontrará con el efecto multiplicador del tiempo de inactividad. Simplemente, es cuando el tiempo de parada de su sistema se convierte en el producto de los tiempos de parada de los componentes individuales. Usted tiene que elegir entre realizar sus llamadas de forma asincrónica o gestionar el tiempo de inactividad. En www.guardian.co.uk han implementado una regla simple en la nueva plataforma - una llamada síncrona por cada solicitud de usuario, mientras que en Netflix, su rediseño de la API de la plataforma ha incorporado asincronicidad en la estructura de la API.
</div>

<a id="microservicios_futuro"></a>
## ¿Son los Microservicios el futuro?

Nuestro principal objetivo al escribir este artículo es explicar las principales ideas y principios de los microservicios. Al tomarnos el tiempo para hacer esto, pensamos claramente que el estilo arquitectónico de los microservicios es una idea importante - una que vale la pena considerar seriamente para las aplicaciones empresariales. Recientemente hemos construido varios sistemas usando el estilo y sabemos de otros que han usado y favorecen este enfoque.

Aquellos de los que sabemos que de alguna manera son pioneros en el estilo arquitectónico incluyen Amazon, Netflix, <a target="blank" href="http://www.theguardian.com/">The Guardian</a>, <a target="blank" href="https://gds.blog.gov.uk/">the UK Government Digital Service</a>, <a target="blank" href="https://martinfowler.com/articles/realestate.com.au">realstate.com.au</a>, Forward y <a target="blank" href="http://www.comparethemarket.com/">comparethemarket.com</a>. El circuito de conferencias en 2013 estuvo lleno de ejemplos de compañías que se están moviendo a algo que se clasificaría como microservicios - incluyendo a Travis CI. Además, hay muchas organizaciones que han estado haciendo durante mucho tiempo lo que clasificaríamos como microservicios, pero sin usar nunca el nombre. (A menudo esto se etiqueta como SOA - aunque, como hemos dicho, SOA viene en muchas formas contradictorias. <a href="#nota_15">[15]</a>)

A pesar de estas experiencias positivas, no estamos argumentando que estemos seguros de que los microservicios son la dirección futura para las arquitecturas de software. Aunque nuestras experiencias hasta ahora son positivas en comparación con las aplicaciones monolíticas, somos conscientes del hecho de que no ha pasado suficiente tiempo para que podamos hacer un juicio completo.

A menudo las verdaderas consecuencias de sus decisiones arquitectónicas sólo son evidentes varios años después de haberlas tomado. Hemos visto proyectos en los que un buen equipo, con un fuerte deseo de modularidad, ha construido una arquitectura monolítica que se ha deteriorado con el paso de los años. Mucha gente cree que tal deterioro es menos probable con los microservicios, ya que los límites de los servicios son explícitos y difíciles de parchar. Sin embargo, hasta que no veamos suficientes sistemas con suficiente edad, no podremos evaluar realmente cómo maduran las arquitecturas de microservicios.

Ciertamente hay razones por las que uno podría esperar que los microservicios maduren pobremente. En cualquier esfuerzo de componentización, el éxito depende de lo bien que el software encaje en los componentes. Es difícil averiguar exactamente dónde deben estar los límites de los componentes. El diseño evolutivo reconoce las dificultades de establecer bien los límites y, por lo tanto, la importancia de que sea fácil refactorizarlos. Pero cuando sus componentes son servicios con comunicaciones remotas, entonces la refactorización es mucho más difícil que con las librerías en proceso. Mover código es difícil a través de los límites del servicio, cualquier cambio en la interfaz necesita ser coordinado entre los participantes, capas de retrocompatibilidad necesitan ser agregadas, y los testeos son más complicados.

Otro problema es que si los componentes no se componen limpiamente, todo lo que está haciendo es cambiar la complejidad del interior de un componente a las conexiones entre componentes. Esto no sólo mueve la complejidad, sino que la traslada a un lugar que es menos explícito y más difícil de controlar. Es fácil pensar que las cosas son mejores cuando se mira el interior de un componente pequeño y simple, en vez de conexiones complejas entre servicios.

Por último, está el factor de la habilidad del equipo. Las nuevas técnicas tienden a ser adoptadas por equipos más hábiles. Pero una técnica que es más efectiva para un equipo más hábil no necesariamente va a funcionar para equipos menos hábiles. Hemos visto muchos casos de equipos menos hábiles construyendo arquitecturas monolíticas desordenadas, pero lleva tiempo ver lo que ocurre cuando este tipo de lío ocurre con los microservicios. Un equipo pobre siempre creará un sistema pobre - es muy difícil saber si los microservicios reducen el desorden en este caso o lo empeoran.

Un argumento razonable que hemos escuchado es que no se debe empezar con una arquitectura de microservicios. En su lugar, <a target="blank" href="https://martinfowler.com/bliki/MonolithFirst.html">comience con un monolito</a>, manténgalo modular y divídalo en microservicios una vez que el monolito se convierta en un problema. (Aunque <a target="blank" href="https://martinfowler.com/articles/dont-start-monolith.html">este consejo no es el ideal</a>, ya que una buena interfaz en proceso no suele ser una buena interfaz de servicio.)

Así que escribimos esto con cauteloso optimismo. Hasta ahora, hemos visto suficiente sobre el estilo del microservicio como para sentir que puede ser <a target="blank" href="https://martinfowler.com/microservices/">un camino que valga la pena recorrer</a>. No podemos decir con seguridad dónde terminaremos, pero uno de los retos del desarrollo de software es que sólo se pueden tomar decisiones basadas en la información imperfecta que actualmente se tiene a mano.

<hr />

### Notas a pie de página

<a id="nota_1">[1]</a> El término "microservicio" fue discutido en un taller de arquitectos de software cerca de Venecia en mayo de 2011 para describir lo que los participantes vieron como un estilo arquitectónico común que muchos de ellos habían estado explorando recientemente. En mayo de 2012, el mismo grupo se decidió por "microservicios" como el nombre más apropiado. James presentó algunas de estas ideas como un estudio de caso en marzo de 2012 en el 33º Grado en Cracovia en Microservicios - Java, el camino de Unix al igual que Fred George más o menos al mismo tiempo. Adrian Cockcroft en Netflix, describiendo este enfoque como "SOA de grano fino" fue pionero en el estilo a escala web, al igual que muchos de los otros mencionados en este artículo - Joe Walnes, Dan North, Evan Botcher y Graham Tackley.

<a id="nota_2">[2]</a> El término monolito ha sido usado por la comunidad Unix durante algún tiempo. Aparece en The Art of Unix Programming para describir sistemas que se hacen demasiado grandes.

<a id="nota_3">[3]</a> Muchos diseñadores orientados a objetos, incluidos nosotros mismos, utilizamos el término objeto de servicio en el sentido de diseño orientado a dominio (DDD) para un objeto que lleva a cabo un proceso significativo que no está vinculado a una entidad. Este es un concepto diferente de cómo estamos usando "servicio" en este artículo. Lamentablemente el término servicio tiene ambos significados y tenemos que vivir con la polisemia.

<a id="nota_4">[4]</a> Consideramos que una aplicación es una construcción social que une un código base, un grupo de funcionalidad y un cuerpo de financiamiento.

<a id="nota_5">[5]</a> El artículo original se puede encontrar en el sitio web de Melvyn Conway.

<a id="nota_6">[6]</a> No podemos resistirnos a mencionar la declaración de Jim Webber de que ESB significa "Egregious Spaghetti Box".

<a id="nota_7">[7]</a> Netflix hace explícito el enlace - hasta hace poco se refería a su estilo arquitectónico como SOA de grano fino.

<a id="nota_8">[8]</a> En los extremos de escala, las organizaciones a menudo pasan a protocolos binarios - protobufs por ejemplo. Los sistemas que utilizan estos sistemas todavía exhiben la característica de puntos finales inteligentes, comunicaciones simples y transparencia de intercambio por escala. La mayoría de las propiedades web y, sin duda, la gran mayoría de las empresas no necesitan hacer esta compensación - la transparencia puede ser una gran victoria.

<a id="nota_9">[9]</a> "YAGNI" o "You are't going to need it" es un principio de XP y exhortación a no añadir características hasta que sepas que las necesitas.

<a id="nota_10">[10]</a> Es un poco poco insensato de nuestra parte afirmar que los monolitos son un solo lenguaje - para construir sistemas en la web de hoy en día, es probable que necesites saber JavaScript y XHTML, CSS, el lenguaje de tu servidor de elección, SQL y un dialecto ORM. Difícilmente un solo lenguaje, pero sabes a lo que nos referimos.

<a id="nota_11">[11]</a> Adrian Cockcroft menciona específicamente "developer self-service" y "Developers run what they wrote"(sic) en esta excelente presentación realizada en Flowcon en noviembre de 2013.

<a id="nota_12">[12]</a> Estamos siendo un poco insensatos aquí. Obviamente, desplegar más servicios, en topologías más complejas es más difícil que desplegar un único monolito. Afortunadamente, los patrones reducen esta complejidad; sin embargo, la inversión en herramientas sigue siendo una necesidad.

<a id="nota_13">[13]</a> De hecho, Dan North se refiere a este estilo como Arquitectura de Componentes Reemplazables en lugar de microservicios. Puesto que esto parece hablar de un subconjunto de las características, preferimos estas últimas.

<a id="nota_14">[14]</a> Kent Beck destaca esto como uno de sus principios de diseño en Patrones de Implementación.

<a id="nota_15">[15]</a> Y SOA no es la raíz de esta historia. Recuerdo que la gente decía "hemos estado haciendo esto durante años" cuando el término SOA apareció a principios de siglo. Un argumento fue que este estilo ve sus raíces en la forma en que los programas COBOL se comunicaban a través de archivos de datos en los primeros días de la informática empresarial. En otra dirección, se podría argumentar que los microservicios son lo mismo que el modelo de programación de Erlang, pero aplicados a un contexto de aplicación empresarial.
