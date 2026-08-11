

# PropFlow Catalyst for Unreal Engine
![Unreal Engine](https://img.shields.io/badge/UnrealEngine-5-blue?logo=unrealengine) ![License](https://img.shields.io/badge/License-MIT-green) [![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/ZarzaNg/AruEditorUtilities)

PropFlow Catalyst is a Unreal Engine plugin that facilitates the bulk configuration of assets based on the assetisation of the configuration process.

## ✨Features
- **Rule-Driven Attribute Batch Modification**: Modify multiple attributes in bulk using predefined rule templates, eliminating manual repetition.
- **Configuration Pipeline Assetization**: Save complete configuration workflows as version-controlled, shareable assets.
- **Modular Configuration Design**: Split large configuration tables into independent modules to reduce maintenance complexity.

## 🕹Getting Started

### 📥 Installation
1. **Download the Plugin**  
   - Clone this repository:  
     ```bash
     https://github.com/ZarzaNg/AruEditorUtilities.git
     ```
   - Or [download as a ZIP](https://github.com/ZarzaNg/AruEditorUtilities/archive/refs/heads/main.zip) and extract it.

2. **Install to Your Project**  
   - Copy the entire plugin folder to your project's `Plugins/` directory:  
     ```
     YourProject/  
     └── Plugins/  
         └── # Paste the downloaded plugin here
     ```

3. **Enable the Plugin**  
   - Open your project in the editor.  
   - Edit → Plugins, find this plugin and enable. ![EnablePlugin](Images/EnablePlugin.png)
  
### 🛠️ Usage
1. **Access Asset Actions**
   - Right-click on any asset you want to edit in your project's content browser.  
   - Navigate to the context menu: **ScriptedAssetActions → Handyman 68** ![AccessAssetActions](Images/AccessAssetActions.png)

2. **Choose Operation Mode**  
   - 🔥 **Quick Operation**  
     - Select `Process Asset Values` from submenu  
     - In the popup window:  
       1️⃣ **Setup Conditions** - Define filtering rules (e.g. "Does property name match?")  
       2️⃣ **Setup Actions** - Choose processing method (e.g. "Set new value to property.")  
       3️⃣ **Execute** - Applies to all selected assets matching conditions
![QuickProcessing](Images/QuickProcessing.gif)

   - 📁 **Preset DataAsset Workflow**  
     - Prepare preset DataAssets containing preconfigured:  
       - `Conditions` (Asset filtering logic)  
       - `Actions` (Processing sequence)  
     - Select `Process Asset Values with Configs` from submenu  
     - Choose your prebuilt data asset from file picker  
     - System will automatically validate and execute the workflow
![ModularSettings](Images/ModularSettings.gif)

3. **Modular Configuration**
   - When managing objects with numerous settings (e.g. 50+ parameters), extract frequently-tuned parameters into DataAsset and configure them via:
      - **DataTables** (for CSV-driven bulk updates)
      - **Parameter Inheritance** (reference values from master assets)
![GetValueFromDataTable](Images/GetValueFromDataTable.gif)

▶️ **Recommendation**  
- Start with Quick Operation for single-batch tasks  
- Use DataAsset presets for recurring complex workflows

⚠️ **Note**:
- Always verify filter conditions before execution, a condition like "PropertyName=MyFloat" will include all the properties named "MyFloat" in different scopes.
- Use version control diff tools to inspect modifications.

## 🎯Interfaces
### 🔍 Custom Filters
Create a custom filter by inheriting FAruFilter and implementing the condition check logic:
```C++
USTRUCT()
struct FMyCustomFilter : public FAruFilter
{
    GENERATED_BODY()
public:
    virtual ~FMyCustomFilter() override {};
    virtual bool IsConditionMet(const FProperty* InProperty, const void* InValue, const FInstancedPropertyBag& InParameters) const override
    {
        // Your custom condition check logic here
        // InProperty: Metadata of the current property
        // InValue: Memory address of the property value
        
        // Example: Check if property name contains "Health"
        return InProperty->GetName().Contains(TEXT("Health"));
    }
};
```

### ⚙️ Custom Predicates (Operations)
Create a custom operation by inheriting FAruPredicate and implementing the execution logic:
```C++
USTRUCT()
struct FMyCustomAction : public FAruPredicate
{
    GENERATED_BODY()
public:
    virtual ~FMyCustomAction() override {};
    virtual bool Execute(const FProperty* InProperty, void* InValue, const FInstancedPropertyBag& InParameters) const override
    {
        // Example: Double integer properties
        if (FIntProperty* IntProp = CastField<FIntProperty>(InProperty))
        {
            int32 Value = IntProp->GetPropertyValue(InValue);
            IntProp->SetPropertyValue(InValue, Value * 2);
            // Return true if the operation is successful.
            return true;
        }
        return false;
    }
};
```

## 📚 Asset Collector: Tag-based Checking

This plugin provides `AssetCollector/` capabilities to organize and validate assets using maintained tag objects.

- **Key Types**
  - `UAruAssetCollector` (`Source/AruEditorUtilities/Public/AssetCollector/AruAssetCollector.h`)
    - `Collect()`: Implement in Blueprint to populate internal `AruAssetObjects`.
    - `Get()`: Retrieve all collected asset objects.
    - `ClearAssets()`: Clear current collection results.
    - `AddInstance(UObject* SourceObject, const TArray<FName>& Tags)`: Create and append a `UAruAssetObject` with object and tags.
  - `UAruAssetObject` (`Source/AruEditorUtilities/Public/AssetObject/AruAssetObject.h`)
    - Maintains `ReferencedObject`, `AssetName`, and `AssetTags` (`TArray<FName>`).
    - Exposes `AddAssetTag()` and `GetAssetTags()` to manage and read tags.

- **Recommendations**
  - Use `UAruAssetCollector::AddInstance()` to attach business-relevant tags during collection (e.g., `NeedsLOD`, `HasGameplayTag`, `MaterialVariantA`).
  - Before actions/validations, read `UAruAssetObject::GetAssetTags()` to perform include/require/mutually-exclusive checks and decide whether to proceed.
  - Combine with `ActionTags`/`ValidationTags` (if used in your configs) to scope definitions by category.

- **Blueprint Flow (Conceptual)**
  - Derive from `UAruAssetCollector` and implement `Collect()`:
    1. Iterate assets (Content Browser query or custom source).
    2. For each match, call `AddInstance(Object, Tags)` to create and store a `UAruAssetObject`.
    3. Later, inspect `Get()` results and evaluate tags to decide which action sets to run.

> The collector maintains a pool of `UAruAssetObject`, each carrying the source `UObject` and a set of `FName` tags. Build your checks around these tags to gate subsequent actions/validations.

## 🧩 Proxy Coverage Extension

We expanded proxy coverage so filters can perform Blueprint-defined checks across many value types without C++ changes.

- **Key Types and Locations**
  - `FAruFilter_Proxy` / `FAruFilter_BlueprintProxy` (`Source/AruEditorUtilities/Public/AssetFilters/AruFilter_Proxy.h`)
  - `UAruFilterProxy` (Blueprint-implementable proxy)

- **Overridable check entries (`UAruFilterProxy`)**
  - `CheckBoolValue(bool)`
  - `CheckIntValue(int32)`
  - `CheckFloatValue(float)`
  - `CheckStringValue(const FString&)`
  - `CheckTextValue(const FText&)`
  - `CheckObjectValue(const UObject*)`
  - `CheckNameValue(const FName&)`
  - `CheckInstancedStructValue(const FInstancedStruct&)`
  - `CheckGameplayTagValue(const FGameplayTag&)`
  - `CheckGameplayTagContainerValue(const FGameplayTagContainer&)`
  - `CheckEnumValue(int32, const UEnum*)`

- **Usage**
  1. Create a Blueprint class derived from `UAruFilterProxy` and implement the `Check*` functions you need (others can remain default).
  2. In configs, use `FAruFilter_BlueprintProxy`, set `ProxyClass`, or provide an instanced `ProxyInstance`.
  3. At runtime, the filter routes to the appropriate `Check*` entry based on the property type.

- **Working with tag checks**
  - If your `UAruAssetCollector` populated tags for filtering, incorporate that context inside your proxy checks to decide condition matches.

> The proxy-based filter allows Blueprint-defined checks for various value kinds (bool, int, float, string, text, object, name, struct, GameplayTag(s), enum), making it easy to extend without C++ changes.

### Predicate-side Proxy

- **Key Types and Locations**
  - `FAruPredicate_Proxy` / `FAruPredicate_BlueprintProxy` (`Source/AruEditorUtilities/Public/AssetPredicates/AruPredicate_Proxy.h`)
  - `UAruPredicateProxy` (Blueprint-implementable proxy)

- **Overridable processing entries (`UAruPredicateProxy`)**
  - `ProcessBoolValue(bool)`
  - `ProcessIntValue(int32)`
  - `ProcessFloatValue(float)`
  - `ProcessStringValue(const FString&)`
  - `ProcessTextValue(const FText&)`
  - `ProcessObjectValue(const UObject*)`
  - `ProcessNameValue(const FName&)`
  - `ProcessStructValue(const FInstancedStruct&)`
  - `ProcessInstancedStructValue(const FInstancedStruct&)`
  - `ProcessGameplayTagValue(const FGameplayTag&)`
  - `ProcessGameplayTagContainerValue(const FGameplayTagContainer&)`
  - `ProcessEnumValue(int64, const UEnum*)`

- **Usage**
  1. Create a Blueprint class derived from `UAruPredicateProxy` and implement the relevant `Process*` entries to transform/write back values.
  2. In configs, use `FAruPredicate_BlueprintProxy`, set `ProxyClass`, or provide an instanced `ProxyInstance`.
  3. At runtime, the predicate chooses the appropriate `Process*` entry based on property type and writes the returned value back.

> Predicate proxy lets you transform target values in Blueprint per type, then write them back during execution.
