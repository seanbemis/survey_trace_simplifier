# Field Geophysics Survey Line Processor

A Python-based Jupyter Notebook tool designed to clean noisy GNSS/GPR survey lines, extract high-fidelity topographic profiles from DEMs, and format the data for immediate use in geophysical processing software.

## Features

* **File Handling:** Natively reads `.shp`, `.geojson`, `.kml`, and compressed `.kmz` files on Windows/Mac/Linux.
* **Interactive Geometry Tuning:** Uses `ipywidgets` to visually tune line smoothing, outlier removal, spike windows, and clustering tolerances in real-time before committing saves.
* **Algorithm Choices:** Supports both Ramer-Douglas-Peucker (RDP) decimation for angular paths and B-Spline interpolation for fluid, curving paths.
* **Rolling Median Outlier Filter:** Replaces standard geometric baselines with a statistical moving window to successfully "outvote" and slice off multi-node GNSS multipath errors.
* **Bilinear DEM Extraction:** Discretizes the survey line to match GPR trace spacing and performs sub-pixel bilinear interpolation against a high-res DEM to prevent "stair-step" topographic profiles.
* **Automated Provenance:** Generates an accompanying `_processing_metadata.txt` log documenting every tuning parameter and data source used to create the output files.

## Environment Setup

Geospatial libraries rely on complex underlying C-libraries (like GDAL and PROJ) that can easily conflict. It is highly recommended to run this tool in a dedicated, isolated Conda environment.

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

## Security & Path Configuration

To prevent sensitive local file paths (e.g., client names, computer usernames, proprietary drive structures) from being committed to version control, this notebook uses an external configuration file.

1. Create a file named `local_paths.py` in the same directory as the notebook.
2. Ensure `local_paths.py` is listed in your `.gitignore`.
3. Define your absolute file paths within this file:

```python
# local_paths.py example
RAW_LINE = r"D:\Path\To\raw_line.kmz"
CLEANED_LINE = r"D:\Path\To\cleaned_line.shp"
DEM = r"D:\Path\To\project_dem.tif"
```

## Workflow Guide

The notebook is split into two distinct operational phases:

### Phase 1: Geometry Cleaning
1. Update your `local_paths.py` file to point to your new survey data.
2. If you know the physical 3D ground distance (e.g., from an odometer wheel), enter it in the config cell to calculate an automated 2D-to-3D scaling factor.
3. Run the **Interactive Tuner** cell. 
   * Use the **Spike Window** slider (odd numbers) to set the "voting window" size to drop multipath strings.
   * Adjust **Outlier**, **Cluster**, and **Smooth Factor** until the blue output line appropriately models your physical path.
4. Run the **Export Cell** to lock in the geometry and save the shapefile.

### Phase 2: Topographic Profile Extraction
1. Set your `trace_spacing_m` to exactly match the collection parameters of your GPR system.
2. Execute the extraction pipeline. The tool will handle all CRS reprojections, perform the bilinear extraction, and generate:
   * A visual cross-section plot for QC.
   * A dynamically named `.csv` file with `Distance_m`, `Easting`, `Northing`, and `Elevation_m` rounded to cm-scale precision.
   * A `_processing_metadata.txt` file logging the exact parameters and DEM used.