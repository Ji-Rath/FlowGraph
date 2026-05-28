# FlowGraph

Node editor for scripting game flow in Unreal Engine.

## Capabilities

| Capability | Description |
|------------|-------------|
| **Event-based Nodes** | Each node is a UObject, encapsulating logic with data |
| **Async/Latent by Design** | Nodes wait for events then trigger output pins |
| **Custom Pin Definitions** | Every node defines its own input/output pins |
| **Debug Visualization** | Editor displays debug info on nodes and wires |
| **World Partition Support** | Compatible with UE 5.0+ World Partition |
| **Gameplay Tag Integration** | Obtain actors using Gameplay Tags for loose coupling |

## Key Classes

| Class | Purpose |
|-------|---------|
| `UFlowNode` | Base class for all Flow nodes |
| `UFlowAsset` | Flow graph asset containing nodes |
| `UFlowSubsystem` | Central subsystem for Flow management |
| `UFlowComponent` | Component for actors to interact with Flow |
| `UFlowGraph` | Graph editor for Flow assets |
| `UFlowGraphNode` | Graph node representation |

## Key Structs & Interfaces

| Name | Type | Purpose |
|------|------|---------|
| `FFlowPin` | Struct | Input/output pin definition |
| `FFlowConnection` | Struct | Connection between pins |
| `FFlowEvent` | Struct | Event data for node communication |
| `IFlowInterface` | Interface | `OnFlowEvent()` receiver |
| `IFlowComponent` | Interface | `RegisterWithFlow()` registration |

## Common Pitfalls

- Forgetting to call `TriggerOutput()` — flow hangs
- Using direct actor references — breaks World Partition
- Node class renames without asset updates — breaks graphs
- Not implementing `Cleanup()` — leaks resources/delegates
- Missing PreDefault dependencies — load order matters

See `.agents/patterns.md` for implementation patterns.

## Integration Points

- Used by: SUQSFlow (quest nodes), HorrorFeatures (narrative nodes)
- See `.agents/plugin-integration.md` for cross-plugin dependency matrix

## Human Review Required

- Changes to `UFlowNode` base class
- Changes to `UFlowAsset` or serialization format
- Adding new pin types or connection policies
- See `.agents/human-review-checklist.md` for full list