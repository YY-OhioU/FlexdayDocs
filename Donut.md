# Donut

## Installation

Donut can be installed in either a conda environment or in a naive Python environment. It is recommended to install Donut in a virtualized environment because some of its dependencies are older.

### dependencies

  Donut needs to work with these specific versions of dependencies. The weights in pre-trained models have the same shape as the models implemented in these exact versions.

- Pytorch:  Tested version (2.1.0)   
   Follow the instructions on the official website of PyTorch to install PyTorch to the environment.

- Other dependencies:
  
  ```
  pip install transformers==4.25.1
  pip install pytorch-lightning==1.6.4
  pip install timm==0.5.4
  pip install gradio
  ```

### Donut

- Find the source code of Donut
  Unzip the zip package. The source code is in the directory: `Flexday_deliver/donut_solution/donut` (the folder with `setup.py`)
  
  There are some changes made to the original Donut code. 
  
  Details of the modifications can be found in this GitHub Issue: [https://github.com/clovaai/donut/issues/175](https://github.com/clovaai/donut/issues/175)

- Install Donut as a Python library
  
  ```bash
  cd Flexday_deliver/donut_solution/donut
  pip install . 
  ```

- dependencies fix:
  
  The `pytorch-lightning` required by Donut is compatible with the older version of pytorch, which will cause the program to crash during training. Therefore, it is necessary to modify the `pytorch-lightning` package installed in the environment for this project.  [https://github.com/clovaai/donut/issues/255](https://github.com/clovaai/donut/issues/255)
  
  - Fix problems during training:
    
    - Find the file in `site-packages` in the current environment: `pytorch_lightning/utilities/types.py`
      
      - `site-packages` is the directory where packages in a Python environment are installed.
    
    - In this file locate
      
      ```python
      LRSchedulerTypeTuple = (torch.optim.lr_scheduler._LRScheduler, torch.optim.lr_scheduler.ReduceLROnPlateau)
      ```
    
    - Change this line to
      
      ```python
      LRSchedulerTypeTuple = (torch.optim.lr_scheduler.LRScheduler, torch.optim.lr_scheduler.ReduceLROnPlateau)
      ```

## Usage

### Necessary folders:

There is no requirement on the names of the folders as long as the names are consistent with the names used in the commands and YAML files. But for the sake of simplicity, this manual will use the name listed below:

- donut: Source code

- config_folder: Holds all the config .yaml files. This folder will be referred to as `config_folder` in this manual

- dataset_root: Described in the next section. The root folder of datasets for training, testing, and validation. (referred to as `dataset_root`)
  
- In the package we provide, there's a dataset folder located at `datasets/sample_dataset`

- result: Folder containing trained models and logs. (`result`)

- OU_model: This folder will be provided along with the source code. It holds the models trained by the OU team. (`pretrained_model`)

### Dataset

This repository assumes the following structure of the dataset:

```bash
> tree sample_dataset
sample_dataset
├── test
│   ├── metadata.jsonl
│   ├── {image_path0}
│   ├── {image_path1}
│             .
│             .
├── train
│   ├── metadata.jsonl
│   ├── {image_path0}
│   ├── {image_path1}
│             .
│             .
└── validation
    ├── metadata.jsonl
    ├── {image_path0}
    ├── {image_path1}
              .
              .

> cat dataset_name/test/metadata.jsonl
{"file_name": {image_path0}, "ground_truth": "{\"gt_parse\": {ground_truth_parse}, ... {other_metadata_not_used} ... }"}
{"file_name": {image_path1}, "ground_truth": "{\"gt_parse\": {ground_truth_parse}, ... {other_metadata_not_used} ... }"}
     .
     .
```

- The structure of `metadata.jsonl` file is in `JSON Lines text format`([https:://jsonlines.org](https://jsonlines.org)), i.e., `.jsonl`. Each line consists of
  
  - `file_name`: relative path to the image file.
  
  - `ground_truth`: string format (JSON dumped), the dictionary contains `gt_parse`.
    
    Sample code to generate one line in metadata.jsonl:
    
    ```python
    import json
    
    with open("metadata.jsonl", "w") as f:
        gt_dict = {"gt_parse": {"key_1": "val_1", "key_2": "val_2"}}
        gt_str = json.dumps(gt_dict)
        line_dict = {"file_name": "img0.png", "ground_truth": gt_str}
        f.write(json.dumps(line_dict))
    ```
    
    - Example of one row in metadata.jsonl: 
      
      ```json
      {"file_name": "img0.png", 
      "ground_truth": "{\"gt_parse\": {\"p_id\": \"passport_id\", \"first_name\": \"First\", \"last_name\": \"Last\", \"dob\": \"DateOfBirth\", \"date_of_issue\": \"DateIssued\", \"place_of_birth\": \"PlaceOfBirth\", \"valid_through\": \"ExpireDate\", \"gender\": \"Gender\"}}"}
      ```

- `donut` interprets all tasks as a JSON prediction problem. As a result, all `donut` model training share the same pipeline. For training and inference, the only thing to do is to prepare `gt_parse` or `gt_parses` for the task in the format described below.

### Training

- Edit the config file
  
  `sample_config.yaml` is an example yaml file to train from Donut's official pre-trained model.
  
  Place `sample_config.yaml` under `config_folder` 
  
  ```yaml
  resume_from_checkpoint_path: null # only used for resume_from_checkpoint option in PL
  result_path: "../result/"
  pretrained_model_name_or_path: "naver-clova-ix/donut-base-finetuned-cord-v2" # loading a pre-trained model from HuggingFace(official pre-trained Donut model)
  # pretrained_model_name_or_path: "../OU_model" # loading a pre-trained model (pre-trained on passport dataset)
  dataset_name_or_paths: ["../datasets/sample_dataset/"] # loading datasets (from HuggingFace or path)
  sort_json_key: False # cord dataset is preprocessed, and publicly available at https://huggingface.co/datasets/naver-clova-ix/cord-v2
  train_batch_sizes: [8]
  val_batch_sizes: [1]
  # input_size: [1075, 761]
  input_size: [1280, 960 ] # When the input resolution differs from the pre-training setting, some weights will be newly initialized (but the model training would be okay)
  max_length: 768
  align_long_axis: False
  num_nodes: 1
  seed: 2022
  lr: 3e-5
  warmup_steps: 300 # 800/8*30/10, 10%
  num_training_samples_per_epoch: 800
  max_epochs: 100
  max_steps: -1
  num_workers: 8
  val_check_interval: 1.0
  check_val_every_n_epoch: 2
  gradient_clip_val: 1.0
  verbose: False  # Set to True if detailed predictions and ground truth during validation are wanted.
  ```

- Train
  
  In the `donut` folder, execute the following commands
  
  `python train.py --config ../config_folder/sample_config.yaml  --exp_version "version_1" --task_name donut_passport`
  
  After training, you can find the trained model and the logs in `<path to result>/<config_name>/<exp_version>`. In our example, the folder is `Flexday_deliver/donut_solution/result/sample_config/version_1`

- Testing
  
  In the `donut` folder, execute the following commands
  
  `python test.py --pretrained_model_name_or_path result/sample_config/version_1 --dataset_name_or_path <path to dataset> --task_name donut_passport --save_path <file to save>`
  
  `file to save` should point to a regular text file to which the results are written. The script will create the file if the file doesn't exist.
  
  Then the report for testing can be found in `<file to save>`
  
  By default, testing is performed on the test dataset, but you can change it to `train` or `validation` by adding an extra argument `-split`.  Eg: add `-split train` to test on `dataset_root/train` 

- Process a single image:
  
  `python inference_one.py --pretrained_path <pretrained model path> --input <input image path>`
  
 If the trained model needs to be used as a checkpoint, the `results` folder can be used as a `<pretrained model path>` for future training.
  
  - This example uses the model pre-trained by OU team to process an image:
    
    `python inference_one.py --pretrained_path ../OU_model --input<input image path>`

### Notice
- `task_name` and `task_start_token`

  Donut network's language decoder needs a special token as a prompt. This token can be anything as long as it is consistent during training and testing.

  Both `train.py` and `test.py` assemble the start token from the `task_name` variable read from the command line.

  Currently, the model is trained with the `task name` being ***donut_passport***


