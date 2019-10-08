# tfcoreml

TensorFlow (TF) to Core ML Converter

[New] :fire: tfcoreml converter with Core ML 3
----------------------------------------------
To try out the new converter with Core ML 3, install `coremltools` 3.0 and `tfcoreml` 1.0.

```shell
pip install coremltools==3.0
pip install tfcoreml==1.0
```

There is a new flag `target_ios` which can be utilized for setting minimum targeting iOS, default to `target_ios='12'`.

In addition, node names must be passed instead of tensor names to `input_name_shape_dict` and `output_feature_names` for `target_ios` 13 or later.
For example:

```python
import tfcoreml as tf_converter

tf_converter.convert(tf_model_path='model.pb',
                     mlmodel_path='model.mlmodel',
                     output_feature_names=['softmax'],
                     input_name_shape_dict={'input': [1, 227, 227, 3]},
                     target_ios='13')
```

Dependencies
------------

- tensorflow >= 1.5.0
- coremltools >= 0.8
- numpy >= 1.6.2
- protobuf >= 3.1.0
- six >= 1.10.0

## Installation

### Install from Source

To get the latest version of the converter, clone this repository and install from source:

```shell
git clone https://github.com/tf-coreml/tf-coreml.git
cd tf-coreml
```

To install as a package with `pip`, either run (at the root directory):

```shell
pip install -e .
```

or run:

```shell
python setup.py bdist_wheel
```

This will generate a `pip` installable wheel inside the `dist/` directory.

### Install From PyPI

To install the PyPI package:

```shell
pip install -U tfcoreml
```

## Usage

See iPython notebooks in the directory `examples/` for examples of
how to use the converter.

The following arguments are required by the Core ML converter:
- path to the frozen *.pb* graph file to be converted
- path where the *.mlmodel* should be written
- a list of output tensor names present in the TF graph
- a dictionary of input names and their shapes (as list of integers).
  This is only required if input tensors' shapes are not fully defined in the frozen *.pb* file
	(e.g. they contain `None` or `?`)

Note that the frozen *.pb* file can be obtained from the checkpoint and graph def files
by using the `tensorflow.python.tools.freeze_graph` utility.

For details of freezing TensorFlow graphs, please refer to the
[TensorFlow documentation](https://www.tensorflow.org/api_docs/python/tf/graph_util/remove_training_nodes)
and the Jupyter notebooks in directory `examples/` in this repo.
There are scripts in the `utils/` directory for visualizing and writing out a text summary of a given frozen TensorFlow graph.
This could be useful in determining the input/output names and shapes.
Another useful tool for visualizing frozen TensorFlow graphs is [Netron](https://github.com/lutzroeder/Netron).

There are additional arguments that the converter can take. For details, refer to the full function definition [here](https://github.com/tf-coreml/tf-coreml/blob/4873258a145664106154922ad8ee09a0a3729ee0/tfcoreml/_tf_coreml_converter.py#L395).

**For example:**

When input shapes are fully determined in the frozen *.pb* file:

```python
import tfcoreml as tf_converter
tf_converter.convert(tf_model_path='my_model.pb',
                     mlmodel_path='my_model.mlmodel',
                     output_feature_names=['softmax:0'])
```

When input shapes are not fully specified in the frozen *.pb* file:

```python
import tfcoreml as tf_converter
tf_converter.convert(tf_model_path='my_model.pb',
                     mlmodel_path='my_model.mlmodel',
                     output_feature_names=['softmax:0'],
                     input_name_shape_dict={'input:0': [1, 227, 227, 3]})
```


Following topics are discussed in the Jupyter notebooks under the `examples/` folder:

**custom_layer_examples.ipynb**: A few examples to demonstrate the process of adding custom Core ML layers for unsupported TF ops.

**deep_speech.ipynb**: An example to freeze and convert a popular TensorFlow implementations of deep speech model.

**inception_v1_preprocessing_steps.ipynb**: How to generate a classifier model with image input types and the importance of properly setting the preprocessing parameters.

**inception_v3.ipynb**: How to strip the "DecodeJpeg" op from the TF graph to prepare it for Core ML conversion.

**linear_mnist_example.ipynb**: How to get a frozen graph from the checkpoint and graph description files generated by training in TF.

**ssd_example.ipynb**: How to extract a portion of the TF graph that can be converted, from the overall graph that may have unsupported ops.

**style_transfer_example.ipynb**: How to edit a Core ML model to get an image output type (by default the outputs are MultiArrays).

## Supported Operators

List of **[TensorFlow operators supported in Core ML via the converter](https://github.com/apple/coremltools/blob/2a08445ade3c0da81fb2b25cf6de9f88c993be0c/coremltools/converters/nnssa/coreml/ssa_converter.py#L330)**.

Note that certain parameterizations of these ops may not be fully supported. For unsupported ops or configurations, the [custom layer](https://developer.apple.com/documentation/coreml/core_ml_api/creating_a_custom_layer) feature of Core ML can be used. For details, refer to the `examples/custom_layer_examples.ipynb` notebook.

Scripts for converting several of the following pre-trained models can be found at `tests/test_pretrained_models.py`.
Other models with similar structures and supported ops can be converted.
Below is a list of publicly available TensorFlow frozen models that can be converted with this converter:

- [Inception v1 (Slim)](https://storage.googleapis.com/download.tensorflow.org/models/inception_v1_2016_08_28_frozen.pb.tar.gz)
- [Inception v2 (Slim)](https://storage.googleapis.com/download.tensorflow.org/models/inception_v2_2016_08_28_frozen.pb.tar.gz)
- [Inception v3 (Slim)](https://storage.googleapis.com/download.tensorflow.org/models/inception_v3_2016_08_28_frozen.pb.tar.gz)
- [Inception v4 (Slim)](https://storage.googleapis.com/download.tensorflow.org/models/inception_v4_2016_09_09_frozen.pb.tar.gz)
- [Inception v3 (non-Slim)*](https://storage.googleapis.com/download.tensorflow.org/models/inception_dec_2015.zip)
- [Inception/ResNet v2 (Slim)](https://storage.googleapis.com/download.tensorflow.org/models/inception_resnet_v2_2016_08_30_frozen.pb.tar.gz)
- MobileNet variations (Slim):
  - Image size: 128 ([1](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.25_128_frozen.tgz),
                      [2](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.50_128_frozen.tgz),
                      [3](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.75_128_frozen.tgz),
                      [4](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_1.0_128_frozen.tgz))
  - Image size: 160 ([1](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.25_160_frozen.tgz),
                      [2](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.50_160_frozen.tgz),
                      [3](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.75_160_frozen.tgz),
                      [4](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_1.0_160_frozen.tgz))
  - Image size: 192 ([1](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.25_192_frozen.tgz),
                      [2](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.50_192_frozen.tgz),
                      [3](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.75_192_frozen.tgz),
                      [4](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_1.0_192_frozen.tgz))
  - Image size: 224 ([1](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.25_224_frozen.tgz),
                      [2](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.50_224_frozen.tgz),
                      [3](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_0.75_224_frozen.tgz),
                      [4](
                      https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_1.0_224_frozen.tgz))
- [Image stylization network+](https://storage.googleapis.com/download.tensorflow.org/models/stylize_v1.zip)
- [Mobilenet SSD*](https://storage.googleapis.com/download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_android_export.zip)

`*` Converting these models requires extra steps to extract subgraphs from the TF frozen graphs. See `examples/` for details.

### Limitations

`tfcoreml` converter has the following constraints for `target_ios <= 12`:

- TensorFlow graph must be cycle free (cycles are generally created due to control flow ops like `if`, `while`, `map`, etc.)
- Must have `NHWC` ordering (Batch size, Height, Width, Channels) for image feature map tensors.
- Must have tensors with rank less than or equal to 4 (`len(tensor.shape) <= 4`).
- The converter produces Core ML model with float values. A quantized TF graph (such as the style transfer network linked above) gets converted to a float Core ML model

for `target_ios >= 13`, `tfcoreml` added supports for control flow and tensors from rank 1-5 (`1 <= len(tensor.shape) <= 5`).

## Running Unit Tests

In order to run unit tests, you need `pytest`.

```shell
pip install pytest
```

To add a new unit test, add it to the `tests/` folder. Make sure you
name the file with a 'test' as the prefix.
To run all unit tests, navigate to the `tests/` folder and run

```shell
pytest
```

## Directories

- `tfcoreml`: the tfcoreml package
- `examples`: examples to use this converter
- `tests`: unit tests
- `utils`: general scripts for graph inspection

## License
[Apache License 2.0](LICENSE)
