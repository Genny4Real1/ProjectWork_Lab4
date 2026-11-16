# Sicurezza Immersiva 81

**Sicurezza Immersiva 81** is a collaborative VR serious game project about recognizing and managing workplace emergencies. The project brings multiple teams’ scenarios together as one coherent application, with shared interactions, feedback, and learning goals.

## Contents

- [Project objectives](#project-objectives)
- [Tools](#tools)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Naming conventions](#naming-conventions)

## Project objectives

The Project Work turns the concepts developed in the earlier lab into one modular VR application for immersive safety learning. Players practice observing a situation, assessing risks, making decisions, taking action, and learning from the consequences. Mistakes should have understandable consequences and support learning rather than reduce the experience to a quiz.

The class project is organized around **four groups, two scenarios per group, eight scenarios integrated into the shared build, and a catalogue of twelve possible scenarios**. The application also provides shared areas such as Home, Training, Mission Selection, profile and results, settings, and debriefing.

> This project is for educational purposes and does not replace mandatory safety training. Safety procedures and scenario outcomes should be reviewed by a qualified safety representative.

## Tools

| Tool | Use in this project |
|---|---|
| Unity Editor | Develop and integrate the VR application and its content. |
| Meta VR CLI (`metavr`) | Look up current Meta VR documentation and support device workflows such as installing and launching builds, collecting logs and screenshots, and profiling. |
| Meta Quest headset | Test the application on target hardware. |
| Git and GitHub | Collaborate on the shared project and its group-owned content. |

For installation, agent integration, and project-specific `metavr` workflows, see the [Meta VR CLI guide](Guides/MetaVRCLI_Guide.md).

## Technology stack

| Component | Version / target |
|---|---|
| Unity | `6000.3.14f1` |
| Meta XR Core SDK | `209.0.0` |
| XR Management | `4.7.0` |
| OpenXR | `1.16.1` |
| Universal Render Pipeline (URP) | `17.3.0` |
| Input System | `1.19.0` |
| Target platform | Meta Quest (see the project guide for the current device workflow) |

Package versions are recorded in `Packages/manifest.json`; the Unity Editor version is in `ProjectSettings/ProjectVersion.txt`.

Builds are stored in the Build/ directory at the root of the repository. Call each build by `Sicurezza-Immersiva-81_ver<num>.apk`

## Project structure

The shared Unity project content is organized under `Assets/_Project/`:

```text
Assets/_Project/
├── Docs/                 # Documentation for developing and integrating the application
├── Groups/
│   ├── G_Ludovica/
│   │   └── Assets/       # Group-owned scenario and project assets
│   ├── G_Marius/
│   │   └── Assets/
│   ├── G_Simone/
│   │   └── Assets/
│   └── G_Tiziano/
│       └── Assets/
├── Home/                 # Shared application architecture outside the scenarios
└── Shared/               # Source assets intended for reuse across groups
```

Each group’s `Assets/` area holds the files for that group’s scenarios. It may also contain scenario-specific instances of materials or other assets sourced from `_Project/Shared/`; keep reusable source assets in `Shared/` so groups can work from a common reference.

`Home/` is for the parts of the application that are not a scenario, including the Home experience, Training, and Mission Selection. `Docs/` is for project documentation that supports developing the application. These folders describe the intended ownership and organization; add group `Assets/` folders as their content is created.

## Naming conventions

Use the conventions summarized from the project’s `Nomenclatura.md` reference to keep assets and code recognizable across groups.

### Asset prefixes

| Prefix | Use |
|---|---|
| `CANVAS_` | UI canvas |
| `BTN_` | Button |
| `IMG_` | UI image |
| `PANEL_` | UI panel |
| `LBL_` | UI text or label |
| `ICN_` | Icon |
| `HUD_` | In-scenario UI element |
| `TEX_` | Texture |
| `MAT_` | Material |
| `MESH_` | 3D model |
| `SFX_` | Short sound effect |
| `VFX_` | Visual effect |
| `SCN_` | Unity scene |
| `SO_` | ScriptableObject configuration data |
| `FNT_` | Font |
| `ANIM_` | Animation or clip |
| `MNG_` | Manager |

### General rules

- Write the prefix in uppercase followed by an underscore; use PascalCase for the descriptive name, for example `BTN_StartMission`.
- Avoid spaces, accents, and special characters in technical names. Add variants as suffixes, such as `BTN_StartMission_Hover` or `TEX_BrickWall_Albedo`.
- Name an asset for its purpose, not its appearance, and use the same term for the same concept throughout the project.
- Use English for technical names. Put user-facing text in localization data with stable keys, such as `ui.button.start_mission`.
- Use PascalCase for classes, functions, and public variables; camelCase for private and protected variables; and UPPERCASE for constants. Choose descriptive names and document functions with summaries.
- Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for writing commit title and descriptions. It is a lightweight convention for creating an explicit commit history.  
  