# Preprocessing scripts
We provide scripts to perform image registration. Ths scripts are put in the folder named `preprocessing_tools`. 

Codes are modified from this article: https://www.sicara.fr/blog-technique/2019-07-16-image-registration-deep-learning

The scripts work in most cases. But there exists cases where the scripts failed completely or generate very bad result.

## File structure
```text
preprocessing_tools/
├── passport_tmp.png
├── registration_dataset.py
└── registration_one.py

```
## Usage
### Batch operation
`registration_dataset.py` finds all the *.png* files in the `SRC_FOLDER` and saves the registered images to `DST_FOLDER` with the same name 
```text
  usage: registration_dataset.py [-h] [-t TMP_PATH] [-s SRC_FOLDER] [-d DST_FOLDER]

   options:
     -h, --help     show this help message and exit
     -t TMP_PATH    template path
     -s SRC_FOLDER  image source folder
     -d DST_FOLDER  destination folder
```
**Need to copy the metadata.jsonl file to the DST_FOLDER manually**

### Process one image
`registration_one.py`
```text
usage: registration_one.py [-h] [-t TMP_PATH] [-s SRC] [-d DST]

options:
  -h, --help   show this help message and exit
  -t TMP_PATH  template path
  -s SRC       image to be registered
  -d DST       output file
```

### Template file
`passport_tmp.png` is the template image. The registration operation will scale and rotate the input images to match with this image.
By default, the `-t` arguments of both scripts point to this file.