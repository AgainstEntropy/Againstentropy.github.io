---
layout: post
title:  How to solve "RuntimeError cuDNN version incompatibility"
date: 2023-05-09 00:00:00
# description: this is a start-up tutorial for scripting in Blender
tags: PyTorch cuDNN Debug
categories: problem-solving
related_posts: false
toc:
  sidebar: left
---

## Problem setting

When you just install a new version of PyTorch, you'd like to verify if you have installed it correctly. Typically, the following codes will be enough:

```python
import torch
print(f"torch version: {torch.__version__}")
use_cuda = torch.cuda.is_available()
if use_cuda:
    GPU_nums = torch.cuda.device_count()
    GPU = torch.cuda.get_device_properties(0)
    print(f"There are {GPU_nums} GPUs in total.\nThe first GPU is: {GPU}")
    print(f"CUDA version: {torch.version.cuda}")
    print(f'cudnn version: {torch.backends.cudnn.version()}')
device = torch.device(f"cuda:0" if use_cuda else "cpu")
print(f"Using {device} now!")
```

Sometimes although CUDA is available, there is still something wrong with cuDNN. This maybe induce error like below:

```plain
RuntimeError: cuDNN version incompatibility: PyTorch was compiled  against (8, 7, 0) but found runtime version (8, 1, 1). PyTorch already comes bundled with cuDNN. One option to resolving this error is to ensure PyTorch can find the bundled cuDNN. Looks like your LD_LIBRARY_PATH contains incompatible version of cudnn. Please either remove it from the path or install cudnn (8, 7, 0)
```

## Problem locating

According to the error message, let's print the `LD_LIBRARY_PATH` and the return may appears like below:

```shell
$ $LD_LIBRARY_PATH
bash: /mnt/lustre/share/cuda-11.8/lib64:/usr/local/cuda/lib:/usr/local/cuda/lib64/:...
```

Let's check each of them. First is the CUDA path:

```shell
$ ls /mnt/lustre/share/cuda-11.8/lib64
cmake                         libcufftw.so             libcurand.so.10.3.0.86       libnppial_static.a       libnppist.so.11.8.0.86   libnvperf_target.so
libaccinj64.so                libcufftw.so.10          libcurand_static.a           libnppicc.so             libnppist_static.a       libnvptxcompiler_static.a
libaccinj64.so.11.8           libcufftw.so.10.9.0.58   libcusolver_lapack_static.a  libnppicc.so.11          libnppisu.so             libnvrtc-builtins.so
libaccinj64.so.11.8.87        libcufftw_static.a       libcusolverMg.so             libnppicc.so.11.8.0.86   libnppisu.so.11          libnvrtc-builtins.so.11.8
libcheckpoint.so              libcufile_rdma.so        libcusolverMg.so.11          libnppicc_static.a       libnppisu.so.11.8.0.86   libnvrtc-builtins.so.11.8.89
libcublasLt.so                libcufile_rdma.so.1      libcusolverMg.so.11.4.1.48   libnppidei.so            libnppisu_static.a       libnvrtc-builtins_static.a
libcublasLt.so.11             libcufile_rdma.so.1.4.0  libcusolver.so               libnppidei.so.11         libnppitc.so             libnvrtc.so
libcublasLt.so.11.11.3.6      libcufile_rdma_static.a  libcusolver.so.11            libnppidei.so.11.8.0.86  libnppitc.so.11          libnvrtc.so.11.2
libcublasLt_static.a          libcufile.so             libcusolver.so.11.4.1.48     libnppidei_static.a      libnppitc.so.11.8.0.86   libnvrtc.so.11.8.89
libcublas.so                  libcufile.so.0           libcusolver_static.a         libnppif.so              libnppitc_static.a       libnvrtc_static.a
libcublas.so.11               libcufile.so.1.4.0       libcusparse.so               libnppif.so.11           libnpps.so               libnvToolsExt.so
libcublas.so.11.11.3.6        libcufile_static.a       libcusparse.so.11            libnppif.so.11.8.0.86    libnpps.so.11            libnvToolsExt.so.1
libcublas_static.a            libcufilt.a              libcusparse.so.11.7.5.86     libnppif_static.a        libnpps.so.11.8.0.86     libnvToolsExt.so.1.0.0
libcudadevrt.a                libcuinj64.so            libcusparse_static.a         libnppig.so              libnpps_static.a         libOpenCL.so
libcudart.so                  libcuinj64.so.11.8       liblapack_static.a           libnppig.so.11           libnvblas.so             libOpenCL.so.1
libcudart.so.11.0             libcuinj64.so.11.8.87    libmetis_static.a            libnppig.so.11.8.0.86    libnvblas.so.11          libOpenCL.so.1.0
libcudart.so.11.8.89          libculibos.a             libnppc.so                   libnppig_static.a        libnvblas.so.11.11.3.6   libOpenCL.so.1.0.0
libcudart_static.a            libcupti.so              libnppc.so.11                libnppim.so              libnvjpeg.so             libpcsamplingutil.so
libcufft.so                   libcupti.so.11.8         libnppc.so.11.8.0.86         libnppim.so.11           libnvjpeg.so.11          stubs
libcufft.so.10                libcupti.so.2022.3.0     libnppc_static.a             libnppim.so.11.8.0.86    libnvjpeg.so.11.9.0.86
libcufft.so.10.9.0.58         libcupti_static.a        libnppial.so                 libnppim_static.a        libnvjpeg_static.a
libcufft_static.a             libcurand.so             libnppial.so.11              libnppist.so             libnvperf_host.so
libcufft_static_nocallback.a  libcurand.so.10          libnppial.so.11.8.0.86       libnppist.so.11          libnvperf_host_static.a
```

and find there is no cuDNN at all. Look into the remain paths and finally we locate the imcompatible version of cuDNN:

```shell
$ ls /usr/local/cuda/lib64
libaccinj64.so               libcudnn_cnn_infer.so.8       libcufftw.so.10.4.1.152      libnppc.so                libnppig_static.a        libnvblas.so.11.4.1.1043
libaccinj64.so.11.2          libcudnn_cnn_infer.so.8.1.1   libcufftw_static.a           libnppc.so.11             libnppim.so              libnvjpeg.so
libaccinj64.so.11.2.152      libcudnn_cnn_train.so         libcuinj64.so                libnppc.so.11.3.2.152     libnppim.so.11           libnvjpeg.so.11
libcublasLt.so               libcudnn_cnn_train.so.8       libcuinj64.so.11.2           libnppc_static.a          libnppim.so.11.3.2.152   libnvjpeg.so.11.4.0.152
libcublasLt.so.11            libcudnn_cnn_train.so.8.1.1   libcuinj64.so.11.2.152       libnppial.so              libnppim_static.a        libnvjpeg_static.a
libcublasLt.so.11.4.1.1043   libcudnn_ops_infer.so         libculibos.a                 libnppial.so.11           libnppist.so             libnvptxcompiler_static.a
libcublasLt_static.a         libcudnn_ops_infer.so.8       libcurand.so                 libnppial.so.11.3.2.152   libnppist.so.11          libnvrtc-builtins.so
libcublas.so                 libcudnn_ops_infer.so.8.1.1   libcurand.so.10              libnppial_static.a        libnppist.so.11.3.2.152  libnvrtc-builtins.so.11.2
libcublas.so.11              libcudnn_ops_train.so         libcurand.so.10.2.3.152      libnppicc.so              libnppist_static.a       libnvrtc-builtins.so.11.2.152
libcublas.so.11.4.1.1043     libcudnn_ops_train.so.8       libcurand_static.a           libnppicc.so.11           libnppisu.so             libnvrtc.so
libcublas_static.a           libcudnn_ops_train.so.8.1.1   libcusolverMg.so             libnppicc.so.11.3.2.152   libnppisu.so.11          libnvrtc.so.11.2
libcudadevrt.a               libcudnn.so                   libcusolverMg.so.11          libnppicc_static.a        libnppisu.so.11.3.2.152  libnvrtc.so.11.2.152
libcudart.so                 libcudnn.so.8                 libcusolverMg.so.11.1.0.152  libnppidei.so             libnppisu_static.a       libnvToolsExt.so
libcudart.so.11.0            libcudnn.so.8.1.1             libcusolver.so               libnppidei.so.11          libnppitc.so             libnvToolsExt.so.1
libcudart.so.11.2.152        libcudnn_static.a             libcusolver.so.11            libnppidei.so.11.3.2.152  libnppitc.so.11          libnvToolsExt.so.1.0.0
libcudart_static.a           libcudnn_static_v8.a          libcusolver.so.11.1.0.152    libnppidei_static.a       libnppitc.so.11.3.2.152  libOpenCL.so
libcudnn_adv_infer.so        libcufft.so                   libcusolver_static.a         libnppif.so               libnppitc_static.a       libOpenCL.so.1
libcudnn_adv_infer.so.8      libcufft.so.10                libcusparse.so               libnppif.so.11            libnpps.so               libOpenCL.so.1.0
libcudnn_adv_infer.so.8.1.1  libcufft.so.10.4.1.152        libcusparse.so.11            libnppif.so.11.3.2.152    libnpps.so.11            libOpenCL.so.1.0.0
libcudnn_adv_train.so        libcufft_static.a             libcusparse.so.11.4.1.1152   libnppif_static.a         libnpps.so.11.3.2.152    nvrtc-prev
libcudnn_adv_train.so.8      libcufft_static_nocallback.a  libcusparse_static.a         libnppig.so               libnpps_static.a         stubs
libcudnn_adv_train.so.8.1.1  libcufftw.so                  liblapack_static.a           libnppig.so.11            libnvblas.so
libcudnn_cnn_infer.so        libcufftw.so.10               libmetis_static.a            libnppig.so.11.3.2.152    libnvblas.so.11
```

## Problem solving

Again, according to the error message, PyTorch is installed with the correct version of cuDNN bundled. To "ensure PyTorch can find the bundled cuDNN", we need to find the bundled lib and add it to `LD_LIBRARY_PATH`.

If you are using conda, the bundled lib can be found at `site-packages/torch/lib` under your environment directory:

```shell
$ ls [/path/to/your/conda]/envs/[env-name]/lib/python3.9/site-packages/torch/lib
libc10_cuda.so      libcublas.so.11             libcudnn_cnn_infer.so.8  libcudnn.so.8              libnvrtc-builtins.so.11.8    libtorch_cuda_linalg.so  libtorch.so
libc10.so           libcudart-d0da41ae.so.11.0  libcudnn_cnn_train.so.8  libgomp-a34b3233.so.1      libnvToolsExt-847d78f2.so.1  libtorch_cuda.so
libcaffe2_nvrtc.so  libcudnn_adv_infer.so.8     libcudnn_ops_infer.so.8  libnvfuser_codegen.so      libshm.so                    libtorch_global_deps.so
libcublasLt.so.11   libcudnn_adv_train.so.8     libcudnn_ops_train.so.8  libnvrtc-672ee683.so.11.2  libtorch_cpu.so              libtorch_python.so
```

The last thing is to modify the environment variables, i.e., add the following lines to your `~/.bashrc`:

```bash
# cuDNN
export MY_TORCH_LIB="[/path/to/your/conda]/envs/[env-name]/lib/python3.9/site-packages/torch/lib"
export LD_LIBRARY_PATH=$MY_TORCH_LIB:$LD_LIBRARY_PATH
```

Don't use `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MY_TORCH_LIB` becasue we want `$MY_TORCH_LIB` to have a higher priority.

Finally, don't forget to call `source ~/.bashrc`, which adds the new `LD_LIBRARY_PATH` to the environment variables.