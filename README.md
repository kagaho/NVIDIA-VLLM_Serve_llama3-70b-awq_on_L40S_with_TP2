# Lab 1 — vLLM OpenAI Server: Llama 3 70B Instruct (AWQ) on NVIDIA L40S (TP=2)

This lab validates that **vLLM (OpenAI-compatible API server)** can serve **TechxGenus/Meta-Llama-3-70B-Instruct-AWQ** on **NVIDIA L40S GPUs** using **Tensor Parallelism = 2**, and captures the key artifacts/metrics needed for reproducibility and troubleshooting.

---

## STEPS:

- Downloaded and staged the **Llama 3 70B Instruct (AWQ)** model locally.
- Copied the model folder into the server under:
  - `/mnt/shockwave/soundwave/models/llama3-70b-awq`
- Started the **vLLM OpenAI server** in a Podman container:
  - Bound to host networking
  - Exposed on `http://0.0.0.0:8002`
  - Using **2 GPUs** (`tensor-parallel-size 2`)
  - Enabled **DEBUG logs**
  - Persisted logs to a mounted folder
- Tested inference using `curl` and measured latency.
- Queried **Prometheus metrics** from `/metrics`.
- Collected environment snapshot with `vllm collect-env`.

---


...

## Directory layout (server)

```bash
/mnt/shockwave/soundwave/
├── logs/
│   ├── vllm_tp2_default_DEBUG_8002.log
│   ├── vllm_tp2_awq.log
│   └── vllm_debug_eager_8005.log
├── models/
│   └── llama3-70b-awq/
│       ├── config.json
│       ├── generation_config.json
│       ├── model-00001-of-00009.safetensors
│       ├── ...
│       ├── model-00009-of-00009.safetensors
│       ├── model.safetensors.index.json
│       ├── tokenizer.json
│       ├── tokenizer_config.json
│       └── special_tokens_map.json
├── hf_cache/
└── vllm_cache/
```
```
podman run --rm -it --net=host \
  --hooks-dir=/usr/share/containers/oci/hooks.d \
  --security-opt=label=disable \
  --device nvidia.com/gpu=0 \
  --device nvidia.com/gpu=1 \
  -e HF_HOME=/mnt/shockwave/soundwave/hf_cache \
  -e VLLM_CACHE_DIR=/mnt/shockwave/soundwave/vllm_cache \
  -v /mnt/shockwave/soundwave:/mnt/shockwave/soundwave \
  --entrypoint /bin/bash \
  docker.io/vllm/vllm-openai:latest -lc \
  'vllm serve /mnt/shockwave/soundwave/models/llama3-70b-awq \
     --port 8002 \
     --tensor-parallel-size 2 \
     2>&1 | tee /mnt/shockwave/soundwave/logs/vllm_tp2_default_DEBUG_8002.log'
```

```
rteixeira@shockwave:~$ podman ps
CONTAINER ID  IMAGE                              COMMAND               CREATED         STATUS         PORTS       NAMES
36b35d9a98c7  docker.io/vllm/vllm-openai:latest  -lc vllm serve /m...  20 minutes ago  Up 20 minutes              fervent_lovelace
```


podman logs:
```
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:287]
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:287]        █     █     █▄   ▄█
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:287]  ▄▄ ▄█ █     █     █ ▀▄▀ █  version 0.16.0
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:287]   █▄█▀ █     █     █     █  model   /mnt/shockwave/soundwave/models/llama3-70b-awq
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:287]    ▀▀  ▀▀▀▀▀ ▀▀▀▀▀ ▀     ▀
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:287]
(APIServer pid=46) INFO 02-27 20:31:36 [utils.py:223] non-default args: {'model_tag': '/mnt/shockwave/soundwave/models/llama3-70b-awq', 'api_server_count': 1, 'port': 8002, 'model': '/mnt/shockwave/soundwave/models/llama3-70b-awq', 'tensor_parallel_size': 2}
(APIServer pid=46) INFO 02-27 20:31:43 [model.py:529] Resolved architecture: LlamaForCausalLM
(APIServer pid=46) INFO 02-27 20:31:43 [model.py:1549] Using max model len 8192
(APIServer pid=46) INFO 02-27 20:31:44 [awq_marlin.py:162] The model is convertible to awq_marlin during runtime. Using awq_marlin kernel.
(APIServer pid=46) INFO 02-27 20:31:44 [scheduler.py:224] Chunked prefill is enabled with max_num_batched_tokens=2048.
(APIServer pid=46) INFO 02-27 20:31:44 [vllm.py:689] Asynchronous scheduling is enabled.
(EngineCore_DP0 pid=314) INFO 02-27 20:31:51 [core.py:97] Initializing a V1 LLM engine (v0.16.0) with config: model='/mnt/shockwave/soundwave/models/llama3-70b-awq', speculative_config=None, tokenizer='/mnt/shockwave/soundwave/models/llama3-70b-awq', skip_tokenizer_init=False, tokenizer_mode=auto, revision=None, tokenizer_revision=None, trust_remote_code=False, dtype=torch.float16, max_seq_len=8192, download_dir=None, load_format=auto, tensor_parallel_size=2, pipeline_parallel_size=1, data_parallel_size=1, disable_custom_all_reduce=False, quantization=awq_marlin, enforce_eager=False, enable_return_routed_experts=False, kv_cache_dtype=auto, device_config=cuda, structured_outputs_config=StructuredOutputsConfig(backend='auto', disable_fallback=False, disable_any_whitespace=False, disable_additional_properties=False, reasoning_parser='', reasoning_parser_plugin='', enable_in_reasoning=False), observability_config=ObservabilityConfig(show_hidden_metrics_for_version=None, otlp_traces_endpoint=None, collect_detailed_traces=None, kv_cache_metrics=False, kv_cache_metrics_sample=0.01, cudagraph_metrics=False, enable_layerwise_nvtx_tracing=False, enable_mfu_metrics=False, enable_mm_processor_stats=False, enable_logging_iteration_details=False), seed=0, served_model_name=/mnt/shockwave/soundwave/models/llama3-70b-awq, enable_prefix_caching=True, enable_chunked_prefill=True, pooler_config=None, compilation_config={'level': None, 'mode': <CompilationMode.VLLM_COMPILE: 3>, 'debug_dump_path': None, 'cache_dir': '', 'compile_cache_save_format': 'binary', 'backend': 'inductor', 'custom_ops': ['none'], 'splitting_ops': ['vllm::unified_attention', 'vllm::unified_attention_with_output', 'vllm::unified_mla_attention', 'vllm::unified_mla_attention_with_output', 'vllm::mamba_mixer2', 'vllm::mamba_mixer', 'vllm::short_conv', 'vllm::linear_attention', 'vllm::plamo2_mamba_mixer', 'vllm::gdn_attention_core', 'vllm::kda_attention', 'vllm::sparse_attn_indexer', 'vllm::rocm_aiter_sparse_attn_indexer', 'vllm::unified_kv_cache_update'], 'compile_mm_encoder': False, 'compile_sizes': [], 'compile_ranges_split_points': [2048], 'inductor_compile_config': {'enable_auto_functionalized_v2': False, 'combo_kernels': True, 'benchmark_combo_kernel': True}, 'inductor_passes': {}, 'cudagraph_mode': <CUDAGraphMode.FULL_AND_PIECEWISE: (2, 1)>, 'cudagraph_num_of_warmups': 1, 'cudagraph_capture_sizes': [1, 2, 4, 8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 88, 96, 104, 112, 120, 128, 136, 144, 152, 160, 168, 176, 184, 192, 200, 208, 216, 224, 232, 240, 248, 256, 272, 288, 304, 320, 336, 352, 368, 384, 400, 416, 432, 448, 464, 480, 496, 512], 'cudagraph_copy_inputs': False, 'cudagraph_specialize_lora': True, 'use_inductor_graph_partition': False, 'pass_config': {'fuse_norm_quant': False, 'fuse_act_quant': False, 'fuse_attn_quant': False, 'eliminate_noops': True, 'enable_sp': False, 'fuse_gemm_comms': False, 'fuse_allreduce_rms': False, 'fuse_act_padding': False}, 'max_cudagraph_capture_size': 512, 'dynamic_shapes_config': {'type': <DynamicShapesType.BACKED: 'backed'>, 'evaluate_guards': False, 'assume_32_bit_indexing': False}, 'local_cache_dir': None, 'fast_moe_cold_start': True, 'static_all_moe_layers': []}
(EngineCore_DP0 pid=314) WARNING 02-27 20:31:51 [multiproc_executor.py:921] Reducing Torch parallelism from 64 threads to 1 to avoid unnecessary CPU contention. Set OMP_NUM_THREADS in the external environment to tune this value as needed.
INFO 02-27 20:31:57 [parallel_state.py:1234] world_size=2 rank=0 local_rank=0 distributed_init_method=tcp://127.0.0.1:35273 backend=nccl
INFO 02-27 20:31:57 [parallel_state.py:1234] world_size=2 rank=1 local_rank=1 distributed_init_method=tcp://127.0.0.1:35273 backend=nccl
INFO 02-27 20:31:57 [pynccl.py:111] vLLM is using nccl==2.27.5
WARNING 02-27 20:31:58 [symm_mem.py:67] SymmMemCommunicator: Device capability 8.9 not supported, communicator is not available.
WARNING 02-27 20:31:58 [symm_mem.py:67] SymmMemCommunicator: Device capability 8.9 not supported, communicator is not available.
INFO 02-27 20:31:58 [parallel_state.py:1445] rank 1 in world size 2 is assigned as DP rank 0, PP rank 0, PCP rank 0, TP rank 1, EP rank N/A
INFO 02-27 20:31:58 [parallel_state.py:1445] rank 0 in world size 2 is assigned as DP rank 0, PP rank 0, PCP rank 0, TP rank 0, EP rank N/A
(Worker_TP0 pid=512) INFO 02-27 20:31:59 [gpu_model_runner.py:4124] Starting to load model /mnt/shockwave/soundwave/models/llama3-70b-awq...
(Worker_TP0 pid=512) INFO 02-27 20:32:17 [cuda.py:367] Using FLASH_ATTN attention backend out of potential backends: ['FLASH_ATTN', 'FLASHINFER', 'TRITON_ATTN', 'FLEX_ATTENTION'].
Loading safetensors checkpoint shards:   0% Completed | 0/9 [00:00<?, ?it/s]
Loading safetensors checkpoint shards:  11% Completed | 1/9 [00:01<00:08,  1.10s/it]
Loading safetensors checkpoint shards:  22% Completed | 2/9 [00:02<00:09,  1.29s/it]
Loading safetensors checkpoint shards:  33% Completed | 3/9 [00:03<00:08,  1.35s/it]
Loading safetensors checkpoint shards:  44% Completed | 4/9 [00:05<00:06,  1.38s/it]
Loading safetensors checkpoint shards:  56% Completed | 5/9 [00:06<00:05,  1.40s/it]
Loading safetensors checkpoint shards:  67% Completed | 6/9 [00:08<00:04,  1.40s/it]
Loading safetensors checkpoint shards:  78% Completed | 7/9 [00:09<00:02,  1.40s/it]
Loading safetensors checkpoint shards:  89% Completed | 8/9 [00:10<00:01,  1.27s/it]
Loading safetensors checkpoint shards: 100% Completed | 9/9 [00:10<00:00,  1.04it/s]
Loading safetensors checkpoint shards: 100% Completed | 9/9 [00:10<00:00,  1.21s/it]
(Worker_TP0 pid=512)
(Worker_TP0 pid=512) INFO 02-27 20:32:28 [default_loader.py:293] Loading weights took 10.90 seconds
(Worker_TP0 pid=512) INFO 02-27 20:32:31 [gpu_model_runner.py:4221] Model loading took 18.55 GiB memory and 31.238919 seconds
(Worker_TP0 pid=512) INFO 02-27 20:32:43 [backends.py:916] Using cache directory: /root/.cache/vllm/torch_compile_cache/a75a3a1586/rank_0_0/backbone for vLLM's torch.compile
(Worker_TP0 pid=512) INFO 02-27 20:32:43 [backends.py:976] Dynamo bytecode transform time: 11.70 s
(Worker_TP0 pid=512) INFO 02-27 20:32:55 [backends.py:351] Cache the graph of compile range (1, 2048) for later use
(Worker_TP1 pid=513) INFO 02-27 20:32:59 [backends.py:351] Cache the graph of compile range (1, 2048) for later use
(Worker_TP0 pid=512) INFO 02-27 20:33:04 [backends.py:368] Compiling a graph for compile range (1, 2048) takes 13.42 s
(Worker_TP0 pid=512) INFO 02-27 20:33:04 [monitor.py:34] torch.compile takes 25.12 s in total
(Worker_TP0 pid=512) INFO 02-27 20:33:05 [gpu_worker.py:373] Available KV cache memory: 20.16 GiB
(EngineCore_DP0 pid=314) INFO 02-27 20:33:05 [kv_cache_utils.py:1307] GPU KV cache size: 132,128 tokens
(EngineCore_DP0 pid=314) INFO 02-27 20:33:05 [kv_cache_utils.py:1312] Maximum concurrency for 8,192 tokens per request: 16.13x
Capturing CUDA graphs (mixed prefill-decode, PIECEWISE): 100%|██████████| 51/51 [00:06<00:00,  8.36it/s]
Capturing CUDA graphs (decode, FULL): 100%|██████████| 35/35 [00:02<00:00, 12.36it/s]
(Worker_TP0 pid=512) INFO 02-27 20:33:14 [custom_all_reduce.py:216] Registering 13685 cuda graph addresses
(Worker_TP1 pid=513) INFO 02-27 20:33:15 [custom_all_reduce.py:216] Registering 13685 cuda graph addresses
(Worker_TP0 pid=512) INFO 02-27 20:33:15 [gpu_model_runner.py:5246] Graph capturing finished in 10 secs, took 1.22 GiB
(EngineCore_DP0 pid=314) INFO 02-27 20:33:15 [core.py:278] init engine (profile, create kv cache, warmup model) took 44.16 seconds
(EngineCore_DP0 pid=314) INFO 02-27 20:33:15 [vllm.py:689] Asynchronous scheduling is enabled.
(APIServer pid=46) INFO 02-27 20:33:16 [api_server.py:481] Supported tasks: ['generate']
(APIServer pid=46) INFO 02-27 20:33:16 [serving.py:188] Warming up chat template processing...
(APIServer pid=46) INFO 02-27 20:33:16 [hf.py:318] Detected the chat template content format to be 'string'. You can set `--chat-template-content-format` to override this.
(APIServer pid=46) INFO 02-27 20:33:16 [serving.py:213] Chat template warmup completed in 234.6ms
(APIServer pid=46) INFO 02-27 20:33:16 [api_server.py:486] Starting vLLM API server 0 on http://0.0.0.0:8002
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:38] Available routes are:
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /openapi.json, Methods: GET, HEAD
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /docs, Methods: GET, HEAD
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /docs/oauth2-redirect, Methods: GET, HEAD
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /redoc, Methods: GET, HEAD
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /load, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /version, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /scale_elastic_ep, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /is_scaling_elastic_ep, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /tokenize, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /detokenize, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /inference/v1/generate, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /metrics, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /health, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/models, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /ping, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /ping, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /invocations, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/chat/completions, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/chat/completions/render, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/responses, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/responses/{response_id}, Methods: GET
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/responses/{response_id}/cancel, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/completions, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/completions/render, Methods: POST
(APIServer pid=46) INFO 02-27 20:33:16 [launcher.py:47] Route: /v1/messages, Methods: POST
(APIServer pid=46) INFO:     Started server process [46]
(APIServer pid=46) INFO:     Waiting for application startup.
(APIServer pid=46) INFO:     Application startup complete.
(APIServer pid=46) INFO 02-27 20:33:26 [loggers.py:259] Engine 000: Avg prompt throughput: 2.2 tokens/s, Avg generation throughput: 6.3 tokens/s, Running: 1 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.1%, Prefix cache hit rate: 0.0%
(APIServer pid=46) INFO:     127.0.0.1:54212 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=46) INFO 02-27 20:33:36 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 33.3 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 0.0%
(APIServer pid=46) INFO 02-27 20:33:46 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 0.0 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 0.0%
(APIServer pid=46) INFO:     127.0.0.1:35378 - "GET /metrics HTTP/1.1" 200 OK
(APIServer pid=46) INFO:     127.0.0.1:53558 - "GET /metrics HTTP/1.1" 200 OK
(APIServer pid=46) INFO:     127.0.0.1:41798 - "GET /metrics HTTP/1.1" 200 OK
```


time curl -s http://localhost:8002/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "/mnt/shockwave/soundwave/models/llama3-70b-awq",
    "messages": [{"role":"user","content":"Write exactly 400 tokens about GPU tensor parallelism. No bullets."}],
    "max_tokens": 400,
    "temperature": 0.2
  }' > /dev/null

```
{"id":"chatcmpl-817f6b7680434c82","object":"chat.completion","created":1772224404,"model":"/mnt/shockwave/soundwave/models/llama3-70b-awq","choices":[{"index":0,"message":{"role":"assistant","content":"GPU tensor parallelism is a technique used to accelerate deep learning computations by parallelizing tensor operations across multiple GPUs. This approach takes advantage of the fact that many deep learning models can be split into smaller sub-tasks that can be executed independently, allowing for significant speedups in training and inference times.\n\nBy dividing the model into smaller tensors and processing them in parallel across multiple GPUs, tensor parallelism can achieve near-linear scaling with the number of GPUs used. This is particularly useful for large-scale deep learning models that require massive computational resources to train.\n\nOne of the key challenges in implementing tensor parallelism is ensuring that the parallelized operations are correctly synchronized and communicated across GPUs. This requires careful management of data transfer and synchronization between GPUs, as well as optimized memory allocation and access patterns.\n\nTo address these challenges, various parallelization strategies have been developed, including data parallelism, model parallelism, and pipeline parallelism. Data parallelism involves dividing the input data into smaller chunks and processing them in parallel across multiple GPUs, while model parallelism involves splitting the model into smaller components and processing them in parallel.\n\nPipeline parallelism, on the other hand, involves dividing the computation into a series of stages, each of which is executed on a different GPU. This approach can be particularly effective for models with complex dependencies between layers.\n\nSeveral deep learning frameworks, including TensorFlow and PyTorch, have implemented tensor parallelism using various parallelization strategies. These frameworks provide APIs and tools for developers to easily parallelize their models and take advantage of multiple GPUs.\n\nIn addition to deep learning frameworks, several specialized libraries and tools have been developed to support tensor parallelism, including NCCL, a library for collective operations, and GPUDirect, a technology for direct GPU-to-GPU communication.\n\nTensor parallelism has been widely adopted in various applications, including natural language processing, computer vision, and recommender systems. It has been shown to achieve significant speedups in training and inference times, making it an essential technique for scaling deep learning","refusal":null,"annotations":null,"audio":null,"function_call":null,"tool_calls":[],"reasoning":null},"logprobs":null,"finish_reason":"length","stop_reason":null,"token_ids":null}],"service_tier":null,"system_fingerprint":null,"usage":{"prompt_tokens":24,"total_tokens":424,"completion_tokens":400,"prompt_tokens_details":null},"prompt_logprobs":null,"prompt_token_ids":null,"kv_transfer_params":null}
real	0m11.773s
user	0m0.004s
sys	0m0.012s
```


