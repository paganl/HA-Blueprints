External Sensor Climate Controller
This package contains a Home Assistant automation blueprint for controlling an
air-conditioner/heat pump whose internal room-temperature sensor is not trusted.
Control model
- Virtual climate entity: what you use on the dashboard.
- External temperature sensor: authoritative current room temperature.
- Physical Samsung climate entity: actuator only.
- SmartThings power/energy sensor: optional observability; deliberately not
  part of the demand decision.
Heating
If the external temperature falls below:

virtual target - start deadband

the physical Samsung is set to:
- HVAC mode: heat
- target: the physical entity's max_temp (normally 30 °C)
When the external temperature reaches:

virtual target + stop deadband

the Samsung is switched off, subject to the configured minimum run time.
Cooling
If the external temperature rises above:

virtual target + start deadband

the physical Samsung is set to:
- HVAC mode: cool
- target: the physical entity's min_temp (normally 16 °C)
When the external temperature reaches:

virtual target - stop deadband

the Samsung is switched off, subject to the configured minimum run time.
Deliberate behaviour
1. The Samsung's internal temperature is never used for control.
2. A HEAT ↔ COOL reversal always passes through OFF.
3. The minimum-off timer protects the unit before restart.
4. If the physical target is manually changed while demand exists, the blueprint
   repairs it back to MAX for heating or MIN for cooling.
5. If the external sensor or virtual target becomes unavailable, the default
   fail-safe is to turn the physical unit off.
6. A once-per-minute reconciliation trigger ensures timers complete even when no
   sensor value happens to change.
Suggested initial settings
- Start deadband: 0.3 °C
- Stop deadband: 0.0 °C
- Minimum run time: 180 s
- Minimum off time: 180 s
These are conservative starting points, not Samsung-specific requirements. The
air-conditioner has its own compressor protection as well; tune them after
watching actual room behaviour.
Installation
Copy:

external_sensor_climate_controller.yaml

to:

/config/blueprints/automation/paga/external_sensor_climate_controller.yaml

Then reload automations/blueprints and create an automation from the blueprint.
Virtual climate entity
Use a Home Assistant Template Climate entity as the dashboard-facing
thermostat. It should expose:
- current temperature from your external sensor;
- user target temperature;
- HVAC modes off, heat, cool.
The automation blueprint reads only the virtual climate's mode and target, so it
does not care how that virtual climate entity is implemented.

Current Home Assistant versions support Template Climate directly. Once the exact
Samsung, external-temperature and SmartThings power entity IDs are known, the
virtual climate can be configured so its hvac_action uses the power reading to
show heating, cooling, idle, or off.
Power monitoring
Do not use SmartThings power as the thermostat's demand signal. Use it to answer a
different question: "is the physical unit actually doing work?"

That gives useful diagnostics without making cloud telemetry a dependency of the
temperature-control loop.
