# Triton介绍

**简介**

Triton Inference Server（以下简称Triton）是由NVIDIA推出的开源推理框架，旨在为AI算法模型提供高效的部署和推理能力。Triton采用C/S架构，Client端发送推理请求，Server端推理。Triton具有以下优势：

*   支持多种深度学习框架：包括PyTorch，Tensorflow，TensorRT，ONNX，OpenVINO等产出的模型文件。
    
*   支持多种通讯协议：支持HTTP，GRPC推理协议。
    
*   服务端支持模型前后处理：提供后端API，支持将数据的前处理和模型推理的后处理在服务端实现。
    
*   支持模型并发推理：支持多个模型或者同一模型的多个实例在同一系统上并行执行。
    
*   支持动态批处理（Dynamic batching）：支持将一个或多个推理请求合并成一个批次，以最大化吞吐量。
    
*   支持多模型的集成流水线：支持将多个模型进行连接组合，将其视作一个整体进行调度管理。
    
*   支持自定义Backend，包括Python Backend、C++ Backend。
    
*   C端提供Python API、C API、Java API，方便集成。
    

**注意**：Triton具有二义性，一是指本文所讲到的Triton Inference Server，二是指Triton语言，是[OpenAI](https://www.baidu.com/s?wd=OpenAI&rsv_idx=2&tn=68018901_16_pg&usm=1&ie=utf-8&rsv_pq=aa91cf0f041f372a&oq=triton%E8%AF%AD%E8%A8%80&rsv_t=3c72yijFp6fbQM%2BdJt8R5%2BA6Ke7OH8eLvfS3nmqCGOYC0QcbWUAXoVocUMUKvw6wlN%2BmY%2Bw&rsv_dl=re_dqa_generate&sa=re_dqa_generate)开发的一种用于编写高效自定义深度学习原语的语言和编译器，它的主要目标是提供一个开源环境，使开发者能够以比[CUDA](https://www.baidu.com/s?wd=CUDA&rsv_idx=2&tn=68018901_16_pg&usm=1&ie=utf-8&rsv_pq=aa91cf0f041f372a&oq=triton%E8%AF%AD%E8%A8%80&rsv_t=deedLKpAwcsg5PINZaz%2Fx8Qnt%2FDusvaYIn%2FxCXhR3V5rSHoAJaZZT0y0Y9LzMcZg8E2DkAk&rsv_dl=re_dqa_generate&sa=re_dqa_generate)更高的效率编写代码，同时比其他现有的领域特定语言（DSL）具有更高的灵活性。本文提到的Triton都是指Triton Inference Server。

Triton架构如下图：

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/J9LnWNMeRR2AlvDe/img/711f0bc8-2fa0-4c73-a523-d1b4824d8758.jpg)

下面简要介绍部署模型，包括如何编写Backend代码、client代码以及如何测试和调试，详细的开发和使用说明请参考：

*   Triton源码仓库：[GitHub - triton-inference-server/server: The Triton Inference Server provides an optimized cloud and edge inferencing solution.](https://github.com/triton-inference-server/server)
    
*   NVIDIA官方文档：[Quickstart — NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/getting_started/quickstart.html)
    
*   Python Backend源码仓库：[GitHub - triton-inference-server/python\_backend: Triton backend that enables pre-process, post-processing and other logic to be implemented in Python.](https://github.com/triton-inference-server/python_backend)
    

其中Triton源码仓库与NVIDIA官方文档内容基本一致，参考其中一个即可。

**快速上手**

Triton支持多种深度学习框架，包括PyTorch，Tensorflow，TensorRT，ONNX，OpenVINO，因此有多种Backend，此外还支持用户自定义Backend，自定义Backend支持采用C++或者python语言编写，本文主要讲述采用Python语言编写用户自定义Backend，称为Python Backend，其他类型的Backend可参考官网。Python Backend详细介绍可参考：[GitHub - triton-inference-server/python\_backend: Triton backend that enables pre-process, post-processing and other logic to be implemented in Python.](https://github.com/triton-inference-server/python_backend?tab=readme-ov-file#error-handling) 其中包含example代码。

## 2.1 搭建Triton运行环境

有两种搭建方式：

*   源码编译：[Building Triton — NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/customization_guide/build.html)
    
*   拉取Docker镜像：[Triton Inference Server | NVIDIA NGC](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver/tags)
    

拉取命令：

```plaintext
sudo docker pull nvcr.io/nvidia/tritonserver:tag
```

镜像Tag含义：

| Tag                     | Scope                                                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------ |
| xx.yy-py3               | Support for Tensorflow, PyTorch, TensorRT, ONNX and OpenVINO models                                                |
| xx.yy-py3-sdk           | Contains Python and C++ client libraries, client examples, GenAI-Perf, Performance Analyzer and the Model Analyzer |
| xx.yy-py3-min           | Base for creating custom Triton server                                                                             |
| xx.yy-pyt-python-py3    | Contains the Triton Inference Server with support for PyTorch and Python backends only                             |
| xx.yy-tf2-python-py3    | Contains the Triton Inference Server with support for TensorFlow 2.x and Python backends only                      |
| xx.yy-py3-igpu          | Support for Jetson Orin devices                                                                                    |
| xx.yy-py3-igpu-sdk      | Contains Python and C++ client libraries, client examples, and the Perf Analyzer                                   |
| xx.yy-py3-igpu-min      | Base for creating custom iGPU Triton server containers                                                             |
| xx.yy-vllm-python-py3   | Support for vLLM and Python backends only                                                                          |
| xx.yy-trtllm-python-py3 | Support for TensorRT-LLM and Python backends only                                                                  |

## 2.2 部署模型

首先需要训练模型或者转换模型，这里省略，重点讲述如何构建模型仓库。模型统一放置在一个目录下进行管理，该目录下每个记录了模型的名称，配置，后端代码，模型文件等。

```plaintext
models-repository/
├── vastpipe_passthrough
│   ├── 1
│   │   ├── modelfiles
│   │   │   ├── model_info.json
│   │   │   └── passthrough.pbtxt
│   │   └── model.py
│   └── config.pbtxt
└── vastpipe_sr4k
    ├── 1
    │   ├── modelfiles
    │   │   ├── model_info.json
    │   │   └── passthrough.pbtxt
    │   └── model.py
    └── config.pbtxt
```

| 文件夹或文件                          | 含义                                                                                            |
| ------------------------------------- | ----------------------------------------------------------------------------------------------- |
| models-repository                     | 模型仓库，自定义文件夹名字                                                                      |
| vastpipe\_passthrough、vastpipe\_sr4k | 模型名字，自定义文件夹名字                                                                      |
| 1                                     | 模型版本，字符串类型，自定义                                                                    |
| modelfiles                            | 模型相关文件夹，自定义文件夹名字，可存放任意文件                                                |
| model.py                              | 模型脚本，由Triton框架调用，可在脚本内部实现initialize、execute、finalize，默认文件名为model.py |
| config.pbtxt                          | 模型配置文件                                                                                    |

### 2.2.1 model.py

如果使用Python作为后端，Triton以Python脚本的形式来允许模型，这个Python脚本默认命名为model.py，Triton Inference Server提供了服务端代码模板TritonPythonModel，只需要略微修改即可，本例的服务端代码如下：

```python
import triton_python_backend_utils as pb_utils
import numpy as np
import os,json,sys      
current = os.path.dirname(os.path.realpath(__file__))
# from register_module import register_module
# import vastpipe 


class TritonPythonModel:
    """Your Python model must use the same class name. Every Python model
    that is created must have "TritonPythonModel" as the class name.
    """

    def initialize(self, args):
        """`initialize` is called only once when the model is being loaded.
        Implementing `initialize` function is optional. This function allows
        the model to initialize any state associated with this model.

        Parameters
        ----------
        args : dict
          Both keys and values are strings. The dictionary keys and values are:
          * pbtxt_file : graph file
          * module_name: module_name
        """
        print('Initialized...')
        self.module = None



    def execute(self, requests):
        """`execute` must be implemented in every Python model. `execute`
        function receives a list of pb_utils.InferenceRequest as the only
        argument. This function is called when an inference is requested
        for this model.

        Parameters
        ----------
        requests : list
          A list of pb_utils.InferenceRequest

        Returns
        -------
        list
          A list of pb_utils.InferenceResponse. The length of this list must
          be the same as `requests`
        """

        responses = []
        for request in requests:
             # 获取参数
            params = json.loads(request.parameters())
            print(params)
            # Get the input tensor
            input_tensor = pb_utils.get_input_tensor_by_name(request, 'Input')
            input0 = input_tensor.as_numpy()
            output0 = input0

            # convet to triton output
            out_tensor = pb_utils.Tensor("Output", output0 )
            inference_response = pb_utils.InferenceResponse(output_tensors=[out_tensor])
            responses.append(inference_response)

        # You must return a list of pb_utils.InferenceResponse. Length
        # of this list must match the length of `requests` list.
        return responses


    def finalize(self):
        """`finalize` is called only once when the model is being unloaded.
        Implementing `finalize` function is optional. This function allows
        the model to perform any necessary clean ups before exit.
        """
        print('Cleaning up...')

```

其中initialize方法实现了模型的初始化，execute方法实现了模型的推理逻辑，并且将推理请求passthrough给client，实际部署模型的时候，可以在其中调用vastpipe、vaststreamx、SDK来实现推理，finalize方法执行清理动作。

###  2.2.2 config.pbtxt

```plaintext
backend: "python"
max_batch_size: 32
input [
  {
    name: "Input"
    data_type: TYPE_UINT8
    dims: [ 3, -1 ,-1 ]
  }
]

output [
  {
    name: "Output"
    data_type: TYPE_UINT8
    dims: [ 3, -1 ,-1 ]
  }
]

instance_group [
  { 
    count: 3
    kind: KIND_CPU 
  }
]

```

该配置定义了模型的输入和输出，在此做简要的说明

*   **max\_batch\_size**：一个批次下的最大大小，4代表一次请求最大推理4条样本
    
*   **input**：模型的输入信息，array格式
    
*   **input-name**：一个输入的名称，该名称自定义，但是在服务端代码必须和其保持一致
    
*   **input-data\_type**：一个输入的数据类型
    
*   **input-data\_dims**：一个输入的维度，代表一条样本的维度，若max\_batch\_size不为0，它和max\_batch\_size一起构成了最终的输入大小，若max\_batch\_size为0，则dims就是最终的维度。本例中\[ 3, -1 ,-1 \]，其中-1表示任意维度。
    
*   **output**：模型的输出信息，array格式
    
*   **output-name**：一个输出的名称，该名称自定义，但是在服务端代码必须和其保持一致
    
*   **output-data\_type**：一个输出的数据类型
    
*   **output-data\_dims**：一个输出的维度，本例中\[ 3, -1 ,-1 \]，其中-1表示任意维度。
    
*   **instance\_group**：模型Instance配置，针对Python Backend，Triton将根据此配置创建模型推理进程
    
*   **count**：Instance数目，3表示创建3个Instance
    
*   **kind**：推理使用的硬件，可指定为KIND\_AUTO、KIND\_GPU、KIND\_CPU、KIND\_MODEL，其中KIND\_GPU特指NVIDIA的加速卡，针对其他厂家的加速卡需要指定为KIND\_CPU
    

有关config.pbtxt的详细解释可参考：

[Model Configuration — NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/model_configuration.html)

[common/protobuf/model\_config.proto at main · triton-inference-server/common · GitHub](https://github.com/triton-inference-server/common/blob/main/protobuf/model_config.proto)

## 2.3 运行Server

启动Triton Inference Server：

```plaintext
sudo docker run --shm-size=1g --ulimit memlock=-1 -p 8000:8000 -p 8001:8001 -p 8002:8002 --ulimit stack=67108864 -ti --device /dev/vacc0 -v `pwd`:/root nvcr.io/nvidia/tritonserver:23.12-py3 tritonserver --model-repository /root/python_backend/models  --model-control-mode explicit --load-model "vastpipe_passthrough"
```

其中：

*   \-p用于指定端口，8000为HTTP端口，用于推理请求，8001为grpc端口，用于推理请求，8002为HTTP端口，用于获取相关统计信息（Metrics），如模型执行次数，执行时间等信息，下文会讲述如何使用。
    
*   \--model-repository用于指定model仓库路径，此路径是docker内部的路径
    
*   \--model-control-mode用于控制模型加载模式，有三种模式：
    
    *   **none**：默认设置，该模式下Triton将会将所有在model\_repository下的模型在启动的时候全部加载，并且在启动之后也不会感知到模型文件的改动
        
    *   **poll**：poll模式，Triton将会轮询探查模型文件是否有变动，如果有变动Triton将会自动对模型进行重新加载，探查的频率将有参数--repository-poll-secs进行控制，该参数代表两次检查模型文件间的轮询间隔时间秒
        
    *   **explicit**：explicit模式，Triton在启动的时候将不会自动加载模型，只能手动指定--load-model来加载指定的模型，或者使用API的方式逐个加载模型
        

需要注意的是官方文档指出poll模式存在同步的问题，某些时候poll可能只能观察到部分不完成的变动，因此不建议在生产环境使用poll模式。

*   \--load-model用于指定加载的模型，可重复多次，加载多个模型
    

启动成功后会显示如下信息：

```plaintext
=============================
== Triton Inference Server ==
=============================

NVIDIA Release 23.12 (build 77457706)
Triton Server Version 2.41.0

Copyright (c) 2018-2023, NVIDIA CORPORATION & AFFILIATES.  All rights reserved.

Various files include modifications (c) NVIDIA CORPORATION & AFFILIATES.  All rights reserved.

This container image and its contents are governed by the NVIDIA Deep Learning Container License.
By pulling and using the container, you accept the terms and conditions of this license:
https://developer.nvidia.com/ngc/nvidia-deep-learning-container-license

WARNING: The NVIDIA Driver was not detected.  GPU functionality will not be available.
   Use the NVIDIA Container Toolkit to start this container with GPU support; see
   https://docs.nvidia.com/datacenter/cloud-native/ .

W1120 06:52:12.808096 1 pinned_memory_manager.cc:237] Unable to allocate pinned system memory, pinned memory pool will not be available: CUDA driver version is insufficient for CUDA runtime version
I1120 06:52:12.808154 1 cuda_memory_manager.cc:117] CUDA memory pool disabled
E1120 06:52:12.808219 1 server.cc:243] CudaDriverHelper has not been initialized.
I1120 06:52:12.809969 1 model_lifecycle.cc:461] loading: vastpipe_passthrough:1
I1120 06:52:14.044353 1 python_be.cc:2363] TRITONBACKEND_ModelInstanceInitialize: vastpipe_passthrough_0_1 (CPU device 0)
I1120 06:52:14.044360 1 python_be.cc:2363] TRITONBACKEND_ModelInstanceInitialize: vastpipe_passthrough_0_0 (CPU device 0)
I1120 06:52:14.044480 1 python_be.cc:2363] TRITONBACKEND_ModelInstanceInitialize: vastpipe_passthrough_0_2 (CPU device 0)
Initialized...
Initialized...
Initialized...
I1120 06:52:14.152459 1 model_lifecycle.cc:818] successfully loaded 'vastpipe_passthrough'
I1120 06:52:14.152545 1 server.cc:606] 
+------------------+------+
| Repository Agent | Path |
+------------------+------+
+------------------+------+

I1120 06:52:14.152680 1 server.cc:633] 
+---------+-------------------------------------------------------+--------------------------------------------------------------------------------+
| Backend | Path                                                  | Config                                                                         |
+---------+-------------------------------------------------------+--------------------------------------------------------------------------------+
| python  | /opt/tritonserver/backends/python/libtriton_python.so | {"cmdline":{"auto-complete-config":"true","backend-directory":"/opt/tritonserv |
|         |                                                       | er/backends","min-compute-capability":"6.000000","default-max-batch-size":"4"} |
|         |                                                       | }                                                                              |
+---------+-------------------------------------------------------+--------------------------------------------------------------------------------+

I1120 06:52:14.152761 1 server.cc:676] 
+----------------------+---------+--------+
| Model                | Version | Status |
+----------------------+---------+--------+
| vastpipe_passthrough | 1       | READY  |
+----------------------+---------+--------+

I1120 06:52:14.153301 1 metrics.cc:710] Collecting CPU metrics
I1120 06:52:14.153403 1 tritonserver.cc:2483] 
+----------------------------------+----------------------------------------------------------------------------------------------------------------+
| Option                           | Value                                                                                                          |
+----------------------------------+----------------------------------------------------------------------------------------------------------------+
| server_id                        | triton                                                                                                         |
| server_version                   | 2.41.0                                                                                                         |
| server_extensions                | classification sequence model_repository model_repository(unload_dependents) schedule_policy model_configurati |
|                                  | on system_shared_memory cuda_shared_memory binary_tensor_data parameters statistics trace logging              |
| model_repository_path[0]         | /root/python_backend/models                                                                                    |
| model_control_mode               | MODE_EXPLICIT                                                                                                  |
| startup_models_0                 | vastpipe_passthrough                                                                                           |
| strict_model_config              | 0                                                                                                              |
| rate_limit                       | OFF                                                                                                            |
| pinned_memory_pool_byte_size     | 268435456                                                                                                      |
| min_supported_compute_capability | 6.0                                                                                                            |
| strict_readiness                 | 1                                                                                                              |
| exit_timeout                     | 30                                                                                                             |
| cache_enabled                    | 0                                                                                                              |
+----------------------------------+----------------------------------------------------------------------------------------------------------------+

I1120 06:52:14.191630 1 grpc_server.cc:2495] Started GRPCInferenceService at 0.0.0.0:8001
I1120 06:52:14.191815 1 http_server.cc:4619] Started HTTPService at 0.0.0.0:8000
I1120 06:52:14.233785 1 http_server.cc:282] Started Metrics Service at 0.0.0.0:8002
```

从启动信息可以看到加载的模型信息、端口信息。

## 2.4 运行Client

可使用HTTP或者grpc协议与Server通讯，使用HTTP协议和grpc协议的代码写法基本一致，有细微的差别。client使用何种通讯协议，Backend侧不感知，由Triton框架处理。Client代码如下：

*   使用HTTP协议：
    

```python
from tritonclient.utils import *
import tritonclient.http as httpclient
import numpy as np


def request_resnet(input_data, model_name):
    with httpclient.InferenceServerClient("localhost:8000") as client:
        inputs = [
                httpclient.InferInput("Input", input_data.shape,
                                    np_to_triton_dtype(input_data.dtype)),
        ]
        inputs[0].set_data_from_numpy(input_data)
        outputs = [
            httpclient.InferRequestedOutput("Output")
        ]

        # 定义参数
        parameters = {
            "my_custom_parameter": "vastai"
        }

        response = client.infer(model_name,
                                inputs,
                                parameters=parameters , # 传递参数
                                request_id=str(1),
                                outputs=outputs)
        result = response.get_response()
        output_data = response.as_numpy("Output")
        print(output_data.shape)
        return output_data

if __name__ == "__main__":
    input = np.random.randint(0, 256, size=(1,3, 224,224), dtype=np.uint8)
    output = request_resnet(input, "vastpipe_passthrough")
    assert np.equal(input, output).all()
    input = np.random.randint(0, 256, size=(1,3, 200,224), dtype=np.uint8)
    output = request_resnet(input, "vastpipe_passthrough")
    assert np.equal(input, output).all()

```

*   使用grpc协议：
    

```plaintext
from tritonclient.utils import *
import tritonclient.grpc as grpcclient
import numpy as np


def request_resnet(input_data, model_name):
    with grpcclient.InferenceServerClient("localhost:8001") as client:
        inputs = [
                grpcclient.InferInput("Input", input_data.shape,
                                    np_to_triton_dtype(input_data.dtype)),
        ]
        inputs[0].set_data_from_numpy(input_data)
        # 定义参数
        parameters = {
            "my_custom_parameter": "vastai"
        }

        response = client.infer(model_name,
                                inputs,
                                parameters=parameters)
        output_data = response.as_numpy("Output")
        print(output_data.shape)
        return output_data

if __name__ == "__main__":
    input = np.random.randint(0, 256, size=(1,3, 224,224), dtype=np.uint8)
    output = request_resnet(input, "vastpipe_passthrough")
    assert np.equal(input, output).all()
    input = np.random.randint(0, 256, size=(1,3, 200,224), dtype=np.uint8)
    output = request_resnet(input, "vastpipe_passthrough")
    assert np.equal(input, output).all()

```

运行client代码：

```plaintext
sudo docker run -ti --net host -v `pwd`:/root nvcr.io/nvidia/tritonserver:23.12-py3-sdk python /root/clients/grpc_client.py
```

**Metrics**

## 3.1 获取模型统计信息

```plaintext
curl localhost:8002/metrics
```

返回内容如下：

```plaintext
# HELP nv_inference_request_success Number of successful inference requests, all batch sizes
# TYPE nv_inference_request_success counter
nv_inference_request_success{model="vastpipe_passthrough",version="1"} 2
# HELP nv_inference_request_failure Number of failed inference requests, all batch sizes
# TYPE nv_inference_request_failure counter
nv_inference_request_failure{model="vastpipe_passthrough",version="1"} 0
# HELP nv_inference_count Number of inferences performed (does not include cached requests)
# TYPE nv_inference_count counter
nv_inference_count{model="vastpipe_passthrough",version="1"} 2
# HELP nv_inference_exec_count Number of model executions performed (does not include cached requests)
# TYPE nv_inference_exec_count counter
nv_inference_exec_count{model="vastpipe_passthrough",version="1"} 2
# HELP nv_inference_request_duration_us Cumulative inference request duration in microseconds (includes cached requests)
# TYPE nv_inference_request_duration_us counter
nv_inference_request_duration_us{model="vastpipe_passthrough",version="1"} 45735
# HELP nv_inference_queue_duration_us Cumulative inference queuing duration in microseconds (includes cached requests)
# TYPE nv_inference_queue_duration_us counter
nv_inference_queue_duration_us{model="vastpipe_passthrough",version="1"} 113
# HELP nv_inference_compute_input_duration_us Cumulative compute input duration in microseconds (does not include cached requests)
# TYPE nv_inference_compute_input_duration_us counter
nv_inference_compute_input_duration_us{model="vastpipe_passthrough",version="1"} 254
# HELP nv_inference_compute_infer_duration_us Cumulative compute inference duration in microseconds (does not include cached requests)
# TYPE nv_inference_compute_infer_duration_us counter
nv_inference_compute_infer_duration_us{model="vastpipe_passthrough",version="1"} 44730
# HELP nv_inference_compute_output_duration_us Cumulative inference compute output duration in microseconds (does not include cached requests)
# TYPE nv_inference_compute_output_duration_us counter
nv_inference_compute_output_duration_us{model="vastpipe_passthrough",version="1"} 625
# HELP requests_process_latency_ns Cumulative time spent processing requests
# TYPE requests_process_latency_ns counter
requests_process_latency_ns{model="custom_metrics",version="1"} 139358
# HELP nv_inference_pending_request_count Instantaneous number of pending requests awaiting execution per-model.
# TYPE nv_inference_pending_request_count gauge
nv_inference_pending_request_count{model="vastpipe_passthrough",version="1"} 0
# HELP nv_cpu_utilization CPU utilization rate [0.0 - 1.0]
# TYPE nv_cpu_utilization gauge
nv_cpu_utilization 0.003125
# HELP nv_cpu_memory_total_bytes CPU total memory (RAM), in bytes
# TYPE nv_cpu_memory_total_bytes gauge
nv_cpu_memory_total_bytes 16502972416
# HELP nv_cpu_memory_used_bytes CPU used memory (RAM), in bytes
# TYPE nv_cpu_memory_used_bytes gauge
nv_cpu_memory_used_bytes 9946726400
```

可以看到模型推理次数、推理时间等信息，具体参数含义参考：[Metrics — NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/metrics.html)

## 3.2 获取已加载的模型列表

```python
curl -X POST http://localhost:8000/v2/repository/index
```

返回如下内容：

```python
[{"name":"common"},{"name":"vastpipe_passthrough","version":"1","state":"READY"},{"name":"vastpipe_sr4k"}]
```

其中已经加载的模型是vastpipe\_passthrough，其他模型未加载。

## 3.3 获取模型配置

```python
curl -X GET http://localhost:8000/v2/models/vastpipe_passthrough
```

其中“vastpipe\_passthrough”指模型名字，可根据具体情况替换，获取结果如下：

```python
{"name":"vastpipe_passthrough","versions":["1"],"platform":"python","inputs":[{"name":"Input","datatype":"UINT8","shape":[-1,3,-1,-1]}],"outputs":[{"name":"Output","datatype":"UINT8","shape":[-1,3,-1,-1]}]}
```

此处的模型配置即config.pbtxt文件内容 。

## 3.4 自定义Metrics

Triton支持用户自定义Metrics，自定义Metrics信息同样可以通过上述命令查看。自定义Metrics代码示例如下：

```python
import triton_python_backend_utils as pb_utils


class TritonPythonModel:
    def initialize(self, args):
      # Create a MetricFamily object to report the latency of the model
      # execution. The 'kind' parameter must be either 'COUNTER',
      # 'GAUGE' or 'HISTOGRAM'.
      self.metric_family = pb_utils.MetricFamily(
          name="preprocess_latency_ns",
          description="Cumulative time spent pre-processing requests",
          kind=pb_utils.MetricFamily.COUNTER
      )

      # Create a Metric object under the MetricFamily object. The 'labels'
      # is a dictionary of key-value pairs.
      self.metric = self.metric_family.Metric(
        labels={"model" : "model_name", "version" : "1"}
      )

    def execute(self, requests):
      responses = []

      for request in requests:
        # Pre-processing - time it to capture latency
        start_ns = time.time_ns()
        self.preprocess(request)
        end_ns = time.time_ns()

        # Update metric to track cumulative pre-processing latency
        self.metric.increment(end_ns - start_ns)

      ...

        print("Cumulative pre-processing latency:", self.metric.value())

      return responses
```

完整代码可参考：[python\_backend/examples/custom\_metrics/model.py at main · triton-inference-server/python\_backend · GitHub](https://github.com/triton-inference-server/python_backend/blob/main/examples/custom_metrics/model.py)

代码定义一个名字为preprocess\_latency\_ns的统计参数，当client发送请求之后，可以通过如下命令获取统计信息：

```python
curl localhost:8002/metrics
```

获取的内容如下：

```python
# HELP nv_inference_request_success Number of successful inference requests, all batch sizes
# TYPE nv_inference_request_success counter
nv_inference_request_success{model="vastpipe_passthrough",version="1"} 2
# HELP nv_inference_request_failure Number of failed inference requests, all batch sizes
# TYPE nv_inference_request_failure counter
nv_inference_request_failure{model="vastpipe_passthrough",version="1"} 0
# HELP nv_inference_count Number of inferences performed (does not include cached requests)
# TYPE nv_inference_count counter
nv_inference_count{model="vastpipe_passthrough",version="1"} 2
# HELP nv_inference_exec_count Number of model executions performed (does not include cached requests)
# TYPE nv_inference_exec_count counter
nv_inference_exec_count{model="vastpipe_passthrough",version="1"} 2
# HELP nv_inference_request_duration_us Cumulative inference request duration in microseconds (includes cached requests)
# TYPE nv_inference_request_duration_us counter
nv_inference_request_duration_us{model="vastpipe_passthrough",version="1"} 43717
# HELP nv_inference_queue_duration_us Cumulative inference queuing duration in microseconds (includes cached requests)
# TYPE nv_inference_queue_duration_us counter
nv_inference_queue_duration_us{model="vastpipe_passthrough",version="1"} 90
# HELP nv_inference_compute_input_duration_us Cumulative compute input duration in microseconds (does not include cached requests)
# TYPE nv_inference_compute_input_duration_us counter
nv_inference_compute_input_duration_us{model="vastpipe_passthrough",version="1"} 189
# HELP nv_inference_compute_infer_duration_us Cumulative compute inference duration in microseconds (does not include cached requests)
# TYPE nv_inference_compute_infer_duration_us counter
nv_inference_compute_infer_duration_us{model="vastpipe_passthrough",version="1"} 43222
# HELP nv_inference_compute_output_duration_us Cumulative inference compute output duration in microseconds (does not include cached requests)
# TYPE nv_inference_compute_output_duration_us counter
nv_inference_compute_output_duration_us{model="vastpipe_passthrough",version="1"} 206
# HELP requests_process_latency_ns Cumulative time spent processing requests
# TYPE requests_process_latency_ns counter
requests_process_latency_ns{model="custom_metrics",version="1"} 105559
# HELP nv_inference_pending_request_count Instantaneous number of pending requests awaiting execution per-model.
# TYPE nv_inference_pending_request_count gauge
nv_inference_pending_request_count{model="vastpipe_passthrough",version="1"} 0
# HELP nv_cpu_utilization CPU utilization rate [0.0 - 1.0]
# TYPE nv_cpu_utilization gauge
nv_cpu_utilization 0.002501563477173233
# HELP nv_cpu_memory_total_bytes CPU total memory (RAM), in bytes
# TYPE nv_cpu_memory_total_bytes gauge
nv_cpu_memory_total_bytes 16502972416
# HELP nv_cpu_memory_used_bytes CPU used memory (RAM), in bytes
# TYPE nv_cpu_memory_used_bytes gauge
nv_cpu_memory_used_bytes 10183004160
```

在其中可以看到requests\_process\_latency\_ns参数。

## 3.5 加载和卸载模型

在“2.3 运行Server”章节提到在启动tritonserver的时候可通过\--model-control-mode指定模型加载方式，在explicit模式下可以自由的使用HTTP API请求对模型进行加载和卸载，语句如下：

```python
# 加载
curl -X POST http://0.0.0.0:8000/v2/repository/models/sr4k/load
# 卸载
curl -X POST http://0.0.0.0:8000/v2/repository/models/sr4k/unload
```

其中sr4k是模型名字。

**异常处理**

## 4.1 错误处理

在Server侧，如果在处理请求的过程发生错误，可通过`TritonError` 返回错误消息，代码示例如下：

```python
import triton_python_backend_utils as pb_utils


class TritonPythonModel:
    ...

    def execute(self, requests):
        responses = []

        for request in requests:
            if an_error_occurred:
              # If there is an error, there is no need to pass the
              # "output_tensors" to the InferenceResponse. The "output_tensors"
              # that are passed in this case will be ignored.
              responses.append(pb_utils.InferenceResponse(
                error=pb_utils.TritonError("An Error Occurred")))

        return responses
```

从23.09开始，pb\_utils.TritonError支持指定错误码：

```python
pb_utils.TritonError("The file is not found", pb_utils.TritonError.NOT_FOUND)
```

默认的错误码是：pb\_utils.TritonError.INTERNAL。当前支持的错误码如下：

*   `pb_utils.TritonError.UNKNOWN`
    
*   `pb_utils.TritonError.INTERNAL`
    
*   `pb_utils.TritonError.NOT_FOUND`
    
*   `pb_utils.TritonError.INVALID_ARG`
    
*   `pb_utils.TritonError.UNAVAILABLE`
    
*   `pb_utils.TritonError.UNSUPPORTED`
    
*   `pb_utils.TritonError.ALREADY_EXISTS`
    
*   `pb_utils.TritonError.CANCELLED` (since 23.10)
    

## 4.2 请求取消

在请求未被执行前，请求可能会被取消，此时Backend可以取消执行直接返回错误，当然Backend也可选择继续执行，只是会浪费资源。

```python
import triton_python_backend_utils as pb_utils

class TritonPythonModel:
    ...

    def execute(self, requests):
        responses = []

        for request in requests:
            if request.is_cancelled():
                responses.append(pb_utils.InferenceResponse(
                    error=pb_utils.TritonError("Message", pb_utils.TritonError.CANCELLED)))
            else:
                ...

        return responses
```

**模型并发执行**

Triton支持多模型并发执行，同时也支持每个模型多实例（Instance）并发执行。

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/J9LnWNMeRR2AlvDe/img/5c2592ad-714d-482c-8860-8c3973158758.png)

如果单模型只有一个实例，Triton会对请求进行排队。

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/J9LnWNMeRR2AlvDe/img/96772648-aac3-41e1-b75f-bdb47d347c8c.png)

如果单模型有多个实例，Triton会自动将请求分配给空闲的实例执行，余下的请求自动进入队列排队等候前面的请求处理结束。如下图，model1有三个实例，有4个请求并发，此时3个请求可以并发执行，第4个请求会被分配给最早处理完请求的实例执行。

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/J9LnWNMeRR2AlvDe/img/2b6faf8c-8c90-4127-9c7c-e77ee7635e04.png)

**请求合并动态批处理**

批量推理可以提高推理吞吐量和GPU的利用率，动态批处理指的是Triton服务端会组合推理请求，从而动态创建批处理来高吞吐量，这块内容由Triton的调度策略决定。  
默认的调度策略Default Scheduler不会主动合并请求，而是仅将所有请求分发到各个执行实例上，动态批处理策略**Dynamic Batcher**它可以在服务端将多个batch\_size较小的请求组合在一起形成一个batch\_size较大的任务，从而提高吞吐量和GPU利用率，Dynamic Batcher在config.pbtxt中进行指定，一个例子如下

```css
max_batch_size: 16
dynamic_batching {
    preferred_batch_size: [ 4, 8 ]
    max_queue_delay_microseconds: 100
}

```

*   **preferred\_batch\_size**：期望达到的batch\_size，可以指定一个数值，也可以是一个包含多个值的数组，本例代表期望组合成大小为4或者8的batch\_size，尽可能的将batch\_size组为指定的值，batch\_size不能超过max\_batch\_size
    
*   **max\_queue\_delay\_microseconds**：组合batch的最大时间限制，单位为微秒，本例代表组合batch最长时间为100微秒，超过这个时间则停止组合batch，会把已经打进batch的请求进行推理。这个时间限制越大，延迟越大，但是越容易组合到大的batch\_size，这个时间限制越小。延迟越小，但是吞吐量降低，因此该参数是一个延迟和吞吐之间的trade off
    

配置文件中的dynamic\_batching只是将请求进行聚合，在model.py的自定义后端中实际的作用是requests变成了多个request组成的集合，而不开启dynamic\_batching则requests里面只有一个request，这一点在execute方法的注释中有说明 \*\*“Depending on the batching configuration (e.g. Dynamic Batching) used, `requests` may contain multiple requests” \*\*

```python
    def execute(self, requests):
        """Depending on the batching configuration (e.g. Dynamic
        Batching) used, `requests` may contain multiple requests. 
        """
        for request in requests:
            pass

```

**模型调度**

Triton将模型调度方式分为三种：

*   Stateless：无状态，推理请求与推理响应是一一对应的。
    
*   Stateful：有状态，推理请求之间是有状态的，适用于多进一出的模型，多个请求会路由至同一个模型实例上。
    
*   Ensemble：组合模式。一般模型会有前处理、推理、后处理，如果分为三个请求来完成，效率不高，可以通过Ensemble模式将这三个步骤合并为一个请求，Triton负责将这三个步骤进行合并。
    

详细信息参考：

[Triton Architecture — NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/architecture.html?highlight=stateless#models-and-schedulers)

性能测试工具perf\_analyzer

这是官方提供的一款工具，作用是模拟用户发送请求来衡量当前服务的性能，包括吞吐量、延时等。该工具提供了大量参数帮助用户最大化还原真实的使用场景。同时，它还支持许多当前比较火的AI服务的评估，如LLM。

## 8.1 安装

官方推荐拉取官方提供的SDK docker：

```python
export RELEASE=<yy.mm> # e.g. to use the release from the end of February of 2023, do `export RELEASE=23.02`

docker pull nvcr.io/nvidia/tritonserver:${RELEASE}-py3-sdk
```

docker中已经包含编译好的perf\_analyzer。

另外也可以通过pip命令安装：

```python
pip install tritonclient==2.9.0
```

## 8.2 使用

```python
perf_analyzer -m vastpipe_passthrough --shape Input:3,224,224 --concurrency-range 1:8 -u localhost:8000
```

其中-u指定server url， 默认是localhost:8000， 如果Server所在机器上测试，则可省略。

常用参数：

```python
-m         模型名
-x         模型版本
--async   使用异步模式。如果模型是stateful-model（triton模型架构核心之一，后面单独说）则默认使用async；在同步模式下，perf_analyzer 将启动与并发级别相等的线程。使用异步模式可以限制线程的数量，同时保持并发性。
--sync      强制使用同步模式
--measurement-interval  数据统计采样间隔
--concurrency-range  并发数范围，'end' and --latency-threshold can not be both 0 simultaneously
--request-rate-range 请求发送速率变化范围，'end' and --latency-threshold can not be both 0 simultaneously. 并发数和请求速率不一样
--request-distribution  请求间隔的分布，必须搭配request-rate-range使用；
--request-intervals:  指定一个文件路径，这个文件包含request间隔数据，不能同request-rate-range、concurrency-range同时使用
--num-of-sequences  序列个数，当评估sequence-model时
--latency-threshold  指定延时阈值，超过这个值就中断
--max-threads  最大的线程数
--stability-percentage 指定延迟稳定性测量的波动范围（%）
--max-trials  最大尝试次数
--percentile  使用百分位来评估延迟是否稳定
-b batchsize数
--input-data 指定数据源
--shared-memory 指定共享内存类型
--output-shared-memory-size 指定output tensor能申请到的最大内存空间
--sequence-length  指定sequence的长度
--string-length 指定string类型输入的长度
-u 指定serve的地址
-i  指定通信类型

```

详细参数说明可参考：

[https://github.com/triton-inference-server/perf\_analyzer/blob/main/docs/cli.md](https://github.com/triton-inference-server/perf_analyzer/blob/main/docs/cli.md)