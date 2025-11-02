## Quick summary

This repo is Fooocus — an offline Stable Diffusion XL-based image generator with a Gradio UI. Key entry points and conventions are listed below so an AI coding agent can be productive immediately.

## Where to start (entry points)
- `entry_with_update.py` — primary launcher used in docs; handles CLI flags, environment prep and model downloads.
- `launch.py` — prepares runtime (installs/validates torch/xformers and calls `webui`).
- `webui.py` — builds the Gradio UI; most UI wiring and callbacks live here.
- `modules/` — modular code (config, flags, model loading, workers, utils). Many edits should be made here for core behavior changes.
- `presets/` — JSON presets that define default models/configs per flavor (default/anime/realistic).

## Important workflows (how to run & debug)
- Local (conda):
```bash
conda env create -f environment.yaml
conda activate fooocus
pip install -r requirements_versions.txt
python entry_with_update.py --listen
```
- Local (venv/python): install `requirements_versions.txt` and run `python entry_with_update.py`.
- Windows packaged: use provided `run.bat`, `run_anime.bat`, `run_realistic.bat` which wrap an embedded Python.
- Docker: see `Dockerfile` and `docker-compose.yml`. The Docker image installs `requirements_docker.txt` and exposes port `7865` by default.

Debug flags and helpers
- `--debug-mode` to enable debug logging paths.
- `--listen --port <n>` to bind remote access (Gradio). `--share` to expose via gradio.live.
- Many behaviors are controlled by `args_manager.args` and `modules.config` — change those for global behavior.

## Project-specific conventions & patterns
- Global singletons: the code frequently uses `modules.config`, `args_manager.args`, and `shared` to hold runtime state. Prefer modifying `modules/config.py` or `args_manager` when changing defaults.
- UI wiring: `webui.py` builds the Gradio Blocks and connects callbacks. Callbacks often return `gr.update(...)` tuples — follow this pattern when adding UI interactions.
- Background work and tasks: use `modules/async_worker.AsyncTask` and `worker.async_tasks`. Look at `generate_clicked` in `webui.py` for an example of yielding previews/results.
- Model/download flow: model downloads are orchestrated in `launch.py` (`download_models`) using `modules.model_loader.load_file_from_url`. Presets may override default downloads (see `presets/*.json`).
- Config file: runtime config is generated at `Fooocus/config.txt` after first run. Treat it as authoritative for runtime paths.

## Where to change common behaviors (concrete examples)
- Change default positive prompt: `modules/config.py` → `default_prompt` or edit `presets/default.json`.
- Add a CLI flag: extend `args_manager` and reference it in `entry_with_update.py` / `launch.py` for behavior.
- Add a new Gradio control: modify `webui.py` and follow existing pattern for `.change(...)/.click(...)` handlers and `gr.update` returns.

## Tests & quick verifications
- Tests live under `tests/`. Use pytest from the conda/venv environment. There are project-specific integration points (models, large deps), so unit tests that mock models are preferred for CI.

## Caution & gotchas
- Many files are large and rely on runtime-downloaded models; running end-to-end often requires large binaries and GPU resources.
- `xformers` is optional and installed conditionally (see `launch.py` and `Dockerfile`). Modify `TRY_INSTALL_XFORMERS` and the Dockerfile carefully.
- Configs and model files are stored in `models/` and paths in `modules/config.py`. Changing those paths requires updating Docker `environment` vars in `docker-compose.yml`.

## Useful files to inspect when making changes
- `entry_with_update.py`, `launch.py`, `webui.py` — runtime bootstrap and UI.
- `modules/config.py`, `modules/flags.py`, `modules/model_loader.py`, `modules/async_worker.py` — core helpers.
- `presets/*.json` — default model/preset mapping.
- `Dockerfile`, `docker-compose.yml`, `environment.yaml`, `requirements_versions.txt`, `requirements_docker.txt` — build/runtime environment definitions.

If any section is unclear or you'd like more examples (e.g., how to add a new preset, add a Gradio control, or stub model loading for unit tests), tell me which area and I will expand with concrete, annotated code snippets.
