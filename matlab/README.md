
## System 

Refer to the readme on the upper folder.

## Simulink models

### "CoolingSystem", "CoolingSystemR2024a" and "CoolingSystemR2023a"

Original model, made by mmanngard.

### "CoolingSystemR2024a_modified" and "...modified_IO"

Modified versions of the Simulink model of CoolingSystem that can be exported as an FMU.

Changes made:
- Converted the PI-controller to discrete time
- Added Unit Delays block between m_out - mdot AND inside the mixing node between (u(1)+u(3)) - MUX
- Changed the solver to fixed time ode1be with a step-size of 0.001
- Added variable for the temperature setpoint

For "...modified_IO" in addittion:
- Fixed issue, where radiator took engines Mass
- Routed all the inputs and outputs for the FMU

With these changes the model has been "verified" to work similar to the original model.

## FMUs

The FMU files has been checked with FMU checker and tested with FMPY.gui without any input values. All the FMUs contains both Windows and Linux binaries.

The FMU folder contains:

**CoolingSystem.FMU**
- Made from "CoolingSystemR2024a_modified_IO" model
- Full model that contains everything
- FMU tested on simulink, providing similar results than the model
- The model_description structure is a little bit weird and needs some work on the models side

*** Inputs, Outputs and parameters ***
- Inputs:
    - xxx
    - xxx
- Parameters: 
    - xxx
    - xxx
    - xxx
    - xxx
- Outputs:
    - xxx

**Components**
- Component description can be found on the readme at the folder





