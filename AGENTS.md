# AGENTS.md

This file provides guidance to agents when working with the FlowGraph plugin.

## 1. Plugin Overview

**FlowGraph** is a design-agnostic node editor for scripting game flow in Unreal Engine. It provides a graph editor tailored for scripting the flow of events in virtual worlds, based on a decade of experience with implementing narrative in video games.

### Key Capabilities

| Capability | Description |
|------------|-------------|
| **Event-based Nodes** | Each node is a UObject, not a function like in blueprints, allowing encapsulation of logic with data |
| **Async/Latent by Design** | Active nodes subscribe to delegates and react to events by triggering output pins |
| **Custom Pin Definitions** | Every node defines its own input/output pins for flexible flow design |
| **Debug Visualization** | Editor displays debug information on nodes and wires while playing |
| **World Partition Support** | Works well with UE 5.0+ World Partition (no sublevels, no level blueprints) |
| **Gameplay Tag Integration** | Nodes can obtain actors using Gameplay Tags for loose coupling |

## 2. Runtime Requirements

| Environment | Minimum Version | Recommended Version |
|-------------|-----------------|---------------------|
| Unreal Engine | 5.0 | 5.7 |
| Windows | 10 | 11 |

## 3. Dependencies

### Build Dependencies (Build.cs)

| Module | Type | Required |
|--------|------|----------|
| Core | Public | Yes |
| CoreUObject | Public | Yes |
| Engine | Public | Yes |

### Plugin Dependencies (uplugin)

| Plugin | Purpose |
|--------|---------|
| **AssetSearch** | Asset search functionality |
| **EditorScriptingUtilities** | Editor scripting support |
| **EngineAssetDefinitions** | Engine asset definitions |

## 4. Module & Loading

| Property | Value |
|----------|-------|
| Module name | `Flow` |
| Module type | `Runtime` |
| Loading phase | `PreDefault` |
| Editor module | `FlowEditor` (Editor) |
| Debugger module | `FlowDebugger` (DeveloperTool) |

## 5. Key Classes

| Class | Type | Purpose |
|-------|------|---------|
| `UFlowNode` | `UObject` | Base class for all Flow nodes |
| `UFlowAsset` | `UObject` | Flow graph asset containing nodes |
| `UFlowSubsystem` | `UGameInstanceSubsystem` | Central subsystem for Flow management |
| `UFlowComponent` | `UActorComponent` | Component for actors to interact with Flow |
| `UFlowGraph` | `UEdGraph` | Graph editor for Flow assets |
| `UFlowGraphNode` | `UEdGraphNode` | Graph node representation |

## 6. Key Structs

| Struct | Blueprint Type | Purpose |
|--------|---------------|---------|
| `FFlowPin` | No | Input/output pin definition |
| `FFlowConnection` | No | Connection between pins |
| `FFlowNodeState` | No | Runtime state of a node |
| `FFlowEvent` | No | Event data for node communication |

## 7. Key Interfaces

| Interface | Methods |
|-----------|---------|
| `IFlowInterface` | `OnFlowEvent()` - Receive flow events |
| `IFlowComponent` | `RegisterWithFlow()` - Register component with subsystem |

## 8. Project Structure

```
FlowGraph/
├── Source/
│   ├── Flow/                    # Runtime module
│   │   ├── Public/
│   │   │   ├── Nodes/           # Flow node base classes
│   │   │   ├── Types/           # Data types and pin definitions
│   │   │   └── FlowSubsystem.h  # Central subsystem
│   │   └── Private/
│   ├── FlowEditor/              # Editor module
│   │   ├── Public/
│   │   │   ├── Graph/           # Graph editor classes
│   │   │   ├── Asset/           # Asset editor classes
│   │   │   └── DetailCustomizations/
│   │   └── Private/
│   └── FlowDebugger/            # Debugger module
├── Config/
│   ├── BaseFlow.ini
│   └── DefaultFlow.ini
├── Resources/
│   └── Icons/
└── docs/                        # Documentation
    ├── Overview/
    ├── Features/
    ├── Guides/
    └── Releases/
```

## 9. Creating Custom Flow Nodes

### C++ Example

```cpp
UCLASS()
class UFlowNode_CustomAction : public UFlowNode
{
    GENERATED_BODY()

public:
    // Define input pins
    virtual void CreatePins() override;

    // Execute when input is triggered
    virtual void ExecuteInput(const FName& PinName) override;

    // Cleanup when node is deactivated
    virtual void Cleanup() override;

private:
    UPROPERTY(EditAnywhere, Category = "Custom")
    FString Message;
};
```

### Blueprint Nodes

Flow nodes can also be created in Blueprints, recommended for prototyping or rarely-used custom actions.

## 10. Non-Obvious Code Patterns

- **Nodes are async/latent by design**: Unlike blueprint functions, Flow nodes are meant to wait for events. Use `TriggerOutput` to continue execution.
- **No direct actor references**: Use Gameplay Tags or Soft Object References to identify actors, enabling loose coupling and World Partition compatibility.
- **Node class renames break assets**: Flow nodes are referenced by name in Flow assets. Renaming a C++ class without updating the Flow asset will break the graph.
- **PreDefault loading phase**: The Flow module loads at PreDefault phase, so dependencies must be available at that point.

## 11. Common Pitfalls for New Contributors

1. **Forgetting to call TriggerOutput**: Nodes that don't call `TriggerOutput` will hang the flow. Always ensure execution continues.
2. **Using direct actor references**: This breaks World Partition compatibility. Use Gameplay Tags instead.
3. **Node class renames without asset updates**: Flow assets store node class names. Renaming breaks existing graphs.
4. **Not implementing Cleanup()**: Nodes that allocate resources or subscribe to delegates must clean up in `Cleanup()`.
5. **Missing PreDefault dependencies**: Since Flow loads at PreDefault, all dependencies must be available.

## 12. Build & Test Commands

### Build Editor Target
```bash
"<EnginePath>/Engine/Build/BatchFiles/Build.bat" FlowGraphEditor Win64 Development -Project="<ProjectPath>/YourProject.uproject" -WaitMutex -FromMSBuild
```

## 13. Documentation

- [Plugin documentation on GitHub Pages](https://mothcocoon.github.io/FlowGraph/)
- [Getting Started Guide](docs/Overview/GettingStarted.md)
- [Concept Overview](docs/Overview/Concept.md)
- [Feature Documentation](docs/Features/)
- [Release Notes](docs/Releases/)

## 14. Human Review Required Before Implementing

- Changes to `UFlowNode` base class
- Changes to `UFlowAsset` or serialization format
- Adding new pin types or connection policies
- Changes to the Flow Subsystem
- Renaming or removing existing Flow node classes