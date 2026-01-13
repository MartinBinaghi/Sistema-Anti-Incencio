# Sistema Anti-Incendio - Ladder Logic (LD)
## Para KincoBuilder - MK043E-27DT (2 Zonas)

---

## VARIABLES A DECLARAR PRIMERO

Antes de crear el Ladder, debes declarar todas las variables:

### En la tabla de variables (Variable Table):

```
VARIABLES DE ENTRADA (INPUT):
Nombre                    Tipo    Dirección    Comentario
AI_Temp_Sensor1          INT     %IW0         Sensor Temperatura Zona 1
AI_Temp_Sensor2          INT     %IW1         Sensor Temperatura Zona 2
DI_Reset_Button          BOOL    %IX0.0       Botón Reset
DI_Manual_Override       BOOL    %IX0.1       Anulación Manual
DI_System_Enable         BOOL    %IX0.2       Habilitación Sistema
DI_Door_Zone1            BOOL    %IX0.3       Puerta Zona 1
DI_Door_Zone2            BOOL    %IX0.4       Puerta Zona 2
DI_Emergency_Stop        BOOL    %IX0.5       Parada Emergencia

VARIABLES DE SALIDA (OUTPUT):
Nombre                    Tipo    Dirección    Comentario
DO_Alarm_Horn            BOOL    %QX0.0       Sirena
DO_Alarm_Light           BOOL    %QX0.1       Luz Alarma
DO_Sprinkler_Zone1       BOOL    %QX0.2       Rociador Zona 1
DO_Sprinkler_Zone2       BOOL    %QX0.3       Rociador Zona 2
DO_Emergency_Vent        BOOL    %QX0.4       Ventilación
DO_Emergency_Light       BOOL    %QX0.5       Luces Emergencia
DO_Beacon_Red            BOOL    %QX0.6       Baliza Roja

VARIABLES INTERNAS (VAR):
Nombre                    Tipo    Inicial      Comentario
HMI_Temp_Zone1           REAL    0.0          Temperatura Zona 1 en °C
HMI_Temp_Zone2           REAL    0.0          Temperatura Zona 2 en °C
HMI_Temp_Max             REAL    0.0          Temperatura Máxima
HMI_Temp_Avg             REAL    0.0          Temperatura Promedio
HMI_Threshold_Warning    REAL    50.0         Umbral Advertencia
HMI_Threshold_Critical   REAL    70.0         Umbral Crítico
HMI_Delay_Alarm          INT     3            Retardo Alarma (seg)
HMI_Delay_Sprinkler      INT     5            Retardo Rociadores (seg)
HMI_System_Status        INT     0            Estado Sistema
HMI_Fire_Zone1           BOOL    FALSE        Fuego Zona 1
HMI_Fire_Zone2           BOOL    FALSE        Fuego Zona 2
HMI_Alarm_Active         BOOL    FALSE        Alarma Activa
HMI_Sprinklers_Active    BOOL    FALSE        Rociadores Activos
HMI_Event_Counter        INT     0            Contador Eventos
HMI_System_Hours         REAL    0.0          Horas Operación
HMI_BTN_Reset            BOOL    FALSE        Botón Reset HMI
HMI_BTN_Silence_Alarm    BOOL    FALSE        Botón Silenciar HMI
HMI_BTN_Test_System      BOOL    FALSE        Botón Test HMI
HMI_BTN_Manual_Zone1     BOOL    FALSE        Botón Manual Z1
HMI_BTN_Manual_Zone2     BOOL    FALSE        Botón Manual Z2
System_Active            BOOL    FALSE        Sistema Activo
Fire_Detected_Zone1      BOOL    FALSE        Fuego Detectado Z1
Fire_Detected_Zone2      BOOL    FALSE        Fuego Detectado Z2
Any_Fire_Detected        BOOL    FALSE        Cualquier Fuego
Alarm_Warning            BOOL    FALSE        Alarma Advertencia
Alarm_Critical           BOOL    FALSE        Alarma Crítica
Alarm_Latched            BOOL    FALSE        Alarma con Memoria
Test_Mode                BOOL    FALSE        Modo Prueba
Silence_Active           BOOL    FALSE        Silencio Activo
Reset_Trigger            BOOL    FALSE        Disparo Reset
Silence_Trigger          BOOL    FALSE        Disparo Silenciar
Test_Trigger             BOOL    FALSE        Disparo Test
Temp_Raw1                INT     0            ADC Raw Zona 1
Temp_Raw2                INT     0            ADC Raw Zona 2
Temp_Calc_Z1             REAL    0.0          Cálculo Temp Z1
Temp_Calc_Z2             REAL    0.0          Cálculo Temp Z2

TEMPORIZADORES (TON, TP):
TON_Alarm_Delay          TON                  Timer Retardo Alarma
TON_Sprinkler_Delay      TON                  Timer Retardo Rociadores
TP_Alarm_Pulse           TP                   Pulso Alarma
TP_Beacon_Pulse          TP                   Pulso Baliza
TON_Silence_Timer        TON                  Timer Silencio
TON_Test_Timer           TON                  Timer Test
TON_Hour_Counter         TON                  Contador Horas
```

---

## LADDER LOGIC - RUNG POR RUNG

### SECCIÓN 1: CONVERSIÓN DE SEÑALES ANALÓGICAS

**RUNG 1: Conversión ADC a Temperatura Zona 1**
```
Descripción: Convierte señal 4-20mA (819-4095) a 0-100°C

[AI_Temp_Sensor1]--[SUB]--[819]--[DIV]--[3276.0]--[MUL]--[100.0]--[MOV]--[HMI_Temp_Zone1]
```

Instrucciones en KincoBuilder:
1. Agregar bloque **SUB** (Resta)
   - Input 1: AI_Temp_Sensor1
   - Input 2: Constante 819
   - Output: Temp_Calc_Z1

2. Agregar bloque **DIV** (División)
   - Input 1: Temp_Calc_Z1
   - Input 2: Constante 3276.0
   - Output: Temp_Calc_Z1

3. Agregar bloque **MUL** (Multiplicación)
   - Input 1: Temp_Calc_Z1
   - Input 2: Constante 100.0
   - Output: HMI_Temp_Zone1

**RUNG 2: Conversión Zona 2 (igual que Zona 1)**
```
[AI_Temp_Sensor2]--[SUB]--[819]--[DIV]--[3276.0]--[MUL]--[100.0]--[MOV]--[HMI_Temp_Zone2]
```

**RUNG 3: Limitar Temperatura Mínima Zona 1**
```
[HMI_Temp_Zone1]--[<]--[0.0]----[MOV]--[0.0]--[HMI_Temp_Zone1]
```
Si HMI_Temp_Zone1 < 0.0, entonces = 0.0

**RUNG 4: Limitar Temperatura Máxima Zona 1**
```
[HMI_Temp_Zone1]--[>]--[100.0]--[MOV]--[100.0]--[HMI_Temp_Zone1]
```

**RUNG 5 y 6: Repetir límites para Zona 2**

**RUNG 7: Calcular Temperatura Máxima**
```
[HMI_Temp_Zone1]--[>]--[HMI_Temp_Zone2]--[MOV]--[HMI_Temp_Zone1]--[HMI_Temp_Max]
                 |
[HMI_Temp_Zone2]--[<=]--[HMI_Temp_Zone1]--[MOV]--[HMI_Temp_Zone2]--[HMI_Temp_Max]
```

**RUNG 8: Calcular Temperatura Promedio**
```
[HMI_Temp_Zone1]--[ADD]--[HMI_Temp_Zone2]--[DIV]--[2.0]--[MOV]--[HMI_Temp_Avg]
```

---

### SECCIÓN 2: VERIFICACIÓN DEL SISTEMA

**RUNG 9: Sistema Activo**
```
--[DI_System_Enable]--[/DI_Manual_Override]--[/DI_Emergency_Stop]--( System_Active )
```
Contacto normalmente abierto: DI_System_Enable
Contacto normalmente cerrado: DI_Manual_Override (la barra / indica NC)
Contacto normalmente cerrado: DI_Emergency_Stop
Bobina: System_Active

---

### SECCIÓN 3: DETECCIÓN DE FLANCOS (BOTONES)

**RUNG 10: Flanco Reset**
```
--[DI_Reset_Button]--[R_TRIG]--( Reset_Trigger )
  |
--[HMI_BTN_Reset]--[R_TRIG]--|
```
Usar bloque R_TRIG (Rising Edge Trigger)

**RUNG 11: Flanco Silenciar**
```
--[HMI_BTN_Silence_Alarm]--[R_TRIG]--( Silence_Trigger )
```

**RUNG 12: Flanco Test**
```
--[HMI_BTN_Test_System]--[R_TRIG]--( Test_Trigger )
```

---

### SECCIÓN 4: MODO DE PRUEBA

**RUNG 13: Activar Modo Test**
```
--[Test_Trigger]--( S )--Test_Mode
                  |
                  [ADD]--[1]--[HMI_Event_Counter]--( HMI_Event_Counter )
```
S = Set (establecer)

**RUNG 14: Timer Test 10 segundos**
```
--[Test_Mode]--[TON]--( TON_Test_Timer )
               PT: T#10S
```

**RUNG 15: Resetear Test**
```
--[TON_Test_Timer.Q]--( R )--Test_Mode
```
R = Reset

---

### SECCIÓN 5: DETECCIÓN DE INCENDIO

**RUNG 16: Detección Fuego Zona 1**
```
--[System_Active]--[HMI_Temp_Zone1]--[>=]--[HMI_Threshold_Critical]--( Fire_Detected_Zone1 )
                                                                        ( HMI_Fire_Zone1 )
```
Comparador >= (mayor o igual)

**RUNG 17: Detección Fuego Zona 2**
```
--[System_Active]--[HMI_Temp_Zone2]--[>=]--[HMI_Threshold_Critical]--( Fire_Detected_Zone2 )
                                                                        ( HMI_Fire_Zone2 )
```

**RUNG 18: Cualquier Fuego Detectado**
```
--[Fire_Detected_Zone1]--( Any_Fire_Detected )
  |
--[Fire_Detected_Zone2]--|
```

---

### SECCIÓN 6: LÓGICA DE ALARMAS

**RUNG 19: Alarma de Advertencia Zona 1**
```
--[System_Active]--[HMI_Temp_Zone1]--[>=]--[HMI_Threshold_Warning]--( Alarm_Warning )
```

**RUNG 20: Alarma de Advertencia Zona 2**
```
--[System_Active]--[HMI_Temp_Zone2]--[>=]--[HMI_Threshold_Warning]--( Alarm_Warning )
```

**RUNG 21: Alarma Crítica**
```
--[Any_Fire_Detected]--( Alarm_Critical )
  |
--[Test_Mode]----------|
```

**RUNG 22: Memoria de Alarma (Latch)**
```
--[Alarm_Critical]--( S )--Alarm_Latched
                    |
                    [ADD]--[1]--[HMI_Event_Counter]
```

**RUNG 23: Reset de Alarma**
```
--[Reset_Trigger]--[/Alarm_Critical]--( R )--Alarm_Latched
                                       ( R )--Silence_Active
```

**RUNG 24: Estado Alarma Activa**
```
--[Alarm_Warning]---( HMI_Alarm_Active )
  |
--[Alarm_Critical]-|
  |
--[Alarm_Latched]--|
```

---

### SECCIÓN 7: SILENCIAR ALARMA

**RUNG 25: Activar Silencio**
```
--[Silence_Trigger]--( S )--Silence_Active
```

**RUNG 26: Timer Silencio 120 segundos**
```
--[Silence_Active]--[TON]--( TON_Silence_Timer )
                    PT: T#120S
```

**RUNG 27: Desactivar Silencio**
```
--[TON_Silence_Timer.Q]--( R )--Silence_Active
  |
--[Alarm_Critical]------|
```

---

### SECCIÓN 8: TEMPORIZACIÓN

**RUNG 28: Convertir HMI_Delay_Alarm a TIME**
```
--[HMI_Delay_Alarm]--[MUL]--[1000]--[INT_TO_TIME]--( Delay_Alarm_Time )
```
Variable auxiliar: Delay_Alarm_Time (TIME)

**RUNG 29: Timer Retardo Alarma**
```
--[Alarm_Critical]--[TON]--( TON_Alarm_Delay )
                    PT: Delay_Alarm_Time
```

**RUNG 30: Convertir HMI_Delay_Sprinkler**
```
--[HMI_Delay_Sprinkler]--[MUL]--[1000]--[INT_TO_TIME]--( Delay_Sprinkler_Time )
```

**RUNG 31: Timer Retardo Rociadores**
```
--[Alarm_Critical]--[TON]--( TON_Sprinkler_Delay )
                    PT: Delay_Sprinkler_Time
```

**RUNG 32: Pulso Alarma 500ms**
```
--[/TP_Alarm_Pulse.Q]--[TP]--( TP_Alarm_Pulse )
                       PT: T#500MS
```

**RUNG 33: Pulso Baliza 250ms**
```
--[/TP_Beacon_Pulse.Q]--[TP]--( TP_Beacon_Pulse )
                        PT: T#250MS
```

---

### SECCIÓN 9: ACTIVACIÓN DE SIRENA

**RUNG 34: Sirena Continua (Alarma Crítica)**
```
--[TON_Alarm_Delay.Q]--[/Silence_Active]--( DO_Alarm_Horn )
  |
--[Alarm_Latched]--[/Silence_Active]-----|
```

**RUNG 35: Sirena Intermitente (Advertencia)**
```
--[Alarm_Warning]--[/Alarm_Critical]--[TP_Alarm_Pulse.Q]--[/Silence_Active]--( DO_Alarm_Horn )
```

---

### SECCIÓN 10: ACTIVACIÓN DE LUCES

**RUNG 36: Luz de Alarma**
```
--[HMI_Alarm_Active]--( DO_Alarm_Light )
```

**RUNG 37: Baliza Roja Parpadeante**
```
--[Alarm_Critical]--[TP_Beacon_Pulse.Q]--( DO_Beacon_Red )
  |
--[Alarm_Latched]--[TP_Beacon_Pulse.Q]--|
```

---

### SECCIÓN 11: ACTIVACIÓN DE ROCIADORES

**RUNG 38: Rociador Zona 1**
```
--[Fire_Detected_Zone1]--[TON_Sprinkler_Delay.Q]--( DO_Sprinkler_Zone1 )
  |
--[HMI_BTN_Manual_Zone1]--[TON_Sprinkler_Delay.Q]--|
```

**RUNG 39: Rociador Zona 2**
```
--[Fire_Detected_Zone2]--[TON_Sprinkler_Delay.Q]--( DO_Sprinkler_Zone2 )
  |
--[HMI_BTN_Manual_Zone2]--[TON_Sprinkler_Delay.Q]--|
```

**RUNG 40: Estado Rociadores Activos**
```
--[DO_Sprinkler_Zone1]--( HMI_Sprinklers_Active )
  |
--[DO_Sprinkler_Zone2]--|
```

---

### SECCIÓN 12: VENTILACIÓN Y LUCES EMERGENCIA

**RUNG 41: Ventilación de Emergencia**
```
--[Alarm_Critical]--( DO_Emergency_Vent )
  |
--[Alarm_Latched]--|
```

**RUNG 42: Luces de Emergencia**
```
--[HMI_Alarm_Active]--( DO_Emergency_Light )
```

---

### SECCIÓN 13: ESTADO DEL SISTEMA

**RUNG 43: Sistema Apagado (Status = 0)**
```
--[/System_Active]--[MOV]--[0]--( HMI_System_Status )
```

**RUNG 44: Sistema Crítico (Status = 3)**
```
--[System_Active]--[Alarm_Critical]--[MOV]--[3]--( HMI_System_Status )
  |               |
  |             --[Alarm_Latched]--|
```

**RUNG 45: Sistema Advertencia (Status = 2)**
```
--[System_Active]--[Alarm_Warning]--[/Alarm_Critical]--[MOV]--[2]--( HMI_System_Status )
```

**RUNG 46: Sistema Normal (Status = 1)**
```
--[System_Active]--[/Alarm_Warning]--[/Alarm_Critical]--[MOV]--[1]--( HMI_System_Status )
```

---

### SECCIÓN 14: CONTADOR DE HORAS

**RUNG 47: Timer de 1 Hora**
```
--[System_Active]--[TON]--( TON_Hour_Counter )
                   PT: T#3600S
```

**RUNG 48: Incrementar Contador Horas**
```
--[TON_Hour_Counter.Q]--[ADD]--[1.0]--[HMI_System_Hours]--( HMI_System_Hours )
                        |
                        ( R )--TON_Hour_Counter
```

---

## RESUMEN DE BLOQUES A USAR EN KINCOBUILDER

### Contactos:
- **--[ ]--** : Contacto normalmente abierto (NO)
- **--[/]--** : Contacto normalmente cerrado (NC)

### Comparadores:
- **--[>]--** : Mayor que
- **--[<]--** : Menor que
- **--[>=]--** : Mayor o igual
- **--[<=]--** : Menor o igual
- **--[=]--** : Igual

### Funciones Matemáticas:
- **[ADD]** : Suma
- **[SUB]** : Resta
- **[MUL]** : Multiplicación
- **[DIV]** : División
- **[MOV]** : Mover valor

### Bloques Especiales:
- **[TON]** : Timer On Delay
- **[TP]** : Timer Pulse
- **[R_TRIG]** : Rising Edge Trigger (flanco ascendente)
- **( S )** : Set (establecer)
- **( R )** : Reset
- **( )** : Bobina (salida)

### Conversiones:
- **[INT_TO_REAL]** : Convertir entero a real
- **[INT_TO_TIME]** : Convertir entero a tiempo

---

## ORDEN DE IMPLEMENTACIÓN RECOMENDADO

Para no abrumarte, implementa por secciones:

**DÍA 1:**
1. Declarar todas las variables
2. Sección 1: Conversión de temperaturas (Rungs 1-8)
3. Sección 2: Sistema activo (Rung 9)
4. Probar que las temperaturas se leen bien

**DÍA 2:**
5. Sección 5: Detección de incendio (Rungs 16-18)
6. Sección 6: Alarmas (Rungs 19-24)
7. Sección 9: Sirena (Rungs 34-35)
8. Probar detección y alarma

**DÍA 3:**
9. Sección 8: Temporizadores (Rungs 28-33)
10. Sección 11: Rociadores (Rungs 38-40)
11. Sección 12: Ventilación (Rungs 41-42)
12. Probar sistema completo

**DÍA 4:**
13. Secciones restantes
14. Pantalla HMI
15. Pruebas finales

---

## TIPS IMPORTANTES

1. **Guarda frecuentemente** (Ctrl+S)
2. **Compila después de cada sección** para detectar errores temprano
3. **Usa comentarios** en cada rung para saber qué hace
4. **No te agobies** - es normal que tome tiempo
5. **Prueba sección por sección**

---

¿Estás listo para empezar? ¿Por cuál sección quieres que empecemos? Te puedo guiar paso a paso en KincoBuilder.
