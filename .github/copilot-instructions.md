# ZEN API & CZI Smart Microscopy Workshop - AI Coding Instructions

## Project Overview
This is a **smart microscopy workshop** focused on **ZEISS ZEN API integration** and **CZI (Carl Zeiss Image) file processing**. The codebase demonstrates real-time microscope control, deep learning model integration, and advanced image format conversion workflows.

## Architecture & Key Components

### 1. ZEN API Integration (`workshop/zen_api/`)
- **Real-time microscope control** via RESTful API to ZEN Blue 3.12 software
- **Configuration**: Copy `config_example.ini` to `config.ini` with your ZEN API Gateway settings
- **Core modules**: `zenapi_streaming.py` (pixel streaming), `zenapi_guidedacq.py` (automated acquisition)
- **Authentication**: Uses certificate-based auth with control tokens from ZEN Gateway

### 2. CZI File Processing (`workshop/czi_omezarr/`)
- **Multi-dimensional image data**: CZI files contain 5D+ data (X,Y,Z,T,C dimensions)
- **OME-ZARR conversion**: Convert CZI → OME-ZARR for cloud-native workflows
- **HCS Plate support**: High Content Screening with well/field organization
- **Key libraries**: `pylibCZIrw`, `czitools`, `ome-zarr`, `ngff-zarr`

### 3. Deep Learning Pipeline (`workshop/notebooks/`)
- **CZANN models**: ZEISS-specific format for AI models (`.czann` files)
- **Training workflow**: Label on arivis Cloud → train → export CZANN → use in ZEN/Python
- **Model conversion**: PyTorch/ONNX → CZANN using `czmodel` package
- **Inference**: Both ONNX and PyTorch inference pipelines available

### 4. Processing Utilities (`workshop/utils/`)
- **ArrayProcessor**: Core image processing class with filtering, thresholding, object detection
- **Path handling**: Use `sys.path.insert(0, str(workshop_dir))` pattern for imports
- **GPU acceleration**: CUDA support for deep learning inference

## Development Workflows

### Environment Setup
```bash
# Use provided conda environment
conda env create --file workshop/env_smartmic.yml
conda activate smartmic

# Alternative: pyproject.toml for pip users
pip install -e .
```

### ZEN API Development
1. **Start ZEN Blue 3.12** with API Gateway enabled
2. **Configure API**: Update `workshop/zen_api/config.ini` with host/port/certificate
3. **Test connection**: Use `zen_api_utils.misc.initialize_zenapi()` pattern
4. **Streaming workflow**: See `zenapi_streaming.py` for real-time pixel processing

### CZI Processing Patterns
```python
# Standard CZI reading pattern
from czitools.read_tools import read_tools
from czitools.metadata_tools.czi_metadata import CziMetadata

# Always get metadata first
metadata = CziMetadata(czi_file)
# Then read specific regions/dimensions
array = read_tools.read_czi_image(czi_file, ...)
```

### Jupyter Notebook Structure
- **Data loading**: Use relative paths from `workshop/notebooks/data/`
- **Model files**: Store in `workshop/notebooks/models/`
- **Processing**: Import from `../utils/` with sys.path manipulation

## Project-Specific Conventions

### File Organization
- **CZI sample data**: `workshop/data/*.czi` (multi-GB microscopy files)
- **AI models**: `.czann`, `.pt`, `.onnx` formats in respective directories
- **Configuration**: `.ini` files for ZEN API, `.yml` for conda environments

### Import Patterns
```python
# Always use this pattern in workshop subdirectories
workshop_dir = Path(__file__).parent.parent
sys.path.insert(0, str(workshop_dir))
from utils.processing_tools import ArrayProcessor
```

### Async Programming
- **ZEN API calls**: Use `asyncio` and `qasync` for non-blocking microscope control
- **Real-time processing**: PyQtGraph for live visualization during acquisition

### Data Handling
- **Large files**: CZI files can be multi-GB; use memory-mapped access via `pylibCZIrw`
- **Chunked processing**: Use `dask` for out-of-core processing of large images
- **Metadata-first**: Always parse CZI metadata before pixel data access

## Integration Points

### ZEN Software Integration
- **Pixel streaming**: Real-time data from microscope via gRPC streams
- **Experiment control**: Start/stop acquisitions, move stage, change settings
- **Model deployment**: Load CZANN models directly into ZEN for live inference

### Cloud Platforms
- **arivis Cloud**: For dataset labeling and model training
- **Google Colab**: Notebooks designed to run in Colab environment
- **Docker**: Models can be containerized for deployment

### External Tools
- **Napari**: Image viewer with plugin support for CZI and CZANN files
- **CZI ecosystem**: `CZICompress`, `CZICheck` CLI tools for file management

## Key Dependencies & Versions
- **Python 3.11+** (required for ZEN API compatibility)
- **PyTorch 2.5.1** (specific version for model compatibility)
- **CUDA support** optional but recommended for AI inference
- **ZEN Blue 3.12** required for API functionality

## Critical Files to Understand
- `workshop/zen_api/zenapi_streaming.py` - Real-time microscope control
- `workshop/utils/processing_tools.py` - Core image processing algorithms  
- `workshop/utils/ome_zarr_utils.py` - CZI → OME-ZARR conversion logic
- `workshop/notebooks/using_pylibCZIrw.ipynb` - Comprehensive CZI handling examples