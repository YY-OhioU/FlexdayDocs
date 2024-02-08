# Donut

## Installation

Donut can be installed in either an conda environment or in a naive python environment. It is recommended to install Donut in a virtualized environment because some of its dependencies are older.

- dependencies
  
  Donut needs to work with these specific versions of dependencies. The weights in pretrained models have the same shape with the models implemented in these exact versions.
  
  - Install `Pytorch`:  Tested version (2.1.0)
    
    Follow the instructions on the official website of PyTorch to install PyTorch to the environment.
  
  - Other dependencies:
    
    ```
    pip install transformers==4.25.1
    pip install pytorch-lightning==1.6.4
    pip install timm==0.5.4
    pip install gradio
    ```

- Donut
  
  - Download the provided Donut code
    
    There are some changes made to the original Donut code. 
    
    Details of the modifications can be found in this Github Issue: [https://github.com/clovaai/donut/issues/175](https://github.com/clovaai/donut/issues/175)
  
  - Install Donut as a python library
    
    ```bash
    cd donut && pip install . 
    ```

- dependencies fix:
  
  The `pytorch-lightning` required by Donut is compatible with the older version of pytorch, which will cause the program to crash during training. Therefore, it is necessary to modify the `pytorch-lightning` package installed in the environment for this project.  [https://github.com/clovaai/donut/issues/255](https://github.com/clovaai/donut/issues/255)
  
  - Fix problem during training:
    
    - Find file in site-packages in current environment: ```pytorch_lightning/utilities/types.py```
    
    - In this file locate
      
      ```python
      LRSchedulerTypeTuple = (torch.optim.lr_scheduler._LRScheduler, torch.optim.lr_scheduler.ReduceLROnPlateau)
      ```
    
    - Change this line to
      
      ```python
      LRSchedulerTypeTuple = (torch.optim.lr_scheduler.LRScheduler, torch.optim.lr_scheduler.ReduceLROnPlateau)
      ```

## Usage


