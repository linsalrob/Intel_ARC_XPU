# Install Phold for XPU computing

Using the xpu for phold...does it help?


# install the WSL xpu dependencies

I was having a hard time with launchpad, and this made it go a lot quicker (well, work, actually):

```
sudo apt-get update -o Acquire::ForceIPv4=true
sudo apt-get install xpu-smi -o Acquire::ForceIPv4=true
sudo apt install -y libze1 libze-intel-gpu1 intel-metrics-discovery intel-gsc clinfo
```

Then check you can see the xpu:

```


# Create the phold environment

Create a new environment and install pytorch. These instructions are from the [pytorch page](https://docs.pytorch.org/docs/stable/notes/get_start_xpu.html)

```
mamba create -n pholdXPU python==3.12
mamba activate pholdXPU
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/xpu
```

Next, try a sanity check which should detect the XPU

```
python -c "import torch; print(f'Torch XPU is available: {torch.xpu.is_available()}')"
python -c "import torch; print(f'torch: {torch.__version__}'); [print(f'[{i}]: {torch.xpu.get_device_properties(i)}') for i in range(torch.xpu.device_count())];"
```

This should tell you what your XPU devices are. On my laptop, I get the following output:


```
2.5.10+xpu
[0]: _XpuDeviceProperties(name='Intel(R) Graphics [0x64a0]', platform_name='Intel(R) oneAPI Unified Runtime over Level-Zero', type='gpu', driver_version='1.14.37020+3', total_memory=16870MB, max_compute_units=64, gpu_eu_count=64, gpu_subslice_count=8, max_work_group_size=1024, max_num_sub_groups=64, sub_group_sizes=[16 32], has_fp16=1, has_fp64=1, has_atomic64=1)
```

## Install the dependencies

I don't like doing it this way, because a future `phold` version may break this. However, we can't install over the top of pytorch!

```
mamba install -c conda-forge -c bioconda foldseek=10.941cd33 biopython alive-progress datasets requests pyarrow pandas loguru pyyaml click sentencepiece transformers pyrodigal-gv numpy pycirclize h5py pip protobuf tqdm dataclasses
```

Here are the pip dependencies I needed to install:

```
pip install tiktoken click
```



## .




# Fail

The steps below didn't work, and partly because the code is deprecated and Intel have not provided a link to the appropriate replacement. And partly because I didn't RTFM. Sigh.

Below is the conda install of torch+xpu, but it failed because the latest conda version is v2.5.10, and we need at least v2.6.

Back to the drawingboard.

```
ValueError: Due to a serious vulnerability issue in `torch.load`, even with `weights_only=True`, we now require users to upgrade torch to at least v2.6 in order to use the function. This version restriction does not apply when loading files with safetensors.
See the vulnerability report here https://nvd.nist.gov/vuln/detail/CVE-2025-32434
```


# Install the XPU pytorch drivers

These instructions come from [the Intel installation](https://pytorch-extension.intel.com/installation?platform=gpu&version=v2.3.110%2Bxpu&os=linux%2Fwsl2&package=conda) and we're using Linux under WSL2.

You should use the latest version for which a conda install is available (currently the latest version is 2.7.10+xpu but the conda install is only available for version 2.5.10+xpu

```
mamba install intel-extension-for-pytorch=2.5.10 pytorch=2.5.1 -c https://software.repos.intel.com/python/conda -c conda-forge
```

<details>
<summary>If you want to used the latest version ... </summary>
You can do a `pip install` of the latest versions, but in the subsequent steps, you can't do a `mamba install` otherwise it will attempt to overwrite the pytorch installation, with undetermined results.
</details>


Next, try the sanity check suggested on that Intel site:

```
python -c "import torch; import intel_extension_for_pytorch as ipex; print(torch.__version__); print(ipex.__version__); [print(f'[{i}]: {torch.xpu.get_device_properties(i)}') for i in range(torch.xpu.device_count())];"
```

This should tell you what your XPU devices are. On my laptop, I get the following output:


```
2.5.1+cxx11.abi
2.5.10+xpu
[0]: _XpuDeviceProperties(name='Intel(R) Graphics [0x64a0]', platform_name='Intel(R) oneAPI Unified Runtime over Level-Zero', type='gpu', driver_version='1.14.37020+3', total_memory=16870MB, max_compute_units=64, gpu_eu_count=64, gpu_subslice_count=8, max_work_group_size=1024, max_num_sub_groups=64, sub_group_sizes=[16 32], has_fp16=1, has_fp64=1, has_atomic64=1)
```

<details>

<summary>If you get the dreaded `ModuleNotFoundError: No module named 'pkg_resources'` error</summary>

Then you need to install `setuptools` with a version less than 82. This worked for me:

```
python -m pip install "setuptools<82"
```

</details>


# Install Phold

```
mamba install -c https://software.repos.intel.com/python/conda -c conda-forge -c bioconda phold
```


# Now install the phold databases

```
phold install -d phold_db --foldseek_gpu
```



