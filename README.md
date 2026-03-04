# Field Geophysics Survey Line Processor

A Python-based Jupyter Notebook tool designed to clean noisy GNSS/GPR survey lines, extract high-fidelity topographic profiles from DEMs, and format the data for immediate use in geophysical processing software.

## Features

* **File Handling:** Natively reads `.shp`, `.geojson`, `.kml`, and compressed `.kmz` files on Windows/Mac/Linux.
* **Interactive Geometry Tuning:** Uses `ipywidgets` to visually tune line smoothing, outlier removal, and clustering tolerances in real-time before committing saves.
* **Algorithm Choices:** Supports both Ramer-Douglas-Peucker (RDP) decimation for angular paths and B-Spline interpolation for fluid, curving paths.
* **Bilinear DEM Extraction:** Discretizes the survey line to match GPR trace spacing and performs sub-pixel bilinear interpolation against a high-res DEM to prevent "stair-step" topographic profiles.
* **Automated Provenance:** Generates an accompanying `_metadata.txt` log documenting every tuning parameter and data source used to create the output files.

## Environment Setup

Geospatial libraries rely on complex underlying C-libraries (like GDAL and PROJ) that can easily conflict. It is recommended to run this tool in a dedicated, isolated Conda environment.

1. **Create the environment using conda-forge:**
   ```bash
   conda create -n basicGIS_env -c conda-forge python=3.11
   ```
2. **Activate the environment:**
   ```bash
   conda activate basicGIS_env
   ```
3. **Install the required GIS stack:**
   ```bash
   conda install -c conda-forge geopandas rasterio shapely fiona pyproj pandas matplotlib scipy ipywidgets
   ```
4. **Link to Jupyter/VSCode:**
   ```bash
   python -m ipykernel install --user --name=basicGIS_env --display-name "Python (basicGIS_env)"
   ```

## Workflow Guide

The notebook is split into two distinct operational phases:

### Phase 1: Geometry Cleaning
1. Set your input filepath (e.g., raw `.kmz` from the field) and desired output filepath (`.shp`) in the Configuration cell.
2. If you know the physical 3D ground distance (e.g., from an odometer wheel), enter it to calculate an automated 2D-to-3D scaling factor.
3. Run the **Interactive Tuner** cell. Adjust the sliders (Cluster, Outlier, Smooth Factor) until the blue output line appropriately models your physical path without cutting corners.
4. Run the **Export Cell** to lock in the geometry and save the shapefile.

### Phase 2: Topographic Profile Extraction
1. Provide the path to your newly cleaned `.shp` file and a local DEM (`.tif`).
2. Set your `trace_spacing_m` to exactly match the collection parameters of your GPR system.
3. Execute the pipeline. The tool will handle all CRS reprojections, perform the bilinear extraction, and generate:
   * A visual cross-section plot for QC.
   * A `.csv` file with `Distance_m`, `Easting`, `Northing`, and `Elevation_m`.
   * A `_processing_metadata.txt` file logging the exact parameters and DEM used.