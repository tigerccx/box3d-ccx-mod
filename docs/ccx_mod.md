# CCX Mod

## Velocity setters with wake control

`b3Body_SetLinearVelocityWithWake` and `b3Body_SetAngularVelocityWithWake` add a `bool wake` parameter:

- `wake=true`: a nonzero velocity wakes a sleeping body; angular motion locks are applied before deciding whether to wake.
- `wake=false`: only awake bodies are modified; sleeping bodies do not store a pending velocity.
- Static and disabled bodies are ignored. The original setters retain their wake behavior.

C# `Body` and `Box3DBody` expose `SetLinearVelocity(value, bool wake = true)` and
`SetAngularVelocity(value, bool wake = true)`. Property setters retain their wake behavior.

## Callback before position integration

`b3World_SetPreIntegratePositionsCallback` runs serially on the solver orchestrator for each
awake body in every sub-step, after constraint solving and before position integration.
It receives the body ID, read-only position/rotation deltas and the sub-step duration.
Deltas contain changes accumulated by preceding sub-steps; stored body transforms exclude them.

Read velocity directly from the body. The callback may use the wake-controlled setters above
and linear/angular impulse APIs on the current body with `wake=false`.
Do not access other bodies, change world structure, or call other mutating APIs.
Use impulses for an immediate response; force integration has already run for this sub-step.

Position integration still applies motion locks and speed limits, and constraint relaxation
may change velocity again. Large changes can increase contact or joint error.

Rebuild native and managed code together when changing the callback signature.
C# delta parameters use `in`; the dispatcher does not write velocity or deltas back.
