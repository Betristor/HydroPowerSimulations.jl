# [Operation Problem with `HydroPowerSimulations.jl`](@id op_problem)

!!! note
    
    `HydroPowerSimulations.jl` is an extension library of [`PowerSimulations.jl`](https://nrel-sienna.github.io/PowerSimulations.jl/latest/) for modeling hydro units. Users are encouraged to review the [single-step tutorial in `PowerSimulations.jl`](https://nrel-sienna.github.io/PowerSimulations.jl/latest/tutorials/decision_problem/) before this tutorial.

## Load packages

```@repl op_problem
using PowerSystems
using PowerSimulations
using HydroPowerSimulations
using PowerSystemCaseBuilder
using HiGHS # solver
```

## Data

!!! note
    
    `PowerSystemCaseBuilder.jl` is a helper library that makes it easier to reproduce examples in the documentation and tutorials. Normally you would pass your local files to create the system data instead of calling the function `build_system`.
    For more details visit [PowerSystemCaseBuilder README](https://github.com/NREL-Sienna/PowerSystemCaseBuilder.jl/blob/main/README.md)

```@repl op_problem
sys = build_system(PSITestSystems, "c_sys5_hy")
```

With a single [`PowerSystems.HydroDispatch`](@extref):

```@repl op_problem
hy = only(get_components(HydroDispatch, sys))
```

## Decision Model

Setting up the formulations based on [`PowerSimulations.jl`](https://nrel-sienna.github.io/PowerSimulations.jl/latest/formulation_library/Introduction/):

```@repl op_problem
template = ProblemTemplate(PTDFPowerModel)
set_device_model!(template, ThermalStandard, ThermalBasicDispatch)
set_device_model!(template, PowerLoad, StaticPowerLoad)
set_device_model!(template, Line, StaticBranch)
```

but, now we also include the hydro using [`HydroDispatchRunOfRiver`](@ref):

```@repl op_problem
set_device_model!(template, HydroDispatch, HydroDispatchRunOfRiver)
```

With the template properly set-up, we construct, build and solve the optimization problem:

```@repl op_problem
model = DecisionModel(template, sys; optimizer = HiGHS.Optimizer)
build!(model; output_dir = mktempdir())
solve!(model)
```

## Exploring Results

Results can be explored using:

```@repl op_problem
res = OptimizationProblemResults(model)
```

Use [`read_variable`](@ref InfrastructureSystems.Optimization.read_variable) to read in the dispatch variable results for the hydro:

```@repl op_problem
var = read_variable(res, "ActivePowerVariable__HydroDispatch")
```
