# Control de Carga de Vehículo Eléctrico (EV Smart Charging)

Gestión inteligente para el control de carga del vehículo eléctrico en Home Assistant integrando cargador **Trydan / Raedian (OCPP)**, API **Tesla**, inversor **GoodWe All-in-One** y batería doméstica.

## 📁 Archivos del Proyecto

* **`ev_charging_helpers.yaml`**: Entidades auxiliares (`input_select`, `input_number`, `input_datetime`, `input_boolean`) para controlar los modos y límites desde Lovelace.
* **`ev_charging_blueprint.yaml`**: Blueprint desacoplada y reutilizable con la lógica de control avanzada.

## 🎛️ Modos de Carga

1. **Apagado**: Desactiva la carga inmediatamente.
2. **Manual**: Carga continua al amperaje fijado por `amperios_carga_manual` (6A a 32A).
3. **Nocturno / Finde**: 
   * **Laborables**: Carga dentro del rango horario configurado (`hora_inicio_nocturna` a `hora_fin_nocturna`, ej. 22:00 a 08:00).
   * **Fin de semana**: Permite carga las 24h.
4. **Solar (Excedentes Monofásicos 230V)**: 
   * Ajusta dinámicamente el amperaje $I = \text{Excedentes W} / 230$ entre 6A y 32A.

## 👥 Asociación Inteligente de Cables: Coche Integrado vs Coche Invitado (Amigo)

La Blueprint asigna sensores de cable **independientes** para el Wallbox y para cada vehículo:

* **🚗 Coche Integrado (ej. tu Tesla)**: Si se detecta conectado el cable del Coche 1 (`car_1_cable_sensor`), la automatización **actúa sobre el Tesla** (API) dejando el Wallbox abierto a 32A.
* **🤝 Coche Invitado (ej. coche de un amigo)**: Si el sensor del Wallbox (`wallbox_cable_sensor` de Raedian) detecta un coche conectado pero **ninguno de tus coches integrados reporta conexión**, la automatización detecta que es un **Coche Invitado** y **actúa directamente sobre el Wallbox Raedian** (ajustando su amperaje y encendido/apagado para que tu amigo pueda cargar con tus excedentes solares o tarifa nocturna de forma segura).

## ⚡ Protección de Potencia Dinámica (10 kW con Batería/Solar vs 3.5 kW Solo Red)

* **🌞 Con Producción Solar o Carga en Batería GoodWe**: Techo máximo de **10.000 W (10 kW)** combinados (`potencia_maxima_casa_w`).
* **🌙 De Noche o Sin Batería (Solo Red Eléctrica)**: Reduce el techo máximo a **3.500 W (3,5 kW)** (`potencia_maxima_red_w`), limitando la corriente del coche a ~15A para no sobrepasar la potencia contratada de la red ni hacer saltar el automático (ICP).

## ⚖️ Balanceador Multi-Vehículo (3 Estrategias Seleccionables)

Permite elegir cómo distribuir la energía solar cuando hay 2 coches conectados:

1. **Equitativo (50/50)**: Divide la potencia a partes iguales entre ambos coches (respetando siempre el mínimo de 6A por coche).
2. **Prioridad (Coche 1)**: Asigna la máxima potencia posible al Coche 1 y deriva el excedente restante al Coche 2.
3. **Porcentaje Personalizado**: Distribuye la potencia según el porcentaje fijado en `ev_porcentaje_coche_1` (ej. 70% / 30%).

## ⏱️ Control Anti-Flapping y Estabilización

* **Tiempo Mínimo de Carga Continuada (`ev_min_charge_time_min`)**: Mantiene la carga al menos 5 min ante nubes pasajeras.
* **Tiempo Mínimo de Pausa (`ev_stop_delay_min`)**: Requiere 3 min de pausa antes de volver a encender tras un paro.
* **Estabilización de Amperios (`ev_amp_change_delay_sec`)**: Espera 60 s entre variaciones continuas para no sobrecargar el vehículo.
