# Lab 5 Integration and SOA - Project Report
# David Borrel Seral - 871643

## 1. EIP Diagram (Before)

![Before Diagram](diagrams/before.png)

Describe what the starter code does and what problems you noticed.

### Componentes principales:
- Un ``integerSource()`` que generaba números secuenciales (0, 1, 2, 3...)
- Un ``flujo myFlow()`` que obtenía números del source cada 100ms
- Un canal ``evenChannel`` (publish-subscribe) para números pares
- Un flujo ``evenFlow()`` que procesaba los números pares
- Un flujo ``oddFlow()`` que procesaba números impares con un filtro incorrecto
- Un ``SomeService`` que escuchaba del canal oddChannel
- Un gateway ``SendNumber`` que inyectaba números negativos cada segundo

### Problemas detectados:
- Falta del canal central (``numberChannel``). El código original no tenía un canal para enrutar números a pares o impares.
- El gateway SendNumber tenía requestChannel = ``evenChannel``, enviando números negativos (que son impares) al canal de números pares.
- El canal ``oddChannel`` no publicaba a múltiples suscriptores y no estaba definido el bean.
- Filtro incorrecto en ``oddFlow()``: aceptaba números pares en lugar de impares. Además este filtro no estaba en la versión final del diagrama.

---

## 2. What Was Wrong

Explain the bugs you found in the starter code:

### What was wrong in the starter code?
1. **Ausencia del canal ``numberChannel``:** No existía un canal central para recibir todos los números antes del enrutamiento.
2. **Gateway enviando a canal incorrecto:** ``SendNumber`` enviaba a ``evenChannel`` en lugar de al canal central, causando que números negativos (impares) fueran procesados como pares.
3. **Canal ``oddChannel`` no era publish-subscribe:** Al no estar definido como ``PublishSubscribeChannel``, los mensajes no llegaban a todos los suscriptores.
4. **Filtro con lógica invertida:** En ``oddFlow``, el filtro aceptaba números pares (p % 2 == 0) en lugar de impares, rechazando exactamente los mensajes que debería procesar. Además este filtro no estaba en la versión final del diagrama.

### Why did those issues prevent correct behavior?
1. **Falta de ``numberChannel``:** Sin un punto de entrada central, el flujo no podía manejar múltiples fuentes de mensajes de manera correcta.
2. **Gateway mal configurado:** Los números negativos del gateway iban directamente a ``evenChannel``, saltándose el router y fallando su procesamiento como números pares.
3. **Falta de publish-subscribe:** Sin este patrón, los mensajes impares iban alternativamente a oddFlow O a SomeService, pero no a ambos simultáneamente.
4. **Filtro invertido:** Los números impares (1, 3, 5...) eran rechazados por el filtro, nunca llegando al transformador ni al handler. Solo algunos mensajes llegaban al SomeService debido al load-balancing.

### What code did you modify? 
1. **Añadido ``numberChannel``:** He creado este canal que sirve como punto de entrada central para todos los números.
2. **Actualizado el gateway ``SendNumber``:** Cambié el requestChannel a ``numberChannel`` para asegurar que todos los números pasen por el router.
3. **Definido ``oddChannel`` como Publish-Subscribe:** He añadido la definición del canal para permitir múltiples suscriptores.
4. **Eliminado el filtro en ``oddFlow()``:** Dado que el enrutamiento ya separa los números impares, el filtro era innecesario.

---

## 3. What You Learned

### What you learned about Enterprise Integration Patterns
He apredido que los **Enterprise Integration Patterns** son soluciones ya probadas para problemas comunes en la integración de sistemas. Estos patrones proporcionan una guía estructurada para diseñar flujos de mensajes robustos y escalables, facilitando la comunicación entre diferentes componentes de software.
### How Spring Integration works
Spring Integration es un framework que implementa los principios de los EIP, permitiendo construir aplicaciones basadas en mensajes. Este framework proporciona una variedad de componentes predefinidos, como canales, filtros, transformadores y routers, que facilitan la creación de flujos de integración.
### What was challenging and how you solved it
Lo más complicado de esta práctica de laboratorio fue realizar primero el diagrama con la primera versión del código, a la par que identificar los errores en el código original. La clave para solucionar esto fue entender bien el comportamiento esperado del programa. Realizando una primera versión con papel y boli y luego comparándola con el código, ya se hizo más sencillo comprender el código y diseñar el diagrama correctamente.

---

## 4. AI Disclosure

**Did you use AI tools?** (ChatGPT, Copilot, Claude, etc.)

- He usado Claude para entender alguna parte del código, concretamente para alguna función de Spring Integration que no conocía bien, para poder entender bien su comportamiento.

---

## Additional Notes

Esta sesión de laboratorio ha sido muy útil para comprender mejor los patrones de integración y cómo aplicarlos utilizando Spring Integration. La experiencia de analizar primero el código y realizar un diagrama de flujo de este, creo que es muy efectiva para entender bien el funcionamiento del código, ya que después ha sido mucho más sencillo identificar los errores y corregirlos. Me gustaría destacar la utilidad de los diagramas para facilitar la comprensión del código.
