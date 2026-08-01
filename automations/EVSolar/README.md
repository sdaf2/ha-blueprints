# Control de Carga de Vehículo Eléctrico (EV Smart Charging)

Gestión inteligente para el control de carga del vehículo eléctrico en Home Assistant integrando cargador **Trydan / Raedian (OCPP)**, API **Tesla**, inversor **GoodWe All-in-One** y batería doméstica.

## 📁 Archivos del Proyecto

* **`ev_charging_helpers.yaml`**: Entidades auxiliares (`input_select`, `input_number`, `input_datetime`, `input_boolean`) para crear controles en Lovelace.
* **`ev_charging_blueprint.yaml`**: Blueprint desacoplada y reutilizable para generar la automatización sin hardcodear IDs.

## 🎛️ Modos de Carga

1. **Apagado**: Desactiva la carga inmediatamente.
2. **Manual**: Carga continua al amperaje fijado por `amperios_carga_manual` (6A a 32A).
3. **Nocturno / Finde**: 
   * **Laborables**: Carga dentro del rango horario configurado (`hora_inicio_nocturna` a `hora_fin_nocturna`, ej. 22:00 a 08:00).
   * **Fin de semana**: Permite carga las 24h.
4. **Solar (Excedentes Monofásicos 230V)**: 
   * Ajusta dinámicamente el amperaje $I = \text{Excedentes W} / 230$ entre 6A y 32A.

## ⏱️ Control Anti-Flapping y Estabilización

Para proteger el cargador y los contactores del coche contra arranques/paradas continuas causadas por nubes o fluctuaciones:

* **Tiempo Mínimo de Carga Continuada (`ev_min_charge_time_min`)**: Una vez iniciada la carga solar, la mantiene durante al menos **X minutos** (defecto 5 min) aunque pase una nube.
* **Tiempo Mínimo de Pausa (`ev_stop_delay_min`)**: Al detenerse la carga, espera al menos **Y minutos** (defecto 3 min) antes de poder volver a encender el cargador.
* **Estabilización de Amperios (`ev_amp_change_delay_sec`)**: Espera **Z segundos** (defecto 60 s) entre variaciones continuas de corriente para evitar sobrecargar la electrónica del cargador/vehículo.

## 🛑 Gestión de Prioridad de Excedentes

* **`ev_prioridad_bloqueada`**: Entidad `input_boolean` u otra entidad externa (ej. termo eléctrico o aerotermia). Si esta entidad está en estado `on`, la automatización pausará la carga solar del EV para priorizar esos consumos domésticos.
* **SoC Batería Casa GoodWe (`ev_solar_min_battery_soc`)**: Solo se derivan excedentes al EV si la batería doméstica está por encima de un porcentaje mínimo configurable (defecto 80%).
