# uda-mast-feedstock

Conda packaging for [uda-mast](https://pypi.org/project/uda-mast/), UKAEA's
MAST/MAST-U helpers for the [pyuda](https://github.com/ukaea/UDA) client.
Upstream publishes uda-mast to PyPI only. This repo repackages that exact
release for the `ga-fdp` Anaconda channel:

```bash
conda install -c ga-fdp -c conda-forge uda-mast
```

This pulls in `pyuda` from conda-forge. With `uda-mast` installed,
`pyuda.Client()` gains `list_signals`, `list_shots`, `latest_shot`,
`latest_source_pass`, `get_shot_date_time`, `get_images`, `geometry`, and more.

## What the recipe changes

Only one thing. `recipe/0001-use-conda-opencv.patch` removes the pip-only
`opencv-python-headless` requirement, and the recipe depends on conda-forge's
`py-opencv` instead, which provides the same `cv2` module. Everything else is
upstream's sdist, pinned by sha256.

## Building locally

```bash
rattler-build build --recipe recipe/ -c conda-forge
```

The build runs the recipe tests: imports, `pip check`, `pyuda.Client`
registering the MAST helpers, `cv2` importing, and the version matching.

## Releasing

`.github/workflows/build.yaml` builds and tests every push and pull request.
Publishing uses tags and needs the `ANACONDA_API_TOKEN` secret, which must be
able to upload to `ga-fdp`:

1. Bump `version` and `sha256` in `recipe/recipe.yaml`. Refresh the patch if
   upstream's `pyproject.toml` changed. Reset `build.number` to 0.
2. Push `staging-X.Y.Z`. This uploads to `ga-fdp/label/staging`:
   ```bash
   conda install -c ga-fdp/label/staging -c conda-forge uda-mast
   ```
3. Push `release-X.Y.Z`. This moves those same files to `main` without
   rebuilding.

`.github/scripts/channel_tools.py` does the per-file relabelling. It is copied
from GA-FDP/toksearch, which explains why `anaconda move` can't be used.

## conda-forge

The recipe follows conda-forge conventions, so it could go to
staged-recipes. conda-forge requires a license file, and the sdist doesn't
include one. The package metadata says Apache-2.0.

## License

The recipe files are provided as-is. uda-mast itself is Apache-2.0, by UKAEA.
