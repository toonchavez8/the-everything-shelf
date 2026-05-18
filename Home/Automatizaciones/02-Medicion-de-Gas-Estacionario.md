# Propuesta - Sistema IoT para Medicion de Gas en Tanque Estacionario

## Objetivo

Disenar un sistema IoT para conocer el nivel de gas disponible en un tanque estacionario domestico, visualizarlo en tiempo real, generar alertas de bajo nivel y construir historicos de consumo para anticipar recargas y detectar anomalias.

La idea es que el sistema sea similar en filosofia al proyecto de medicion de agua:

- Nodo de sensado
- Microcontrolador con conectividad
- Integracion con Home Assistant
- Dashboard local y remoto
- Alertas y analitica por etapas

---

## Referencia de tanque objetivo

Ejemplo de referencia:

- [Mercado Libre - Tanque estacionario Cytsa vacio para gas 120 l](https://www.mercadolibre.com.mx/tanque-estacionario-cytsa-vacio-para-gas-120l/up/MLMU529679784)

Datos observados en referencias publicas recientes:

- Capacidad nominal: `120 L`
- Tipo de gas reportado: `GLP`


---

## Principio correcto para medir el nivel de gas

### Lo que si conviene medir

En un tanque estacionario domestico, la manera mas practica y segura de obtener el nivel es leer el indicador mecanico existente de porcentaje, normalmente acoplado a un flotador interno.

Eso permite:

- No perforar el tanque
- No abrir la linea de gas
- No modificar el recipiente a presion
- Reutilizar un mecanismo ya pensado para indicar nivel

### Lo que no conviene usar como metodo principal

#### Presion del gas

No es buena variable para estimar nivel de GLP en uso normal.

Razones:

- La presion depende mucho de la temperatura
- Mientras exista fase liquida, la presion puede verse "normal" aunque el tanque tenga poco producto
- Sirve mas para diagnostico de condiciones anormales que para medir inventario real

#### Sensor ultrasónico externo al tanque

No es una opcion realista para medir el nivel interno de un tanque presurizado ya instalado.

Razones:

- No hay linea de vista directa hacia la superficie del liquido
- El tanque metalico refleja y bloquea la medicion
- Cualquier modificacion fisica al recipiente es indeseable y riesgosa

#### Celda de carga debajo del tanque

Es tecnicamente posible pero generalmente poco practica en instalaciones existentes.

Razones:

- Requiere modificar la base estructural
- Exige una mecanica muy estable
- Es mas compleja de calibrar
- Puede ser costosa para uso domestico

### Aclaracion importante sobre "sensor ultrasonico"

Aqui hay dos conceptos distintos:

- `Ultrasonico de hobby por aire`: como los usados para agua o distancia abierta. Ese no sirve para medir el interior de un tanque estacionario metalico cerrado.
- `Ultrasonico industrial no invasivo`: sensor externo especializado que lee a traves de la pared del tanque. Ese si puede ser una alternativa real.

Conclusión:

- un `JSN-SR04T`, `HC-SR04` o equivalente no es buena solucion para gas estacionario
- un sensor industrial externo de nivel si puede ser viable, pero ya entra en otra liga de costo e instalacion

---

## Opcion recomendada para MVP

### Lectura no invasiva del medidor mecanico del tanque

La mejor idea para una primera version es leer externamente el indicador de porcentaje del tanque, sin intervenir la parte presurizada.

Hay tres caminos viables:

#### Opcion A - Lectura optica del indicador

Idea:

- Colocar una pequena camara o sensor optico apuntando al dial del tanque
- Procesar la imagen para inferir la posicion de la aguja

Ventajas:

- Cero contacto con el sistema de gas
- Muy flexible
- Facil de validar visualmente

Desventajas:

- Mas sensible a suciedad, reflejos y luz ambiente
- Requiere mas procesamiento si se quiere vision robusta

#### Opcion B - Sensor magnetico / Hall sobre el dial

Idea:

- Detectar el cambio de campo magnetico o angulo asociado al indicador
- Leer la posicion con un sensor magnetico montado externamente

Ventajas:

- Solucion compacta
- Muy bajo consumo
- Mas elegante para integracion fija

Desventajas:

- Requiere caracterizar muy bien como se mueve el indicador real
- No todos los medidores se comportan igual
- Puede requerir desarrollo mecanico fino del soporte

#### Opcion C - Camara + OCR / vision simple

Idea:

- Usar una `ESP32-CAM` o una camara conectada a `Raspberry Pi`
- Leer directamente la escala del indicador

Ventajas:

- Facil de iterar
- Permite evidencia visual y depuracion
- Muy util para prototipado

Desventajas:

- Mayor complejidad de software
- Requiere cuidar iluminacion y limpieza

### Recomendacion practica

Para un desarrollo casero y seguro:

- `MVP mas rapido`: lectura optica del dial
- `Version refinada`: lectura magnetica externa del indicador

---

## Escenario alternativo: no hay medidor disponible

Si asumimos que el medidor fue robado, ya no existe o no se puede aprovechar, entonces el sistema ya no puede basarse en leer una aguja. En ese caso, la prioridad pasa a ser:

- no invadir el tanque
- no depender de la presion como variable de nivel
- mantener una arquitectura IoT razonable
- balancear costo, seguridad y precision

### Ruta 1 - Reponer el medidor con tecnico LP

Idea:

- Verificar si el mecanismo interno sigue existiendo
- Pedir a tecnico o proveedor certificado que reinstale el dial o medidor compatible

Ventajas:

- Suele ser la solucion mas simple y confiable
- Aprovecha el sistema original del tanque
- Permite despues agregar lectura remota

Desventajas:

- Depende de que el mecanismo interno siga utilizable
- Requiere intervencion profesional

Cuándo conviene:

- Es la primera ruta que vale la pena revisar antes de construir otra cosa

### Ruta 2 - Sensor ultrasonico industrial no invasivo

Idea:

- Instalar un sensor externo especializado que mida a traves de la pared metalica del tanque

Ventajas:

- No perfora el tanque
- No abre la linea de gas
- Permite monitoreo continuo
- Tiene sentido para una solucion IoT seria

Desventajas:

- Es la ruta mas cara de electronica
- Requiere validar material, espesor y punto de montaje
- La precision depende mucho de una buena instalacion

Cuándo conviene:

- Cuando no existe medidor util y se quiere una solucion estable de largo plazo

### Ruta 3 - Mapa termico externo

Idea:

- Colocar varios sensores de temperatura en una linea vertical por fuera del tanque
- Inferir la frontera liquido-vapor por comportamiento termico

Ventajas:

- Muy barata
- Facil de prototipar con `ESP32`
- No invasiva

Desventajas:

- Precision baja o media
- Muy sensible a sol, viento, lluvia y temperatura ambiente
- Necesita filtrado y calibracion

Cuándo conviene:

- Cuando quieres un prototipo casero rapido
- Cuando aceptas una estimacion aproximada y no una medicion comercial

### Ruta 4 - Celdas de carga en la base

Idea:

- Medir peso total del tanque y estimar inventario de gas por masa

Ventajas:

- Tiene una base fisica solida
- No depende del comportamiento de la presion
- Puede ser precisa si la mecanica es buena

Desventajas:

- Requiere intervenir o redisenar la base
- Es compleja mecanicamente
- Puede salir cara en instalacion real

Cuándo conviene:

- Cuando vas a rehacer soporte o base del tanque
- Cuando aceptas un proyecto mas estructural que electronico

### Comparativa rapida

|Ruta|Costo relativo|Dificultad|Precision esperada|Riesgo tecnico|Comentario|
|---|---|---|---|---|---|
|Reponer medidor con tecnico|Bajo a medio|Baja a media|Media a buena|Bajo|Primera opcion a investigar|
|Ultrasonico industrial externo|Alto|Media|Buena|Medio|Mejor ruta sin medidor aprovechable|
|Mapa termico con DS18B20|Bajo|Media|Baja a media|Bajo|Buen MVP experimental|
|Celdas de carga en base|Medio a alto|Alta|Media a buena|Medio a alto|Buena fisica, mala practicidad en existente|

### Ranking pragmatico

1. Ver si un tecnico puede reponer el medidor.
2. Si no se puede, evaluar sensor ultrasonico industrial externo.
3. Si quieres empezar barato y aprender, hacer mapa termico con `ESP32`.
4. Si vas a rehacer la base, considerar celdas de carga.

### Lo que no recomendaria

- Medir por presion
- Abrir el tanque por cuenta propia
- Intentar reutilizar sensores de agua por aire para mirar "a traves" del tanque

---

## Recomendacion por presupuesto

### Presupuesto bajo - prototipo experimental

Arquitectura sugerida:

- `ESP32`
- `6` a `10` sensores `DS18B20`
- `1` sensor de temperatura ambiente
- Home Assistant para historicos y modelo de inferencia

Objetivo:

- Detectar tendencia
- Construir una primera heuristica de nivel
- Validar si el enfoque termico sirve en tu instalacion real

Esperado:

- Sistema orientativo
- No precision comercial

### Presupuesto medio/alto - solucion seria

Arquitectura sugerida:

- Sensor ultrasonico industrial no invasivo
- Fuente y gabinete adecuados para exterior
- Gateway o integracion industrial hacia Home Assistant

Objetivo:

- Monitoreo continuo mas confiable
- Alertas y tendencia historica de mejor calidad

Esperado:

- La mejor opcion tecnica cuando no hay medidor mecanico aprovechable

### Presupuesto mecanico - proyecto estructural

Arquitectura sugerida:

- Base instrumentada
- Load cells
- `HX711` o ADC equivalente
- `ESP32`

Objetivo:

- Estimar inventario por masa total

Esperado:

- Buena precision potencial
- Mucha mas complejidad de montaje

---

## Arquitecturas IoT sugeridas para estas rutas

### Opcion A - Mapa termico externo

```text
Tanque estacionario
   ↓
Sensores de temperatura externos
   ↓
ESP32
   ↓ WiFi
Home Assistant
   ↓
Inferencia de nivel / dashboard / alertas
```

### Opcion B - Ultrasonico industrial externo

```text
Tanque estacionario
   ↓
Sensor externo no invasivo
   ↓
Gateway / interfaz industrial
   ↓
Home Assistant / MQTT / backend
   ↓
Dashboard / alertas / historicos
```

### Opcion C - Base con celdas de carga

```text
Tanque estacionario
   ↓
Base instrumentada con load cells
   ↓
HX711 / ADC
   ↓
ESP32
   ↓ WiFi
Home Assistant
   ↓
Dashboard / alertas / historicos
```

---

## Recomendacion concreta para tu caso

Si lo aterrizo a un escenario domestico real en Mexico, con presupuesto controlado y sin medidor disponible:

- `Ruta recomendada para experimentar`: mapa termico externo con `ESP32 + DS18B20`
- `Ruta recomendada para una solucion seria`: sensor ultrasonico industrial externo no invasivo
- `Ruta a validar antes que todo`: preguntar si un tecnico puede reponer el medidor sin cambiar todo el sistema

Mi juicio tecnico:

- si quieres empezar ya y aprender, ve por el mapa termico
- si quieres una solucion confiable de largo plazo, no improvisaria mas alla de reponer medidor o usar sensor industrial externo

---

## Arquitectura general propuesta

```text
Tanque estacionario
   ↓
Medidor mecanico de porcentaje
   ↓
Lector externo no invasivo
   ↓
ESP32 / ESP32-CAM / Raspberry Pi
   ↓ WiFi
Home Assistant / MQTT
   ↓
Dashboard / Alertas / Historicos
```

---

## Hardware propuesto por enfoque

### Enfoque 1 - Optico simple

Componentes sugeridos:

- `ESP32-CAM` o camara USB con `Raspberry Pi`
- Iluminacion LED controlada y suave
- Soporte impreso en 3D o abrazadera no invasiva
- Caja para proteger la camara del sol, polvo y lluvia
- Fuente de alimentacion estable

Funcion:

- Capturar imagen del dial cada cierto tiempo
- Detectar angulo de la aguja o zona marcada
- Convertir ese angulo a porcentaje de llenado

### Enfoque 2 - Magnetico

Componentes sugeridos:

- `ESP32`
- Sensor Hall lineal o angular
- Soporte fijo alineado con el indicador
- Caja sellada para electronica
- Fuente estable

Funcion:

- Medir variacion magnetica o posicion relativa del indicador
- Calibrar valores minimos y maximos
- Convertir la senal a porcentaje de llenado

### Enfoque 3 - Sistema mixto

Componentes sugeridos:

- `ESP32` para adquisicion basica
- `Raspberry Pi` si se requiere vision, procesamiento o dashboard local
- Pantalla local para centro de monitoreo

Funcion:

- Unir sensado, historicos y visualizacion en una plataforma mas completa

---

## Variables utiles a mostrar

### Medicion primaria

- Porcentaje de llenado del tanque
- Estado de la ultima lectura
- Hora de la ultima actualizacion

### Variables derivadas

- Litros estimados disponibles
- Consumo diario estimado
- Consumo semanal y mensual
- Dias restantes estimados
- Fecha probable de recarga

### Alertas

- Nivel bajo
- Nivel critico
- Cambio brusco de nivel
- Falta de actualizacion del sensor
- Posible fuga o consumo anormal

---

## Conversion de porcentaje a litros

Para un tanque de referencia de `120 L`, una aproximacion inicial seria:

```text
litros_estimados = capacidad_nominal_litros * (porcentaje / 100)
```

Ejemplo:

- `25 %` -> `30 L`
- `50 %` -> `60 L`
- `80 %` -> `96 L`

Importante:

- Esta formula sirve como aproximacion operativa
- Debe calibrarse con datos reales de carga y consumo
- En tanques de gas, el indicador mecanico puede ser orientativo y no necesariamente un instrumento de precision comercial

---

## Etapas de desarrollo recomendadas

### Etapa 1 - MVP seguro

Objetivo:

- Ver porcentaje y recibir alertas

Alcance:

- Lectura del medidor existente
- Publicacion a Home Assistant
- Dashboard simple
- Alerta por nivel bajo

### Etapa 2 - Sistema util para operacion domestica

Objetivo:

- Entender consumo y planear recargas

Funciones:

- Historicos diarios
- Tendencia semanal
- Estimacion de dias restantes
- Registro de recargas
- Grafica de consumo por temporada

### Etapa 3 - Diagnostico y anomalias

Objetivo:

- Detectar problemas reales y afinar modelo

Funciones:

- Deteccion de consumo nocturno anormal
- Cambios bruscos de nivel
- Cruce con temperatura ambiente
- Modelos de prediccion por estacionalidad
- Deteccion de periodos sin uso

### Etapa 4 - Plataforma domestica integrada

Objetivo:

- Integrar agua, gas y otros recursos del hogar

Funciones:

- Dashboard unificado de servicios
- Priorizacion de recargas
- Notificaciones consolidada
- Reportes mensuales del hogar
- Analitica de costos por recurso

---

## Dashboard recomendado

Vista principal:

- `% de gas disponible`
- `litros estimados`
- `dias restantes`
- `ultima lectura`
- `estado del nodo`

Vistas secundarias:

- Consumo diario
- Tendencia de 30 dias
- Historial de recargas
- Alarmas y eventos
- Diagnostico del sensor

Centro de visualizacion local:

- Puede reutilizar la misma idea de pantalla local para Home Assistant
- Si ya existe un panel mural para agua, el gas puede integrarse ahi mismo
- Lo ideal es no construir dos dashboards aislados sino uno solo de recursos del hogar

---

## Especificaciones tecnicas sugeridas del nodo

### Si eliges lectura magnetica

- Microcontrolador: `ESP32`
- Alimentacion: `5 VDC`
- Logica: `3.3 V`
- Conectividad: `WiFi 2.4 GHz`
- Sensor: `Hall lineal` o `sensor angular magnetico`
- Muestreo recomendado: cada `1` a `5` minutos
- Publicacion: `MQTT` o `ESPHome`

### Si eliges lectura optica

- Controlador: `ESP32-CAM` para prototipo o `Raspberry Pi` para vision mas robusta
- Iluminacion integrada y difusa
- Captura periodica: cada `5` a `15` minutos
- Procesamiento: deteccion de aguja, OCR o clasificacion por imagen

### Entidades sugeridas en Home Assistant

- `sensor.gas_porcentaje`
- `sensor.gas_litros_estimados`
- `sensor.gas_dias_restantes`
- `binary_sensor.gas_nivel_bajo`
- `binary_sensor.gas_nivel_critico`
- `sensor.gas_ultima_actualizacion`
- `sensor.gas_consumo_promedio_diario`

---

## Consideraciones de seguridad

- No perforar, soldar ni modificar el tanque
- No intervenir valvulas, lineas o accesorios presurizados sin tecnico certificado
- No instalar electronica improvisada pegada a puntos de posible fuga
- Mantener la conversion `AC/DC` lejos de la zona del tanque cuando sea posible
- Usar gabinetes, fijaciones y cableado adecuados para intemperie
- Priorizar lectura externa y no invasiva del indicador existente

---

## Recomendacion final

Si quieres construir un sistema IoT de gas con buen balance entre seguridad, complejidad y utilidad, el camino mas razonable es:

1. Leer externamente el indicador mecanico del tanque
2. Publicar el porcentaje a Home Assistant
3. Convertir ese porcentaje en litros y autonomia estimada
4. Integrarlo en el mismo dashboard donde se vera el sistema de agua

La peor decision tecnica para este caso seria intentar medir el nivel por presion o modificar fisicamente el tanque para meter un sensor interno.
