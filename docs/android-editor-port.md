# Android editor port

Status: **porting work required; no Android editor APK is produced by the current project**.

This document records the implementation boundary for bringing the full JasperEngine editor to Android. It is deliberately separate from the Android game player already present in `engine/workspaces/androidstudio`.

## What exists today

- `engine/platform/android` contains the native Android game runtime, GameActivity integration, OpenGL ES-oriented platform services, lifecycle handling and touch input.
- `engine/workspaces/androidstudio` packages that runtime as an Android application. Its Gradle build invokes `engine/CMakeLists.txt`, not the desktop editor target.
- The repository-root `CMakeLists.txt` builds the desktop editor. Its backend selection currently supports Linux, Windows and macOS; unsupported systems terminate configuration. The editor backend sources are `editor/backend/BackendLinux.cpp`, `BackendWin.cpp` and `BackendMac.mm`.
- `editor/main.cpp` starts `editor::Backend::init()`. There is no Android editor backend wired into that executable.

Therefore, the existing Android workspace is a game/player build and must not be described as the mobile editor.

## Porting architecture

The Android editor should be a separate application target that reuses the editor model/UI sources while providing Android-specific platform services.

1. **Build graph**
   - Add an explicit Android editor target and keep the existing player target unchanged.
   - Reuse the editor source/object libraries only after removing desktop-only sources and dependencies from the Android target.
   - Use the Android NDK toolchain, arm64-v8a first, C++20, GameActivity and GLES 3.
   - Do not silently select the player target when the editor is requested.

2. **Native editor backend**
   - Implement an Android backend for the editor's `Backend` contract.
   - Own EGL/GLES context and surface lifecycle, pause/resume, resize, focus and frame scheduling.
   - Translate Android touch, stylus, keyboard and gamepad events into the editor input model.
   - Implement clipboard, text input/IME, safe-area/insets, dialogs and app-private storage/document-provider access.
   - Handle activity recreation and context loss without losing project state.

3. **Mobile interaction**
   - Preserve the desktop editor's visual language and ImGui-based tools.
   - Add responsive layouts: collapsible hierarchy/inspector/resource panels, touch-sized controls, contextual toolbars and a full-screen scene canvas.
   - Support pinch zoom, two-finger pan, tap selection, long-press/context actions and touch-friendly gizmos.
   - Route text editing through Android's IME rather than relying on a physical keyboard.

4. **Project workflow**
   - Create/open/save projects through Android's Storage Access Framework.
   - Keep project paths as URI-backed/document-tree access where applicable; do not assume unrestricted filesystem paths.
   - Verify scene, asset, Lua and resource import/export flows on-device.

5. **Build and validation**
   - Add CI that builds the editor APK for arm64-v8a.
   - Run native/unit tests and an emulator smoke test (launch, create project, create scene/entity, edit component, save, close and reopen).
   - Publish an APK only from a successful, reproducible build. A successful player APK is not evidence that the editor works.

## Acceptance criteria for a usable editor APK

- Launches on a real Android device and survives pause/resume and rotation.
- Creates, saves, reopens and deletes a project using user-selected storage.
- Creates a scene and entities; hierarchy selection and inspector edits persist.
- Imports and previews at least the supported image/audio assets.
- Runs a scene in an embedded play view and returns to editing.
- Lua editing accepts multiline input with the Android keyboard.
- No desktop-only backend, windowing library or host executable is required at runtime.

## Current delivery state

The branch currently contains this porting plan only. The native editor backend, responsive editor UI, CI build and APK are not implemented yet. Do not install or distribute the existing Android player as if it were the editor.
