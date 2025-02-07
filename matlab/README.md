
## System 

Refer to the readme on the upper folder.

## Simulink models

### *CoolingSystem*, *CoolingSystemR2024a* and *CoolingSystemR2023a*

Original model, made by mmanngard.

### *CoolingSystemR2024a_modified* and *CoolingSystemR2024a_modified_IO*

Modified versions of the Simulink model of CoolingSystem that can be exported as an FMU.

Changes made:
- Converted the PI-controller to discrete time
- Added Unit Delays block between m_out - mdot AND inside the mixing node between (u(1)+u(3)) - MUX
- Changed the solver to fixed time ode1be with a step-size of 0.001
- Added variable for the temperature setpoint

For "CoolingSystemR2024a_modified_IO" in addittion:
- Fixed issue, where radiator took engines Mass
- Routed all the inputs and outputs for the FMU

With these changes the model has been "verified" to work similar to the original model.

### *CoolingSystemR2024a_Control* and *CoolingSystemR2024a_woControl*

The physical system and the control system separated. Same as with the LOC-model. 

## FMUs

The FMU files has been checked with FMU checker and tested with FMPY.gui. All the FMUs contains both Windows and Linux binaries.

The FMU folder contains:

:warning: **NOTE** These are exported with MATLABs ow export tool, will not work with Appros. FMIkit versions to be done.

### *CoolingSystem.FMU*
- Made from "CoolingSystemR2024a_modified_IO" model
- Full model that contains everything
- FMU tested on simulink, providing similar results than the model
- The model_description structure is a little bit weird and needs some work on the models side

**Inputs, Outputs and parameters**
- Inputs:
    - `T_in`: Incoming temperature (cold water?) 
    - `Setpoint_temp`: Setpoint sent to controller
- Parameters: (Currently not exported?, mass of components ->) 
    - M_R
    - M_E
    - M_B
    - M_MIX
- Outputs: (Values are from circulating water, expect control signals) 
    - `T_R`: Outcoming temperature from radiator - Engine  -> goes to the PI-controller
    - `T_E`: Outcoming temperature from engine - Boiler
    - `T_B`: Outcoming temperature from boiler - Out/mixing (T_B and T_out is the same in the model??)
    - `T_out`: Outcoming temperature from system -> goes to the relay
    - `T`: Outcoming tmperature from Mixing node - Radiator
    - `mdot`: Massflow from mixing node to radiator
    - `mdot_r`: Massflow from valve to mixing
    - `mdot_out`: Massflow from valve to mixing
    - `u_V`: Relay Control signal, opens the valve (0 or 1) (default 0, when T_out > 125 -> 1)
    - `u_R`: Radiator control signal, PI-controller

:warning: **NOTE** To be validated 

### *CoolingSystem_minimal.FMU*
- Same as above, just with fewer outputs, only the ones moving out of the system

**Inputs, Outputs and parameters**
- Inputs:
    - `T_in`: Incoming temperature (cold water?) 
    - `Setpoint_temp`: Setpoint sent to controller
- Parameters: (Currently not exported?, mass of components ->) 
    - M_R
    - M_E
    - M_B
    - M_MIX
- Outputs: (Values are from circulating water, expect control signals) 
    - `T_out`: Outcoming temperature from system -> goes to the relay
    - `mdot_out`: Massflow from valve to mixing
    - `u_V`: Relay Control signal, opens the valve (0 or 1) (default 0, when T_out > 125 -> 1)
    - `u_R`: Radiator control signal, PI-controller

:warning: **NOTE** To be validated 
    
### *CoolingSystem_woControl.FMU*
- The system without controller
- Tested only with FMU checker and FMPY.gui that it runs
- Made from "CoolingSystemR2024a_modified_IO" model

### *CoolingSystem_PIcontrol.FMU*
- The original PI-controller, including the relay
- changed the output T_out in original model to T_in to describe it is an input for the controller
- Tested only with FMU checker and FMPY.gui that it works as expected with input values
- Made from "CoolingSystemR2024a_modified_IO" model

### FMU/Components
- Folder contains all the components of the cooling system exported as FMUs
- Component description and connections can be found on the readme at the folder





