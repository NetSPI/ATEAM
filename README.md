![ATEAMLogo](https://notpayloads.blob.core.windows.net/images/ATEAM.png) 
<br> 
[![licence badge]][licence] 
[![stars badge]][stars] 
[![forks badge]][forks] 
[![issues badge]][issues]
![Twitter Follow](https://img.shields.io/twitter/follow/netspi.svg?style=social)


[licence badge]:https://img.shields.io/badge/license-New%20BSD-blue.svg
[stars badge]:https://img.shields.io/github/stars/NetSPI/ATEAM.svg
[forks badge]:https://img.shields.io/github/forks/NetSPI/ATEAM.svg
[issues badge]:https://img.shields.io/github/issues/NetSPI/ATEAM.svg


[licence]:https://github.com/NetSPI/ATEAM/blob/master/LICENSE.txt
[stars]:https://github.com/NetSPI/ATEAM/stargazers
[forks]:https://github.com/NetSPI/ATEAM/network
[issues]:https://github.com/NetSPI/ATEAM/issues



# ATEAM: Azure Tenant Enumeration and Attribution Module

A Python reconnaissance tool designed to discover Azure services and attribute tenant ownership information based on their responses. 

### Author, Contributors, and License
* Authors:
 	* Karl Fosaaen ([@kfosaaen](https://twitter.com/kfosaaen)), NetSPI
 	* Thomas Elling ([@thomaselling](https://twitter.com/thomas_elling)), NetSPI
* License: BSD 3-Clause

## 🎯 Purpose

This tool helps security researchers, penetration testers, and Azure administrators discover Azure resources associated with specific tenant IDs. 

## ✨ Features

### **Multi-Service Discovery**
Tests resource names against 6 different Azure services and their subdomains:
- **Azure App Services** (`.azurewebsites.net` and `.scm.azurewebsites.net`)
- **Azure DevOps** (`dev.azure.com`)
- **Azure Key Vault** (`.vault.azure.net`)
- **Azure Storage Accounts** (`.blob.core.windows.net`)
- **SharePoint Online** (`.sharepoint.com`)
- **Azure Databricks** (`.azuredatabricks.net`)

### **Additional Capabilities**
- **Concurrent Processing**: Multi-threaded scanning for faster results
- **Batch Processing**: Processes resources in configurable batches to prevent memory issues
- **DNS Validation**: Confirms DNS records before making HTTP requests
- **Permutation Generation**: Automatically generates variations of resource names
- **Database Storage**: SQLite database for persistent results
- **Multiple Export Formats**: CSV, JSON, and HTML exports
- **Verbose Logging**: Detailed debug information for troubleshooting


## 📋 Prerequisites

- Python 3.7 or higher
- Internet connection
- Required Python packages (see Installation below)

## 🚀 Installation

1. **Clone or download the tool:**
   ```bash
   git clone https://github.com/NetSPI/ATEAM.git
   cd ATEAM
   ```

2. **Install required dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

   Or install manually:
   ```bash
   pip install requests dnspython urllib3
   ```

3. **Verify installation:**
   ```bash
   python ateam.py --help
   ```

## 📖 Usage

### Basic Usage

**Scan a single resource:**
```bash
python ateam.py -r "myapp"
```

**Scan multiple resources:**
```bash
python ateam.py -r "app1" "app2" "app3"
```

**Scan from a text file resource list:**
```bash
python ateam.py -f resources.txt
```

### Advanced Usage

**Verbose output with (20) workers:**
```bash
python ateam.py -f resources.txt -v -w 20
```

**Generate permutations and scan:**
```bash
python ateam.py -f resources.txt -p
```

**Generate permutations with smaller batch size (recommended for large scans):**
```bash
python ateam.py -f resources.txt -p -b 100
```

**Export results to HTML:**
```bash
python ateam.py -e html
```

**Clear database and start fresh:**
```bash
python ateam.py -f resources.txt --clear
```

### Command Line Options

| Option | Description | Example |
|--------|-------------|---------|
| `-f, --file` | File containing resources (one per line) | `-f resources.txt` |
| `-r, --resources` | Space-separated list of resources | `-r "app1" "app2"` |
| `-w, --workers` | Number of concurrent workers (default: 10) | `-w 20` |
| `-b, --batch-size` | Resources per batch (default: 1000) | `-b 100` |
| `-v, --verbose` | Enable verbose logging | `-v` |
| `-l, --list` | List all database entries | `-l` |
| `-e, --export` | Export results (csv, json, html) | `-e html` |
| `--clear` | Clear database before scanning | `--clear` |
| `-t, --tenant` | Filter results by tenant ID | `-t "tenant-id"` |
| `-p, --permutations` | Generate resource name permutations | `-p` |

## 📁 File Structure

```
ATEAM/
├── ateam.py    # Main script
├── requirements.txt          # Python dependencies
├── permutations.txt          # Resource name permutations
└── README.md                 # This file
```

## 🔧 Configuration

### Permutations File
The `permutations.txt` file contains common prefixes and suffixes to generate variations of resource names:

```
dev
prod
test
staging
api
web
...
```

This will generate combinations like:
- `devmyapp`
- `myappdev`
- `prodmyapp`
- `myapp-prod`
- etc.

### Database
Results are stored in `azure_tenants.db` (SQLite) with the following schema:
- `resource_uri`: The discovered resource URL
- `resource_type`: Type of Azure service
- `tenant_id`: Extracted tenant ID
- `discovered_at`: Timestamp of discovery

## 📊 Output Examples

### Console Output
```
2025-06-27 14:37:39 - INFO - Found Storage Account tenant ID 72f988bf-86f1-41af-91ab-2d7cd011db47 for mars
2025-06-27 14:37:40 - INFO - Found App Service SCM tenant ID 'common' for notarealapplication
2025-06-27 14:37:47 - INFO - Found Key Vault tenant ID 72f988bf-86f1-41af-91ab-2d7cd011db47 for mdo
```

### Database Listing
```
Resource URI                             Type            Tenant ID                      	  Discovered At
-----------------------------------------------------------------------------------------------------------------
hanover.scm.azurewebsites.net            AppServices-SCM 72f988bf-86f1-41af-91ab-2d7cd011db47 2025-06-27 21:37:41
dev.azure.com/microsoft                  DevOps          72f988bf-86f1-41af-91ab-2d7cd011db47 2025-06-27 21:37:12
mdo.vault.azure.net                      KeyVault        72f988bf-86f1-41af-91ab-2d7cd011db47 2025-06-27 21:37:47
mars.blob.core.windows.net               StorageAccount  72f988bf-86f1-41af-91ab-2d7cd011db47 2025-06-27 21:37:46
```

## 🔍 How It Works

1. **DNS Validation**: Checks if DNS records exist for the resource
2. **Service Probing**: Makes HTTP requests to the applicable Azure service endpoints
3. **Response Analysis**: Depending on the resource, extracts tenant IDs from:
   - WWW-Authenticate headers
   - OAuth redirect URLs
   - Response headers
   - Error messages
4. **Data Storage**: Saves results to SQLite database
5. **Export**: Generates reports in various formats

## 🛡️ Security Considerations

- **Rate Limiting**: The tool includes delays and respects service limits
- **Error Handling**: Graceful handling of timeouts and errors
- **Logging**: Detailed logs for audit trails
- **Non-Intrusive**: Uses standard HTTP requests without authentication to do anonymous enumeration

**⚠️ Important**: Always ensure you have proper authorization before scanning any resources. Unauthorized scanning may violate terms of service or applicable laws.

## 🐛 Troubleshooting

### Debug Mode
You can enable verbose logging for detailed information, but it can be a bit much:
```bash
python ateam.py -f resources.txt -v
```

### Memory Issues with Permutations
If you experience "killed" messages when using permutations, try reducing the batch size:
```bash
python ateam.py -f resources.txt -p -b 100 -w 5
```

This processes resources in smaller batches with fewer concurrent workers to prevent memory exhaustion.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit:
- Bug reports
- Feature requests
- Code improvements
- Documentation updates


### Related Blogs
* <a href="https://blog.netspi.com/a-beginners-guide-to-gathering-azure-passwords/">A Beginners Guide to Gathering Azure Passwords</a>

### Presentations
* <a href="https://youtu.be/CUTwkuiRgqg">We Know What You Did (in Azure) Last Summer - DEF CON 33 - Cloud Village</a>
  - <a href="https://notpayloads.blob.core.windows.net/slides/ExtractingalltheAzurePasswords.pdf">Slides</a>
