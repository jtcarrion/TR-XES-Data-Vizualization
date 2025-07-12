# Time-Resolved X-Ray Emission Spectroscopy (TR-XES) Data Processing

This repository contains scripts for processing, filtering, and visualizing X-ray Emission Spectroscopy (XES) data collected from SLAC National Accelerator Laboratory's LCLS facility. The pipeline enables femtosecond time-resolved observations of quantum changes in protein complexes by pairing XES data with Serial Femtosecond X-ray Crystallography (SFX) data.

## Overview

The processing pipeline consists of four main components:

1. **Raw Data Processing** (`mpiVDSXESandXTCAVlaser.py`) - Converts raw Epix100 XES detector data into Virtual Data Set (VDS) structures
2. **Laser Event Filtering** (`filter_laser_events.py`) - Identifies and filters events based on laser on/off states
3. **Crystal Hit Filtering** (`filter_cheetah_hits.py`) - Filters events based on SFX crystal hits using Cheetah analysis results
4. **Visualization** (`plot_xes.py`) - Creates plots and quantifies XES spectra

## Installation

### Prerequisites
- Python 3.9+
- Access to SLAC's HPC environment with Psana
- MPI4py for parallel processing
- Required Python packages: `h5py`, `numpy`, `scipy`, `matplotlib`, `psana`

### Setup
```bash
git clone https://github.com/jtcarrion/TR_XES_processing.git
cd TR_XES_processing
```

## Workflow

### Step 1: Raw Data Processing
The main processing script converts raw Epix100 XES detector data into HDF5 format with VDS structures.

```bash
mpirun -np <num_processes> python mpiVDSXESandXTCAVlaser.py -e <experiment> -r <run_number> -o <output_directory>
```

**Required Parameters:**
- `-e, --exp`: Experiment string (e.g., `mfxls0816`)
- `-r, --run`: Run number to process
- `-o, --outdir`: Output directory for processed files

**Optional Parameters:**
- `--laser-on-code`: EVR code for laser on events (default: -1, ask LCLS staff)
- `--store-raw`: Store raw images without pedestal/gain/geometry correction
- `--max-events`: Maximum number of events to process (default: all)
- `--rowmin/--rowmax`: Row range for XES integration (default: 0 to -1)
- `--rotation`: Image rotation in degrees (default: -89)
- `--threshold`: Pixel threshold value (default: 10)
- `--eventsperfile`: Events per output file (default: 10000)
- `--no-xtcav`: Skip XTCAV processing if detector is broken

**Output:**
- Creates HDF5 files with processed spectra, laser states, and timing information
- Generates a VDS file combining all processed data

### Step 2: Laser Event Filtering
Identifies laser on/off events and saves fiducial timestamps.

```bash
python filter_laser_events.py -e <experiment> -r <run_number>
```

**Output:**
- `laser_on_fiducials.h5`: Fiducials for laser on events
- `laser_off_fiducials.h5`: Fiducials for laser off events

### Step 3: Crystal Hit Filtering
Filters events based on Cheetah crystal hit analysis and creates averaged spectra.

```bash
python filter_cheetah_hits.py -f <xes_files> -c <cheetah_hits_file>
```

**Required Parameters:**
- `-f, --files`: Space-separated list of XES H5 files
- `-c, --CheetahHits`: Cheetah hits file (LST format)

**Optional Parameters:**
- `-w, --window`: Smoothing window size (default: 1, no smoothing)
- `-o, --output`: Output filename for plots

**Output:**
- Text files with averaged spectra for different event types
- PNG plots showing laser on/off comparisons
- Background-corrected spectra

### Step 4: Visualization
Creates 2D and 1D visualizations of the processed XES data.

```bash
python plot_xes.py -f <data_file> --upper_bound <upper> --lower_bound <lower>
```

**Required Parameters:**
- `-f, --data`: Input H5 file
- `--upper_bound`: Upper bound for XES spectra integration
- `--lower_bound`: Lower bound for XES spectra integration

**Output:**
- `2D_sum.png`: 2D visualization of thresholded data
- `1D_xes.png`: 1D XES spectrum plot
- `1D_xes.txt`: Numerical data for 1D spectrum

## Example Usage

### Complete Pipeline Example
```bash
# 1. Process raw data (using 8 MPI processes)
mpirun -np 8 python mpiVDSXESandXTCAVlaser.py -e mfxls0816 -r 123 -o ./processed_data

# 2. Filter laser events
python filter_laser_events.py -e mfxls0816 -r 123

# 3. Filter crystal hits (assuming you have Cheetah hits file)
python filter_cheetah_hits.py -f ./processed_data/VDS-r0123-epix.h5 -c cheetah_hits.lst

# 4. Create visualizations
python plot_xes.py -f ./processed_data/VDS-r0123-epix.h5 --upper_bound 100 --lower_bound 50
```

## Data Structure

### Input Data
- **Raw XES Data**: Epix100 detector data from LCLS MFX hutch
- **XTCAV Data**: X-ray pulse characterization data
- **EVR Data**: Event receiver data for laser timing
- **Cheetah Hits**: Crystal hit analysis results from Cheetah software

### Output Data
- **VDS Files**: Virtual Data Set structures containing processed spectra
- **H5 Datasets**: Organized data with laser states, timing, and spectra
- **Averaged Spectra**: Background-corrected spectra for different event types
- **Visualizations**: 2D and 1D plots of XES data

## Key Features

### Signal-to-Noise Improvement
The pipeline significantly improves SNR by:
1. Filtering laser on/off events to reduce background noise
2. Using crystal hit filtering to focus on relevant events
3. Background correction using non-hit background

### Comprehensive Spectral Coverage
- Allows for a full range of energy states across a wide variety of atomic species
- Precise timing synchronization between XES and SFX data

### Parallel Processing
- MPI-based parallel processing for large datasets
- Efficient handling of high-throughput XFEL data
- Scalable to SLAC's HPC environment

## Troubleshooting

### Common Issues
1. **Missing EVR Codes**: Contact LCLS staff for correct laser event codes
2. **XTCAV Issues**: Use `--no-xtcav` flag if detector is malfunctioning
3. **Memory Issues**: Reduce `--eventsperfile` for large datasets
4. **File Permissions**: Ensure write access to output directories

### Performance Tips
- Use appropriate MPI process count for your dataset size
- Monitor memory usage with large datasets
- Consider using `--max-events` for testing on subset of data