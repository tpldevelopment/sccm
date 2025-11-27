# SCCM

System Center Configuration Manager (SCCM) scripts, tools, and documentation for enterprise device management and deployment.

## Overview

This repository contains resources for managing and automating SCCM operations, including:

- PowerShell scripts for SCCM automation
- Custom reports and queries
- Configuration baselines
- Application deployment packages
- Operating system deployment (OSD) task sequences
- Compliance settings

## Getting Started

### Prerequisites

- SCCM Console installed
- PowerShell 5.1 or later
- Appropriate SCCM administrative permissions
- SCCM PowerShell module

### Installation

1. Clone this repository:
   ```powershell
   git clone https://github.com/bml104/sccm.git
   cd sccm
   ```

2. Import the SCCM PowerShell module:
   ```powershell
   Import-Module "$($ENV:SMS_ADMIN_UI_PATH)\..\ConfigurationManager.psd1"
   ```

## Project Structure

```
sccm/
├── scripts/          # PowerShell automation scripts
├── reports/          # Custom SCCM reports
├── queries/          # WQL queries for collections
├── baselines/        # Configuration baselines
├── applications/     # Application deployment resources
├── osd/             # Operating System Deployment files
└── docs/            # Documentation
```

## Usage

### Running Scripts

Most scripts require SCCM administrator privileges. Run PowerShell as administrator:

```powershell
# Example: Connect to SCCM site
cd "<SiteCode>:\"
```

### Best Practices

- Always test scripts in a development environment first
- Use proper error handling and logging
- Document any custom configurations
- Follow naming conventions for collections and packages
- Review change impact before deployment

## Common Tasks

### Creating Collections
```powershell
# Example collection creation script
New-CMDeviceCollection -Name "Windows 11 Devices" -LimitingCollectionName "All Systems"
```

### Deploying Applications
```powershell
# Example application deployment
New-CMApplicationDeployment -CollectionName "Target Collection" -Name "Application Name"
```

### Querying Devices
```powershell
# Get all devices
Get-CMDevice | Select-Object Name, LastLogonUserName, OperatingSystemNameandVersion
```

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Create a Pull Request

## Resources

- [Microsoft SCCM Documentation](https://docs.microsoft.com/en-us/mem/configmgr/)
- [ConfigurationManager PowerShell Module](https://docs.microsoft.com/en-us/powershell/module/configurationmanager/)
- [SCCM Community Hub](https://communityhub.microsoft.com/SCCM)

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues, questions, or contributions, please open an issue in this repository.

## Author

**bml104**

---

*Last updated: November 2025*
