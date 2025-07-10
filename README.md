# OSDR File Downloader

A Python tool for downloading files from NASA's [Open Science Data Repository (OSDR)](https://osdr.nasa.gov/bio/repo/) with advanced filtering and organization capabilities.

## Description

The OSDR File Downloader is a command-line tool that provides programmatic access to files stored in NASA's [Open Science Data Repository (OSDR)](https://osdr.nasa.gov/bio/repo/), which includes omics data from GeneLab and non-omics (e.g. physiologic, phenotypic, imaging, behavioral, etc.) data from ALSDA (Ames Life Sciences Data Archive). The tool uses the [OSDR biodata API](https://visualization.osdr.nasa.gov/biodata/api/) to query, filter, and download files with support for:

- **Intelligent file organization** - Automatically organizes files by measurement type and technology type
- **GeneLab processed file detection** - Identifies and separates GeneLab processed files using protocol references
- **Advanced filtering** - Filter by measurement type, technology type, file extensions (include/exclude)
- **Duplicate handling** - Automatically removes duplicate file entries from metadata queries
- **Robust error handling** - Includes fallback download mechanisms and comprehensive error reporting
- **Preview mode** - List files without downloading to preview what would be downloaded

The tool automatically creates a hierarchical directory structure that separates GeneLab processed data files from other data files (specified as "raw" in the outputs), making it easy to organize and access different types of data files.

## Installation Instructions

### Prerequisites

- Python 3.7 or higher
- Internet connection for API access

### Dependencies

Install the required Python packages:

```bash
pip install requests
```

**Built-in dependencies** (no installation required):
- `argparse` - Command line argument parsing
- `os` - Operating system interface
- `sys` - System-specific parameters
- `urllib.parse` - URL parsing utilities
- `pathlib` - Object-oriented filesystem paths
- `re` - Regular expression operations
- `typing` - Type hints

### Installation

1. Download the `osdr_downloader.py` script
2. Make it executable (optional):
   ```bash
   chmod +x osdr_downloader.py
   ```

### Verification

Test the installation by running:
```bash
python3 osdr_downloader.py --help
```

## Usage Instructions

### Basic Syntax

```bash
python3 osdr_downloader.py --osd <OSD_NUMBER> [OPTIONS]
```

### Example Commands

#### Basic Usage

```bash
# List all files for a dataset
python3 osdr_downloader.py --osd OSD-101 --list

# Download all files for a dataset
python3 osdr_downloader.py --osd OSD-101
```

#### Filtering by Measurement and Technology Type
> For a list of OSDR Measurement and Technology types, visit the [Public Config List in BDME](https://docs.google.com/spreadsheets/d/1m6J9DOwHvkK_Kj82IOdLC9AZuPewIBWt3dN4n5afU5M) google sheet. Note that non-omics Measurement and Technology types are found in the "Phenotypic-Physiological-Imaging-Video" tab and omics ones are found in the "Omics" tab.

```bash
# List files for specific measurement and technology type
python3 osdr_downloader.py --osd OSD-101 --measurement "transcription profiling" --tech "RNA-Seq" --list

# Download RNA-Seq files only
python3 osdr_downloader.py --osd OSD-101 --measurement "transcription profiling" --tech "RNA-Seq"
```

#### File Extension Filtering

```bash
# Download only CSV files
python3 osdr_downloader.py --osd OSD-101 --ext csv

# Download all files except large archive files
python3 osdr_downloader.py --osd OSD-101 --exclude-ext tar.gz

# Download HTML files for all measurement/technology combinations
python3 osdr_downloader.py --osd OSD-101 --ext html
```

#### Advanced Filtering

```bash
# Download transcription profiling CSV files, excluding ZIP files
python3 osdr_downloader.py --osd OSD-101 --measurement "transcription profiling" --ext csv --exclude-ext zip

# List proteomics files with custom output directory
python3 osdr_downloader.py --osd OSD-101 --measurement "protein expression profiling" --out ./proteomics_data --list
```

#### Complex Filtering Examples

```bash
# Download only GeneLab processed RNA-Seq files (CSV format)
python3 osdr_downloader.py --osd OSD-101 --measurement "transcription profiling" --tech "RNA-Seq" --ext csv

# Exclude large raw data files, keep processed data
python3 osdr_downloader.py --osd OSD-101 --exclude-ext tar.gz --exclude-ext fastq.gz

# Download to specific directory with verbose output
python3 osdr_downloader.py --osd OSD-101 --out ~/research/osd-101 --list
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `--osd` | String | Yes | OSD dataset number (e.g., `OSD-101`, `OSD-249`) |
| `--measurement` | String | No | Measurement type filter (e.g., `"transcription profiling"`, `"protein expression profiling"`) |
| `--tech` | String | No | Technology type filter (e.g., `"RNA-Seq"`, `"mass spectrometry"`, `"microarray"`) |
| `--ext` | String | No | File extension to include (e.g., `csv`, `txt`, `html`, `fastq.gz`) |
| `--exclude-ext` | String | No | File extension to exclude (e.g., `tar.gz`, `zip`, `fastqc.html`) |
| `--out` | String | No | Custom output directory path (default: `osdr_downloads_OSD-#`) |
| `--list` | Boolean | No | List files only, do not download (flag, no value needed) |

### Parameter Details

#### `--osd` (Required)
- **Format:** Must follow pattern `OSD-XXX` where XXX is a number
- **Examples:** `OSD-101`, `OSD-249`, `OSD-1`
- **Validation:** Script validates format and exits with error if invalid

#### `--measurement`
- **Common Values:**
  - `"transcription profiling"`
  - `"protein expression profiling"`
  - `"phenotyping"`
  - `"metabolite profiling"`
- **Behavior:** If specified without `--tech`, downloads all technology types for that measurement
- **Case Sensitivity:** Case-insensitive matching

#### `--tech`
- **Common Values:**
  - `"RNA-Seq"` / `"RNA sequencing"`
  - `"microarray"`
  - `"mass spectrometry"`
  - `"microscopy"`
- **Special Handling:** Parentheses in technology names are automatically converted to wildcards for regex matching

#### `--ext` and `--exclude-ext`
- **File Extensions:** Specify without the leading dot (e.g., `csv` not `.csv`)
- **Common Extensions:**
  - Data files: `csv`, `txt`, `tsv`
  - Archives: `tar.gz`, `zip`
  - Raw data: `fastq.gz`, `bam`
  - Reports: `html`, `pdf`
- **Validation:** Cannot specify the same extension for both include and exclude

#### `--out`
- **Default Behavior:** Creates `osdr_downloads_OSD-XXX` in current directory
- **Custom Path:** Can specify relative or absolute paths
- **Directory Creation:** Creates directory if it doesn't exist

#### `--list`
- **Boolean Flag:** No value required, presence of flag enables list mode
- **Behavior:** Shows what would be downloaded without actually downloading
- **Use Case:** Preview files before committing to large downloads

## Output Files and Directory Structure

### Directory Structure

```
osdr_downloads_OSD-XXX/
└── measurement_technology/
    ├── GeneLab_processed_data_files/
    │   ├── GLDS-XXX_processed_file1.csv
    │   ├── GLDS-XXX_normalized_counts.csv
    │   └── GLDS-XXX_differential_expression.csv
    ├── raw_data_file1.tar.gz
    ├── raw_data_file2.fastq.gz
    └── metadata_file.txt
```

### Directory Naming Convention

- **Root Directory:** `osdr_downloads_OSD-XXX` (where XXX is the dataset number)
- **Measurement/Technology Subdirectories:** `measurement_technology` format
  - Spaces replaced with underscores
  - Hyphens replaced with underscores
  - Example: `transcription_profiling_RNA_Seq`

### File Organization

#### GeneLab Processed Data Files
**Location:** `*/GeneLab_processed_data_files/`

**Identification Criteria** (in priority order):
1. **Protocol Reference:** Files associated with "GeneLab * data processing protocol"
2. **File Category:** Files categorized as "GeneLab Processed * Files"
3. **Filename Patterns:** Files matching specific GeneLab output patterns
4. **Data Type:** Files with GeneLab-specific data types

**Common GeneLab File Types:**
- `GLDS-XXX_*_Unnormalized_Counts.csv` - Raw count matrices
- `GLDS-XXX_*_Normalized_Counts.csv` - Normalized expression data
- `GLDS-XXX_*_Differential_Expression.csv` - Differential expression results
- `GLDS-XXX_*_VST_Counts.csv` - Variance stabilized count data
- `GLDS-XXX_*_SampleTable.csv` - Sample metadata tables
- `GLDS-XXX_*_contrasts.csv` - Statistical contrast definitions

#### Raw Data Files
**Location:** `measurement_technology/` (root of measurement/tech directory)

**File Types:**
- **Raw Sequencing Data:** `.fastq.gz`, `.bam`, `.sam`
- **Archive Files:** `.tar.gz`, `.zip` containing multiple raw files
- **Microarray Data:** `.CEL`, `.txt` with raw intensity values
- **Mass Spectrometry:** `.raw`, `.mzML`, `.mgf`
- **Imaging Data:** `.tif`, `.png`, `.jpg`
- **Metadata Files:** ISA-Tab formatted metadata, protocol descriptions

### Output Summary Report

After each run, the tool generates a summary report showing:

```
============================================================
Summary Report:
============================================================
Dataset: OSD-101
Total files found: 25
GeneLab processed files: 8
Raw data files: 17
Files downloaded: 25
Failed downloads: 0
Output directory: osdr_downloads_OSD-101
GeneLab processed files saved to subdirectories: */GeneLab_processed_data_files

Applied filters:
  Measurement type: transcription profiling
  Technology type: RNA-Seq
  File extension (exclude): tar.gz
============================================================
```

### File Metadata Preservation

Each downloaded file retains:
- **Original filename** as stored in OSDR
- **File size** information in download logs
- **Data type** classification from OSDR metadata
- **Measurement/technology** association through directory structure

### Error Handling and Logs

**Download Logs:**
- `✓ Downloaded: filename.csv` - Successful downloads
- `✗ Failed to download: filename.csv` - Failed downloads with error details
- Alternative URL attempts for failed downloads

**Common Issues:**
- **API Connectivity:** Tests connection before starting downloads
- **Invalid File URLs:** Attempts alternative download URLs when primary fails
- **Permission Errors:** Reports directory creation or file write permissions issues
- **Network Timeouts:** Configurable timeout settings for large file downloads

### File Integrity

**Size Verification:** Downloaded file sizes are compared against metadata when available
**Format Preservation:** Files are downloaded in binary mode to preserve exact formatting
**No Compression:** Files are downloaded as-is without additional compression or decompression
