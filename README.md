# IFC Embodied Carbon Calculator

A comprehensive Python tool for calculating embodied carbon emissions from IFC (Industry Foundation Classes) building models. This tool extracts geometric and material properties from IFC files and calculates embodied carbon using configurable carbon factor databases.

## 🌟 Features

- **Comprehensive IFC Support**: Processes walls, slabs, beams, columns, roofs, stairs, doors, windows, and more
- **Intelligent Property Extraction**: Automatically extracts volumes, areas, and material properties from IFC elements
- **Flexible Carbon Factor Matching**: Uses fuzzy matching algorithms to find appropriate carbon factors
- **Material Detection**: Attempts to identify materials from IFC associations with fallback to type-based mapping
- **Robust Error Handling**: Gracefully handles missing data and continues processing
- **Detailed Reporting**: Generates comprehensive reports with summaries by material and element type
- **Multiple Export Formats**: Exports detailed results and summary statistics to CSV
- **Google Colab Ready**: Works seamlessly in Google Colab with file upload interface

## 📋 Requirements

### Python Dependencies
```python
ifcopenshell>=0.6.0
pandas>=1.3.0
numpy>=1.20.0
```

### System Requirements
- Python 3.7 or higher
- Sufficient RAM for processing large IFC files (typically 4GB+ recommended)

## 🚀 Installation

### Option 1: Google Colab (Recommended for beginners)
```python
!pip install ifcopenshell pandas numpy
```

### Option 2: Local Installation
```bash
pip install ifcopenshell pandas numpy
```

### Option 3: Using conda
```bash
conda install -c conda-forge ifcopenshell pandas numpy
```

## 📊 Carbon Factors Database Format

Create a CSV file with the following columns:

| Column | Description | Example |
|--------|-------------|---------|
| `ifc_type` | IFC element type | IfcWall |
| `material` | Material name | concrete |
| `unit` | Unit of measurement | m3 |
| `carbon_factor (kgCO2e/unit)` | Carbon factor value | 315.5 |

### Example `carbon_factors.csv`:
```csv
ifc_type,material,unit,carbon_factor (kgCO2e/unit)
IfcWall,concrete,m3,315.5
IfcSlab,concrete,m3,315.5
IfcBeam,steel,kg,1.46
IfcColumn,steel,kg,1.46
IfcRoof,concrete,m3,315.5
IfcDoor,timber,kg,0.72
IfcWindow,aluminum,kg,8.24
```

## 🔧 Usage

### Google Colab Usage
```python
# Install dependencies
!pip install ifcopenshell pandas

# Run the calculator
from ifc_carbon_calculator import main
main()
```

### Standalone Python Usage
```python
from ifc_carbon_calculator import IFCCarbonCalculator

# Initialize calculator
calculator = IFCCarbonCalculator('carbon_factors.csv')

# Load IFC file
calculator.load_ifc_file('building_model.ifc')

# Calculate embodied carbon
results = calculator.calculate_embodied_carbon()

# Export results
calculator.export_results(results, 'my_carbon_report.csv')
```

### Advanced Usage
```python
# Process specific element types only
specific_elements = ['IfcWall', 'IfcSlab', 'IfcBeam']
results = calculator.calculate_embodied_carbon(specific_elements)

# Generate summary report
summary = calculator.generate_summary_report(results)
print(f"Total embodied carbon: {summary['total_embodied_carbon']:.2f} kgCO2e")
```

## 📈 Supported IFC Elements

| IFC Type | Default Material | Default Density | Preferred Unit |
|----------|------------------|-----------------|----------------|
| IfcWall | concrete | 2400 kg/m³ | m³ |
| IfcSlab | concrete | 2400 kg/m³ | m³ |
| IfcBeam | steel | 7850 kg/m³ | kg |
| IfcColumn | steel | 7850 kg/m³ | kg |
| IfcRoof | concrete | 2400 kg/m³ | m³ |
| IfcStair | concrete | 2400 kg/m³ | m³ |
| IfcRailing | steel | 7850 kg/m³ | kg |
| IfcDoor | timber | 600 kg/m³ | kg |
| IfcWindow | aluminum | 2700 kg/m³ | kg |
| IfcBuildingElementProxy | concrete | 2400 kg/m³ | m³ |

## 🔍 How It Works

### 1. **Property Extraction**
The calculator extracts geometric properties from IFC elements using multiple strategies:
- **Primary**: NetVolume and GrossVolume from property sets
- **Secondary**: Volume from quantity sets
- **Fallback**: Calculate volume from length × width × height
- **Material Detection**: Attempts to identify materials from IFC associations

### 2. **Quantity Calculation**
- Prioritizes NetVolume over GrossVolume for accuracy
- Converts volumes to mass when carbon factors are specified per kg
- Uses element-specific density values for conversion

### 3. **Carbon Factor Matching**
The calculator uses a hierarchical matching system:
1. **Exact Match**: IFC type + material + unit
2. **Material + Unit Match**: Any element with matching material and unit
3. **Material Only Match**: Any element with matching material
4. **No Match**: Uses 0.0 as carbon factor and logs warning

### 4. **Results Generation**
- Calculates embodied carbon = quantity × carbon factor
- Generates detailed element-by-element results
- Creates summary statistics by material and IFC type

## 📊 Output Files

### 1. **Detailed Results** (`carbon_report.csv`)
Contains individual element calculations:
- Element GUID and Name
- IFC Type and Material
- Quantity Used and Unit
- Carbon Factor
- Embodied Carbon (kgCO2e)

### 2. **Summary Report** (`carbon_report_summary.csv`)
Contains high-level statistics:
- Total elements processed
- Total embodied carbon
- Elements without carbon factors

### 3. **Console Output**
Real-time progress and final summary including:
- Processing status for each element type
- Summary statistics
- Breakdown by material and IFC type

## 🎯 Best Practices

### IFC File Preparation
1. **Quality Check**: Ensure IFC file contains proper quantity information
2. **Material Assignment**: Assign materials to elements in your BIM software
3. **Property Sets**: Include standard property sets (Pset_WallCommon, etc.)

### Carbon Factors Database
1. **Comprehensive Coverage**: Include factors for all materials in your model
2. **Consistent Units**: Use consistent units (m³ for volume-based, kg for mass-based)
3. **Source Documentation**: Document carbon factor sources and assumptions
4. **Regular Updates**: Keep factors updated with latest environmental data

### Performance Optimization
1. **Large Files**: For files >100MB, consider processing in batches
2. **Memory Management**: Close unnecessary applications when processing large models
3. **Element Filtering**: Process only necessary element types for faster execution

## 🛠️ Troubleshooting

### Common Issues

#### "No module named 'ifcopenshell'"
```bash
pip install ifcopenshell
```

#### "IFC file not found"
- Verify file path is correct
- Ensure file extension is .ifc
- Check file permissions

#### "No carbon factors found"
- Verify CSV format matches expected columns
- Check for typos in material names
- Ensure units are consistent

#### "Zero embodied carbon results"
- Check if IFC elements contain volume/quantity information
- Verify carbon factors database has matching entries
- Review material assignments in BIM model

### Performance Issues
- **Large Files**: Process in smaller batches or upgrade system RAM
- **Slow Processing**: Check IFC file quality and complexity
- **Memory Errors**: Increase virtual memory or use cloud computing

## 🔄 Extending the Calculator

### Adding New Element Types
```python
# Add to element_mapping dictionary
self.element_mapping["IfcNewType"] = {
    "material": "new_material",
    "density": 1500,
    "unit_preference": "m3"
}
```

### Custom Material Detection
```python
def custom_material_detector(self, element):
    # Your custom material detection logic
    return material_name, density
```

### Adding New Export Formats
```python
def export_to_json(self, df, output_path):
    df.to_json(output_path, orient='records', indent=2)
```

## 📚 References

### Standards and Guidelines
- **ISO 14040/14044**: Life Cycle Assessment principles
- **EN 15978**: Sustainability assessment of construction works
- **IFC4**: Industry Foundation Classes standard

### Carbon Factor Sources
- **RICS Professional Standards**: Embodied carbon databases
- **ICE Database**: Inventory of Carbon & Energy
- **EPD Databases**: Environmental Product Declarations
- **National Standards**: Country-specific carbon factors

### IFC Resources
- **buildingSMART**: Official IFC documentation
- **IfcOpenShell**: IFC processing library documentation
- **IFC Examples**: Sample IFC files for testing

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork the Repository**
2. **Create Feature Branch**: `git checkout -b feature/new-feature`
3. **Make Changes**: Follow existing code style
4. **Add Tests**: Include unit tests for new functionality
5. **Update Documentation**: Update README and code comments
6. **Submit Pull Request**: With detailed description

### Development Setup
```bash
git clone https://github.com/your-repo/ifc-carbon-calculator
cd ifc-carbon-calculator
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

### Getting Help
- **Issues**: Report bugs and request features on GitHub
- **Documentation**: Check this README and code comments
- **Community**: Join discussions in GitHub Discussions

### Professional Support
For commercial support, custom development, or consulting services, please contact [8066@cch.edu.pk].

## 🙏 Acknowledgments

- **IfcOpenShell**: For excellent IFC processing capabilities
- **buildingSMART**: For IFC standard development
- **Contributors**: All contributors to this project
- **Research Community**: For embodied carbon research and databases

## 📋 Changelog

### Version 1.0.0
- Initial release with comprehensive IFC support
- Intelligent carbon factor matching
- Detailed reporting capabilities
- Google Colab integration

---

**Note**: This tool is for educational and research purposes. Always verify results with professional environmental assessment tools and consult with LCA experts for critical applications.# Extracting-Embodied-Carbon-from-Structures-from-IFC-file
