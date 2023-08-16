# llm-rk3588

This repository is intend to provide a complete guide on how to run LLMs on rk3588 SBC, specifically Orange Pi 5 Plus. But other rk3588 based board should be able to run without problem.

## Link
[BLOG](https://blog.mlc.ai/2023/08/09/GPU-Accelerated-LLM-on-Orange-Pi) | [MLC-LLM](https://github.com/mlc-ai/mlc-llm) | [Apache TVM](https://github.com/apache/tvm)

## Environment setup

* Download and install the Ubuntu 22.04 for your board from [here](https://github.com/Joshua-Riek/ubuntu-rockchip/releases/tag/v1.22)

* Download and install `libmali-g610.so`

    ```
    cd /usr/lib && sudo wget https://github.com/JeffyCN/mirrors/raw/libmali/lib/aarch64-linux-gnu/libmali-valhall-g610-g6p0-x11-wayland-gbm.so
    ```

* Check if file `mali_csffw.bin` exist under path `/lib/firmware` by running `ls /lib/firmware`

* If not, then download it with command:

    ```
    cd /lib/firmware && sudo wget https://github.com/JeffyCN/mirrors/raw/libmali/firmware/g610/mali_csffw.bin
    ```

* Download OpenCL ICD loader and manually add libmali to ICD

    ```
    sudo apt install mesa-opencl-icd
    sudo mkdir -p /etc/OpenCL/vendors
    echo "/usr/lib/libmali-valhall-g610-g6p0-x11-wayland-gbm.so" | sudo tee /etc/OpenCL/vendors/mali.icd
    ```

* Download and install `libOpenCL`

    ```
    sudo apt install ocl-icd-opencl-dev
    ```

* Download and install dependencies for Mali OpenCL

    ```
    sudo apt install libxcb-dri2-0 libxcb-dri3-0 libwayland-client0 libwayland-server0 libx11-xcb1 clinfo
    ```

* Run `clinfo` to check if OpenCL runs successfully

    ```
    $ clinfo
    arm_release_ver: g13p0-01eac0, rk_so_ver: 3
    Number of platforms                               2
        Platform Name                                   ARM Platform
        Platform Vendor                                 ARM
        Platform Version                                OpenCL 2.1 v1.g6p0-01eac0.2819f9d4dbe0b5a2f89c835d8484f9cd
        Platform Profile                                FULL_PROFILE
        ...
    ```

Feel free to check [this](https://github.com/mlc-ai/mlc-llm/blob/main/docs/install/gpu.rst) article for other platform

## MLC-LLM setup

### Use prebuilt (recommended)
* Clone `mlc-llm` repository

    ```
    sudo apt install git git-lfs
    git clone --recursive https://github.com/mlc-ai/mlc-llm.git && cd mlc-llm
    mkdir -p dist/prebuilt && cd dist/prebuilt
    git clone https://github.com/mlc-ai/binary-mlc-llm-libs.git lib
    git clone https://huggingface.co/mlc-ai/mlc-chat-RedPajama-INCITE-Chat-3B-v1-q4f16_1
    cd ../../..
    ```

* Build `mlc-chat-cli` from source

    ```
    cd mlc-llm/
    mkdir -p build && cd build
    python3 ../cmake/gen_cmake_config.py
    cmake .. && cmake --build . --parallel $(nproc) && cd ..
    ```

* Verify installation

    ```
    # expect to see `mlc_chat_cli`, `libmlc_llm.so` and `libtvm_runtime.so`
    ls -l ./build/
    ```
    
    ```
    # expect to see help message
    ./build/mlc_chat_cli --help
    ```

* Run LLMs

    ```
    ./build/mlc_chat_cli --local-id RedPajama-INCITE-Chat-3B-v1-q4f16_1 –device mali
    ```

### Compile your own LLMs

* Install `mlc-llm` package

    ```
    git clone --recursive https://github.com/mlc-ai/mlc-llm.git && cd mlc-llm
    pip install .
    ```

* Verify installation

    ```
    # expect to see help info
    python3 -m mlc_llm.build --help
    ```

* Compile models

    Make sure the model you are using is Huggingface format (read model description before you download)

    ```
    python3 -m mlc_llm.build --hf-path togethercomputer/RedPajama-INCITE-Chat-3B-v1 --target opencl --quantization q4f16_1
    ```

    OR, you can use models you downloaded in local computer

    ```
    python3 -m mlc_llm.build --model /path/to/model --target opencl --quantization q4f16_1
    ```

    Available quantization codes are: `q3f16_0`, `q4f16_1`, `q4f16_2`, `q4f32_0`, `q0f32`, and `q0f16`