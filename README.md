# KeyPoint Extraction Server For Insurance Certificates

Processing Server based on the modified version of [PICK model](https://github.com/wenwenyu/PICK-pytorch) from the ["PICK: Processing Key Information Extraction from Documents using Improved Graph 
Learning-Convolutional Networks"](https://arxiv.org/abs/2004.07464) paper. 


## Key modifications

- The Batch Normalization layers in the ResNet model was changed on [Group Normalization layers](https://arxiv.org/pdf/1803.08494.pdf).
- The final convolution block in the ResNet with block expansion and number of features = output_channels was removed.
- The prosses of encoding image segments to vector representation was changed: the output of RoIAlign algorithm is prosessed by stack of linear layers.
- Instead of summation the text and image segments embeddings were concatenated and projected to dimension = out_dim.
- Normalization of input images was performed using precalculated statistics for train dataset.
- Changes in the model configuration: embedding_dim = 512; out_dim = 512; feedforward_dim = 1024; image_encoder = "resnet34", nheaders = 8.
- The training was performed on single GPU using batch size 2 and stepwise learning scheduler with gamma 0.5.
- The number of fully correctly recognized documents was used as a main monitoring metric during training.
- Now train dataloader supports batch bucketing technique (see data_utils/batch_sampler.py and config.json).
- Now encoder block supports SE ResNet architectures (see model/resnet.py and model/encoder.py)
- The gradient clipping was added to training loop.

## Model performance

- The model achieves the value of mean F1 score 95.7 % on validation data.
- 50.3 % of the validation documents were recognized fully correct.
- 88.6 % of the validation documents were recognized with F1 score > 90 %.
- 95.2 % of the validation documents were recognized with F1 score > 80 %.

## Requirements
* python = 3.6 
* torchvision = 0.6.1
* tabulate = 0.8.7
* overrides = 3.0.0
* opencv_python = 4.3.0.36
* numpy = 1.16.4
* pandas = 1.0.5
* allennlp = 1.0.0
* torchtext = 0.6.0
* tqdm = 4.47.0
* torch = 1.5.1
* pytesseract = 0.3.8
* urllib3 = 1.26.4
* flask = 1.1.2
```bash
sudo apt-get install tesseract-ocr
pip install -r requirements.txt
```
Note, server requires weights of the pretrained PICK model saved as 'pretrained_model/pretrained.pth'.

### Usage
Run server:
```bash
python get_entities_server.py -d cpu
```
Use '-d' to specify device for calculations ('cpu' or 'cuda').

Request processing:
```bash
python request_processing.py -i input_examples/t0_sf_c1_0_0.jpg -o same
```
Use '-i' to specify path for input image and '-o' for output JSON file. Use '-o same' to save output file to same directory and name as input file.

### Example
Input image:
<div align="center">
  <img src="input_examples/t0_sf_c1_0_0.jpg" width="400" />
</div>

Output JSON file:
```bash
{
    "Type": "declaration of insurance",
    "Address": "1019 bristol ct johns creek ga",
    "Zip Code": "30022",
    "First Name": "venkata usha",
    "Last Name": "dammala",
    "Additionall Interest": "arium johns creek po box 3976 albany ny 12203",
    "Policy Number": "81-gb-c127-3",
    "Carrier": "state farm fire casualty company",
    "Liability Limit": "100000",
    "Premium": "",
    "Effective Date": "08/01/2020",
    "Expiration Date": "08/01/2021",
    "Print Date": "09/21/2020"
}
```
