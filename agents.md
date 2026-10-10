# KiCad Custom Parts Repository

## 1. Purpose

This repository is a reusable catalog of custom KiCad components.

It contains symbols, footprints, and optionally 3D models for parts that are not available in the standard KiCad libraries, or for which the standard KiCad representation is unsuitable for the project.

The repository is intended primarily for use by automated tools and LLM-based engineering agents.

It is **not a runtime dependency of KiCad projects**.

A project using a component from this repository MUST receive a physical copy of the required component files or definitions. Projects MUST NOT reference this repository directly.

The fundamental rule is:

**The central repository is a source catalog. Each KiCad project must remain completely self-contained.**

---

## 2. Design Principles

The repository follows these principles:

* Keep the organization simple.
* Prefer deterministic and descriptive part names over complex categorization.
* Do not create unnecessary categories for individual or uncommon components.
* Use manufacturer part numbers whenever possible.
* Store reusable components centrally.
* Copy only the components required by a project into that project.
* Never create a dependency from a KiCad project to this repository.
* A project must remain usable if this repository is unavailable, moved, renamed, or deleted.
* Existing projects must not change automatically when a component in the central repository is updated.

The project-local copy of a component acts as a snapshot of the component at the time it was incorporated into the project.

---

# 3. Repository Structure

The recommended repository structure is:

```text
kicad-parts-db/
├── README.md
├── symbols/
│   └── CustomParts.kicad_sym
├── footprints/
│   └── CustomParts.pretty/
│       ├── Manufacturer_PartNumber.kicad_mod
│       ├── AnotherPart.kicad_mod
│       └── ...
└── 3d/
    ├── Manufacturer_PartNumber.step
    └── ...
```

Complex category hierarchies SHOULD NOT be introduced unless the repository becomes sufficiently large that they provide a clear practical benefit.

---

# 4. Component Naming

Components SHOULD use the manufacturer's exact part number whenever one exists.

Preferred naming format:

```text
Manufacturer_PartNumber
```

Examples:

```text
JST_BM03B-SRSS-TB
WCH_CH32V003F4P6
Espressif_ESP32-C3-MINI-1
TI_TPS5430DDAR
```

If the manufacturer is unnecessary for disambiguation, the exact part number MAY be used directly.

For generic modules without a formal manufacturer part number, use the commonly recognized product/module name:

```text
ESP32-C3-SuperMini
RP2040-PicoZero
Si4732-Generic-Module
```

If multiple physically incompatible versions of a generic module exist, explicitly distinguish them:

```text
ESP32-C3-SuperMini_RevA
ESP32-C3-SuperMini_RevB
```

Do not assume that two modules sold under the same commercial name have identical footprints.

---

# 5. Symbols

Custom symbols are stored in:

```text
symbols/CustomParts.kicad_sym
```

A custom symbol SHOULD only be created when an appropriate symbol does not already exist in the standard KiCad libraries.

Before creating a new symbol, the agent MUST:

1. Search the standard KiCad libraries.
2. Determine whether an existing symbol accurately represents the electrical device.
3. Search this custom repository.
4. Create a new symbol only if neither source contains an appropriate representation.

A custom footprint does NOT necessarily require a custom symbol.

For example, a component may use a perfectly suitable standard KiCad symbol while requiring a custom physical footprint.

---

# 6. Symbol Metadata

Whenever practical, custom symbols SHOULD contain enough metadata to identify the physical component unambiguously.

Recommended fields include:

```text
Manufacturer
MPN
Datasheet
Description
Footprint
```

Example:

```text
Manufacturer: Espressif
MPN: ESP32-C3-MINI-1
Datasheet: <manufacturer datasheet URL>
Description: ESP32-C3 Wi-Fi/BLE module
Footprint: CustomParts:Espressif_ESP32-C3-MINI-1
```

Additional metadata SHOULD only be introduced when it provides useful engineering information.

Avoid turning symbol metadata into a complicated external inventory or ERP system.

---

# 7. Footprints

Custom footprints are stored as individual `.kicad_mod` files inside:

```text
footprints/CustomParts.pretty/
```

Example:

```text
CustomParts.pretty/
├── JST_BM03B-SRSS-TB.kicad_mod
├── ESP32-C3-SuperMini.kicad_mod
└── WCH_CH32V003F4P6.kicad_mod
```

The footprint name SHOULD identify the exact physical component or package it represents.

An agent MUST NOT assume physical compatibility solely from similar component names.

Pin numbering, pad dimensions, pitch, orientation, mechanical dimensions, and package geometry MUST be verified against reliable documentation whenever a new footprint is created.

---

# 8. 3D Models

3D models are optional.

When a custom component has an associated 3D model, it SHOULD be stored under:

```text
3d/
```

The 3D model SHOULD use the same base naming convention as the corresponding footprint whenever practical.

Example:

```text
footprints/CustomParts.pretty/ESP32-C3-SuperMini.kicad_mod
3d/ESP32-C3-SuperMini.step
```

When a 3D model is copied into a project, its footprint reference MUST use a project-relative path or another path that remains valid within the self-contained project.

It MUST NOT reference the central repository.

---

# 9. Project Integration

The central repository MUST NEVER be registered as a required external library for a project.

Do NOT make project library tables point to this repository.

Do NOT use:

* absolute paths to the repository;
* environment variables pointing to the repository;
* symbolic links to the repository;
* Git submodules for this repository;
* external repository dependencies;
* machine-specific paths.

Instead, physically copy the required components into the KiCad project.

A recommended project structure is:

```text
MyProject/
├── MyProject.kicad_pro
├── MyProject.kicad_sch
├── MyProject.kicad_pcb
├── sym-lib-table
├── fp-lib-table
│
└── libraries/
    ├── symbols/
    │   └── project_symbols.kicad_sym
    │
    ├── footprints/
    │   └── project_footprints.pretty/
    │       ├── RequiredPart1.kicad_mod
    │       └── RequiredPart2.kicad_mod
    │
    └── 3d/
        └── RequiredPart1.step
```

Everything required to open, edit, and manufacture the project SHOULD exist inside the project repository, except for components supplied by the standard KiCad installation.

---

# 10. Project Library Tables

Project-specific library tables SHOULD reference the project's local libraries using `${KIPRJMOD}`.

Example symbol library:

```scheme
(sym_lib_table
  (version 7)
  (lib
    (name "ProjectSymbols")
    (type "KiCad")
    (uri "${KIPRJMOD}/libraries/symbols/project_symbols.kicad_sym")
    (options "")
    (descr "")
  )
)
```

Example footprint library:

```scheme
(fp_lib_table
  (version 7)
  (lib
    (name "ProjectFootprints")
    (type "KiCad")
    (uri "${KIPRJMOD}/libraries/footprints/project_footprints.pretty")
    (options "")
    (descr "")
  )
)
```

The project MUST NOT require configuration of external paths before it can be opened.

---

# 11. Copying Symbols Into a Project

KiCad symbol libraries may contain multiple symbols in a single `.kicad_sym` file.

Therefore, when a project requires one custom symbol, the agent SHOULD NOT blindly copy the entire central `CustomParts.kicad_sym` library.

Instead:

```text
Central repository:

symbols/
└── CustomParts.kicad_sym
        ├── Part_A
        ├── Part_B
        ├── Part_C
        └── Part_D
```

If the project requires only `Part_B` and `Part_D`, the resulting project library should contain:

```text
project_symbols.kicad_sym
    ├── Part_B
    └── Part_D
```

Only the required symbol definitions should be incorporated into the project-local symbol library.

Existing project-local symbols MUST be preserved.

---

# 12. Copying Footprints Into a Project

Footprints are naturally stored as individual files.

If the central repository contains:

```text
CustomParts.pretty/
├── Part_A.kicad_mod
├── Part_B.kicad_mod
├── Part_C.kicad_mod
└── Part_D.kicad_mod
```

and the project requires `Part_B` and `Part_D`, copy:

```text
project_footprints.pretty/
├── Part_B.kicad_mod
└── Part_D.kicad_mod
```

Do not copy unrelated footprints unless there is a specific reason to do so.

---

# 13. Standard KiCad Libraries vs. Custom Repository

The agent SHOULD prefer standard KiCad components when they accurately represent the required component.

The custom repository exists to supplement KiCad, not duplicate it.

For each required component, consider the symbol and footprint independently.

Possible cases include:

```text
Standard symbol + Standard footprint
Standard symbol + Custom footprint
Custom symbol   + Standard footprint
Custom symbol   + Custom footprint
```

Only copy or create custom resources that are actually necessary.

---

# 14. LLM Component Selection Workflow

When an LLM agent needs to add a component to a KiCad project, it MUST follow this general process:

1. Identify the exact physical component.

2. Determine its manufacturer and manufacturer part number when available.

3. Search the standard KiCad libraries for a suitable symbol.

4. Search the standard KiCad libraries for a suitable footprint.

5. Search this repository for missing custom resources.

6. If an appropriate custom resource exists, copy it into the project's local libraries.

7. If no appropriate resource exists, create it based on authoritative component documentation.

8. When a newly created component is generally reusable, add it to the central repository as well as to the current project.

9. Ensure the project references only its local custom libraries.

10. Verify that no project file contains a dependency on the central repository.

---

# 15. Creating New Components

When creating a component that does not yet exist, authoritative sources SHOULD be preferred in this order:

1. Manufacturer datasheet
2. Manufacturer mechanical drawing
3. Manufacturer product documentation
4. Reliable distributor documentation
5. Verified physical measurements
6. Other sources only when necessary

The agent MUST NOT invent mechanical dimensions, pad geometry, pin assignments, or package orientation.

If required information cannot be established reliably, the agent SHOULD explicitly report the uncertainty instead of silently guessing.

---

# 16. Central Repository Updates

Adding or correcting a component in the central repository MUST NOT automatically modify existing projects.

For example:

```text
Central repository:

ESP32-C3-SuperMini.kicad_mod
        ↓ updated
```

must NOT automatically modify:

```text
OldProject/libraries/footprints/project_footprints.pretty/
    ESP32-C3-SuperMini.kicad_mod
```

The project copy is intentionally independent.

Updating an existing project's component from the central repository MUST be an explicit operation.

This prevents a library update from silently changing an already designed PCB.

---

# 17. Updating a Project From the Central Repository

If an agent intentionally updates a project-local component from the central repository, it MUST:

1. Identify the existing project-local component.
2. Compare it with the central version.
3. Determine whether the update changes electrical or mechanical behavior.
4. Replace or modify the project-local component only when explicitly intended.
5. Check symbol pin mapping if the symbol changed.
6. Check footprint pad numbering and geometry if the footprint changed.
7. Check PCB placement and routing implications.
8. Report any potentially breaking change.

An update MUST NOT be treated as a simple file synchronization operation.

---

# 18. Portability Requirement

A completed project should satisfy the following test:

> Copy or clone the project onto another computer with a normal KiCad 7 installation. The project should open correctly without cloning this custom-parts repository, configuring custom environment variables, correcting paths, or installing additional custom libraries.

This is the primary portability requirement.

---

# 19. Rules for Automated Agents

An automated agent working with this repository MUST follow these rules:

**MUST**

* Keep KiCad projects self-contained.
* Physically copy required custom resources into each project.
* Use project-relative library references.
* Prefer `${KIPRJMOD}` for project-local library paths.
* Search standard KiCad libraries before creating duplicates.
* Search this repository before creating a new custom component.
* Preserve existing project-local components.
* Verify component identity before selecting a footprint.
* Use authoritative documentation when creating new components.
* Treat existing project copies as independent snapshots.

**MUST NOT**

* Reference this central repository directly from a KiCad project.
* Require users to clone this repository to open another project.
* Create absolute machine-specific library paths.
* require external environment variables for custom project components.
* use symlinks as project dependencies.
* use this repository as a Git submodule of projects.
* overwrite project-local components merely because a newer central version exists.
* assume that similarly named generic modules are mechanically identical.
* invent missing package dimensions or pin mappings.

**SHOULD**

* Keep naming deterministic and based on manufacturer part numbers.
* Copy only resources actually required by the project.
* Reuse standard KiCad symbols and footprints whenever appropriate.
* Keep the repository hierarchy shallow.
* Include useful manufacturer and datasheet metadata.
* Copy required 3D models when they are useful to the project.

---

# 20. Core Mental Model

An LLM agent should understand the architecture as:

```text
               STANDARD KICAD LIBRARIES
                        │
                        │ use directly when suitable
                        ▼
                 ┌─────────────┐
                 │   PROJECT   │
                 │             │
                 │ self-       │
                 │ contained   │
                 └──────▲──────┘
                        │
                        │ COPY required custom parts
                        │
               ┌────────┴─────────┐
               │ CUSTOM PARTS DB  │
               │                  │
               │ reusable source  │
               │ catalog only     │
               └──────────────────┘
```

The Custom Parts DB is a **catalog**, not a dependency.

The project-local libraries are the **authoritative components for that specific project**.

Once a component has been copied into a project, that project does not depend on the central copy.

---

# 21. Primary Rule

When uncertain about how a custom component should be integrated, apply this rule:

> **A KiCad project must contain everything custom that it needs to remain usable independently of this repository.**

If deleting or disconnecting the central Custom Parts repository would break an existing KiCad project, the integration is incorrect.
