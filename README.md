
# Use DICOMweb™ Standard APIs with Python

This tutorial uses Python to demonstrate working with the Medical Imaging Server for DICOM.

## load-dicom-samples

This script loads a collection of samples from specific studies and sends them to the DICOM server a batch at a time.


### Import the appropriate Python libraries
import os
import requests
import pydicom
from pathlib import Path
from urllib3.filepost import encode_multipart_formdata, choose_boundary


#### Install pydicom

If you're missing the pydicom library it can easily be installed using PIP.

```python
pip install pydicom


### Configure user-defined variables to be used throughout
Replace all variable values wrapped in { } with your own values. Additionally, validate that any constructed variables are correct.  For instance, `base_url` is constructed using the default URL for the service. If you're using a custom URL, you'll need to override that value with your own.

```python
dicom_server_name = "{server-name}"
path_to_dicoms_dir = "{path to the folder that includes green-square.dcm and other dcm files}"

base_url = f"https://{dicom_server_name}.azurewebsites.net"

### Reading in the files

My input dicom files were grouped by subdirectory.  This section reads each directory and then gets the files inside them.  Each file is
read, a tuple created and placed into a dictionary.

```python
dirEntries = os.listdir(path_to_dicoms_dir)
for dir in dirEntries:
    fileEntries = os.listdir(Path(path_to_dicoms_dir).joinpath(dir))
    files = {}
    for file in fileEntries:
        filepath = Path(path_to_dicoms_dir).joinpath(dir).joinpath(file)
        print("Working on "+filepath.name)
        with open(filepath, 'rb') as reader:
            rawfile = reader.read()            
            files[filepath.name] = ('dicomfile', rawfile, 'application/dicom')


Then we encoded all the files together and send up to the DICOM server in batches by subdirectory.

```python
    #encode as multipart_related
    body, content_type = encode_multipart_related(fields = files)

    headers = {'Accept':'application/dicom+json', "Content-Type":content_type}

    url = f'{base_url}/studies'
    response = client.post(url, body, headers=headers, verify=False)

    print('response:', response)