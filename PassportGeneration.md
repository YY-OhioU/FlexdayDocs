# Passport Generation

## Installation

### dependencies

`pip install -r requirements.txt`

### Find source code

`cd Flexday_deliver/Passport-Generator`

## Usage

### Generate images and annotations

Run `main.py` with desired arguments.

Code will generate images with annotations.
Annotations will be organized in a json lines format in `ground_truth.jsonl`

```plaintext
usage: python main.py [-h] [-n NUMBER] [-o OUTPUT] [-b] [-s] [-r] [-p]

Generate fake passport images

options:
  -h, --help            show this help message and exit
  -n NUMBER, --number NUMBER
                        num of images
  -o OUTPUT, --output OUTPUT
                        output directory
  -b, --bbox            draw bounding boxes
  -s, --scaling         Scale image
  -r, --rotation        Rotation
  -p, --projection      Apply projection

```

`OUTPUT` folder needs to be created before the execution of this script

### Convert annotations to Donut format

The `main.py` dumps data into one folder. But Donuts requires data to be split into seperate folders. A simple way is to call `main.py` three times to generate data in different folders.

Also, the generated annotations don't meet the Donut's requirements.  `gt2donut.py` is designed to convert the annotation file to the Donut format.

- Create folders
  
  ```shell
  mkdir dataset_root
  cd dataset_root
  mkdir train test validation
  ```

- Generate data
  
  ```shell
  python main.py -n <number of samples> -o ../donut_solution/datasets/sample_dataset/train -sr
  python main.py -n <number of samples> -o ../donut_solution/datasets/sample_dataset/test -sr
  python main.py -n <number of samples> -o ../donut_solution/datasets/sample_dataset/validation -sr
  ```

- Convert `ground_truth.jsonl` to Donut `metadata.jsonl`
  
  ```shell
  python gt2donut -f ../donut_solution/datasets/sample_dataset/train
  python gt2donut -f ../donut_solution/datasets/sample_dataset/test
  python gt2donut -f ../donut_solution/datasets/sample_dataset/validation
  ```
