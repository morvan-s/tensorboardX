[![Build Status](https://travis-ci.org/lanpa/tensorboard-pytorch.svg?branch=master)](https://travis-ci.org/lanpa/tensorboard-pytorch)
[![PyPI version](https://badge.fury.io/py/tensorboardX.svg)](https://badge.fury.io/py/tensorboardX)
[![Downloads](https://img.shields.io/badge/pip--downloads-5K+-brightgreen.svg)](https://bigquery.cloud.google.com/savedquery/966219917372:edb59a0d70c54eb687ab2a9417a778ee)
# tensorboard-pytorch

Write tensorboard events with simple command.

including scalar, image, histogram, audio, text, graph and embedding.

see [demo](http:35.197.26.245:6006) (result of `demo.py` and some images generated by BEGAN)

## Install

`#tested on anaconda2/anaconda3, pytorch 0.2, torchvision 0.1.9`

`pip install tensorboardX`
`pip install tensorflow` (for tensorboard web server)

or build from source:
`pip install git+https://github.com/lanpa/tensorboard-pytorch`

## API
http://tensorboard-pytorch.readthedocs.io/en/latest/tensorboard.html

## Usage
```python
import torch
import torchvision.utils as vutils
import numpy as np
import torchvision.models as models
from torchvision import datasets
from tensorboardX import SummaryWriter

resnet18 = models.resnet18(False)
writer = SummaryWriter()
sample_rate = 44100
freqs = [262, 294, 330, 349, 392, 440, 440, 440, 440, 440, 440]

for n_iter in range(100):
    s1 = torch.rand(1) # value to keep
    s2 = torch.rand(1)
    writer.add_scalar('data/scalar1', s1[0], n_iter) #data grouping by `slash`
    writer.add_scalar('data/scalar2', s2[0], n_iter)
    writer.add_scalars('data/scalar_group', {"xsinx":n_iter*np.sin(n_iter),
                                             "xcosx":n_iter*np.cos(n_iter),
                                             "arctanx": np.arctan(n_iter)}, n_iter)
    x = torch.rand(32, 3, 64, 64) # output from network
    if n_iter%10==0:
        x = vutils.make_grid(x, normalize=True, scale_each=True)
        writer.add_image('Image', x, n_iter)
        x = torch.zeros(sample_rate*2)
        for i in range(x.size(0)):
            x[i] = np.cos(freqs[n_iter//10]*np.pi*float(i)/float(sample_rate)) # sound amplitude should in [-1, 1]
        writer.add_audio('myAudio', x, n_iter, sample_rate=sample_rate)
        writer.add_text('Text', 'text logged at step:'+str(n_iter), n_iter)
        for name, param in resnet18.named_parameters():
            writer.add_histogram(name, param.clone().cpu().data.numpy(), n_iter)

dataset = datasets.MNIST('mnist', train=False, download=True)
images = dataset.test_data[:100].float()
label = dataset.test_labels[:100]
features = images.view(100, 784)
writer.add_embedding(features, metadata=label, label_img=images.unsqueeze(1))

# export scalar data to JSON for external processing
writer.export_scalars_to_json("./all_scalars.json")

writer.close()
```

`python demo.py`

`tensorboard --logdir runs`

## Screenshots
<img src="screenshots/Demo.gif">


## Tweaks
To show more images in tensorboard's image tab, just
modify the hardcoded `event_accumulator` in
`~/anaconda3/lib/python3.6/site-packages/tensorflow/tensorboard/backend/application.py`
as you wish.

## Reference:

https://github.com/TeamHG-Memex/tensorboard_logger

https://github.com/dmlc/tensorboard
