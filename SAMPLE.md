# Sample Markdown

This is a sample markdown file demonstrating basic formatting.

## Overview

Path2Mech is an AI-assisted H&E screening tool built around a LazySlide + CONCH pipeline.

## Pipeline Steps

1. Load whole-slide image and initialize `.zarr` storage
2. Detect tissue regions and generate tiles
3. Extract image features with the CONCH model
4. Encode text prompts and compute text-image similarity
5. Generate shape annotations from similarity heatmaps
6. Export annotations to QuPath as GeoJSON
7. Launch a Gradio viewer for side-by-side comparison

## Example Code Block

```python
import lazyslide as zs

slide = zs.open_wsi("example.svs", backed_file="results/example.zarr")
zs.pp.find_tissues(slide)
```

## Links

- [LazySlide](https://github.com/rendeirolab/LazySlide)
- [CONCH](https://github.com/mahmoodlab/CONCH)
