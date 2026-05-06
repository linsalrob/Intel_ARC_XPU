# Install Phold for XPU computing

Using the xpu for phold...does it help?


# install the WSL xpu dependencies

I was having a hard time with launchpad, and this made it go a lot quicker (well, work, actually):

```
sudo apt-get update -o Acquire::ForceIPv4=true
sudo apt-get install xpu-smi -o Acquire::ForceIPv4=true
sudo apt install -y libze1 libze-intel-gpu1 intel-metrics-discovery intel-gsc clinfo
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




# Now install the phold databases

After this, you can continue with phold as-is

```
phold install -d phold_db 
```



