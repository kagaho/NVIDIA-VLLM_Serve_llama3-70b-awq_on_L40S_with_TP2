### Redo the VLLM Container:

#### Activity:  

- Add Nsight app, nsys profile and write the report to mounted volume and run the container for the interval of 240 seconds, with a warm up of 30 seconds(important), promoting the nsys profiling
- Run, as much as oyou want, the curl commands below. 
- When the container terminates, at the Server side, run the report

```bash
podman run --rm -it --net=host \
  --name vllm-llama3-70b-awq \
  --hooks-dir=/usr/share/containers/oci/hooks.d \
  --security-opt=label=disable \
  --security-opt=seccomp=unconfined \
  --cap-add=SYS_ADMIN \
  --cap-add=SYS_PTRACE \
  --cap-add=PERFMON \
  --device nvidia.com/gpu=0 \
  --device nvidia.com/gpu=1 \
  -e HF_HOME=/mnt/elita/soundwave/hf_cache \
  -e VLLM_CACHE_DIR=/mnt/elita/soundwave/vllm_cache \
  -v /mnt/elita/soundwave:/mnt/elita/soundwave \
  -v /opt/nvidia/nsight-systems/2025.5.2:/opt/nsys:ro \
  --entrypoint /bin/bash \
  docker.io/vllm/vllm-openai:latest -lc \
  'set -o pipefail
   mkdir -p /mnt/elita/soundwave/profiles
   /opt/nsys/bin/nsys profile \
     -o /mnt/elita/soundwave/profiles/vllm_tp2_$(date +%F_%H%M%S) \
     --trace=cuda,nvtx,osrt \
     --cuda-graph-trace=node \
     --delay 30 \
     --duration 240 \
     vllm serve /mnt/elita/soundwave/models/llama3-70b-awq \
       --port 8002 \
       --tensor-parallel-size 2 \
     2>&1 | tee /mnt/elita/soundwave/logs/vllm_tp2_nsys_8002.log'

```
  
- From the container we can see we have now nsight mapped into /opt:
  
```bash
root@elita:/opt/nsys# /opt/nsys/bin/nsys --version 
NVIDIA Nsight Systems version 2025.5.2.266-255236693005v0
```

- run some some user queries with some load, it is cool to see more concurrent user, batched more token/second:
  
- single user query:
```
curl -s http://127.0.0.1:8002/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "/mnt/elita/soundwave/models/llama3-70b-awq",
    "messages": [{"role":"user","content":"Summarize the tradeoffs between tensor parallelism and pipeline parallelism in LLM serving."}],
    "max_tokens": 800,
    "temperature": 0.2,
    "stream": false
  }'
```
  
  
- multiple users(20 concurrent requests):  
```
seq 1 20 | xargs -P 20 -I{} sh -c '
  curl -s -o /dev/null -w "user={} status=%{http_code} time_total=%{time_total}\n" \
    http://127.0.0.1:8002/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d "{\"model\":\"/mnt/elita/soundwave/models/llama3-70b-awq\",\"user\":\"user-{}\",\"messages\":[{\"role\":\"user\",\"content\":\"User {}: explain CUDA graphs in 3 sentences.\"}],\"max_tokens\":256,\"temperature\":0.2}"
'

- let's do even more tests:
  
```
seq 1 100 | xargs -P 20 -I{} sh -c '
  u=$(( ({}-1)%20 + 1 ))
  curl -s -o /dev/null -w "req={} user=$u status=%{http_code} t=%{time_total}\n" \
    http://127.0.0.1:8002/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d "{\"model\":\"/mnt/elita/soundwave/models/llama3-70b-awq\",\"user\":\"user-$u\",\"messages\":[{\"role\":\"user\",\"content\":\"User $u request {}: give 5 bullet points about tensor parallelism.\"}],\"max_tokens\":256,\"temperature\":0.2}"
```

### Output:  

```bash
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:287]
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:287]        █     █     █▄   ▄█
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:287]  ▄▄ ▄█ █     █     █ ▀▄▀ █  version 0.16.0
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:287]   █▄█▀ █     █     █     █  model   /mnt/elita/soundwave/models/llama3-70b-awq
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:287]    ▀▀  ▀▀▀▀▀ ▀▀▀▀▀ ▀     ▀
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:287]
(APIServer pid=78) INFO 03-02 15:20:10 [utils.py:223] non-default args: {'model_tag': '/mnt/elita/soundwave/models/llama3-70b-awq', 'api_server_count': 1, 'port': 8002, 'model': '/mnt/elita/soundwave/models/llama3-70b-awq', 'tensor_parallel_size': 2}
(APIServer pid=78) INFO 03-02 15:20:15 [model.py:529] Resolved architecture: LlamaForCausalLM
(APIServer pid=78) INFO 03-02 15:20:15 [model.py:1549] Using max model len 8192
(APIServer pid=78) INFO 03-02 15:20:15 [awq_marlin.py:162] The model is convertible to awq_marlin during runtime. Using awq_marlin kernel.
(APIServer pid=78) INFO 03-02 15:20:15 [scheduler.py:224] Chunked prefill is enabled with max_num_batched_tokens=2048.
(APIServer pid=78) INFO 03-02 15:20:15 [vllm.py:689] Asynchronous scheduling is enabled.
(EngineCore_DP0 pid=363) INFO 03-02 15:20:21 [core.py:97] Initializing a V1 LLM engine (v0.16.0) with config: model='/mnt/elita/soundwave/models/llama3-70b-awq', speculative_config=None, tokenizer='/mnt/elita/soundwave/models/llama3-70b-awq', skip_tokenizer_init=False, tokenizer_mode=auto, revision=None, tokenizer_revision=None, trust_remote_code=False, dtype=torch.float16, max_seq_len=8192, download_dir=None, load_format=auto, tensor_parallel_size=2, pipeline_parallel_size=1, data_parallel_size=1, disable_custom_all_reduce=False, quantization=awq_marlin, enforce_eager=False, enable_return_routed_experts=False, kv_cache_dtype=auto, device_config=cuda, structured_outputs_config=StructuredOutputsConfig(backend='auto', disable_fallback=False, disable_any_whitespace=False, disable_additional_properties=False, reasoning_parser='', reasoning_parser_plugin='', enable_in_reasoning=False), observability_config=ObservabilityConfig(show_hidden_metrics_for_version=None, otlp_traces_endpoint=None, collect_detailed_traces=None, kv_cache_metrics=False, kv_cache_metrics_sample=0.01, cudagraph_metrics=False, enable_layerwise_nvtx_tracing=False, enable_mfu_metrics=False, enable_mm_processor_stats=False, enable_logging_iteration_details=False), seed=0, served_model_name=/mnt/elita/soundwave/models/llama3-70b-awq, enable_prefix_caching=True, enable_chunked_prefill=True, pooler_config=None, compilation_config={'level': None, 'mode': <CompilationMode.VLLM_COMPILE: 3>, 'debug_dump_path': None, 'cache_dir': '', 'compile_cache_save_format': 'binary', 'backend': 'inductor', 'custom_ops': ['none'], 'splitting_ops': ['vllm::unified_attention', 'vllm::unified_attention_with_output', 'vllm::unified_mla_attention', 'vllm::unified_mla_attention_with_output', 'vllm::mamba_mixer2', 'vllm::mamba_mixer', 'vllm::short_conv', 'vllm::linear_attention', 'vllm::plamo2_mamba_mixer', 'vllm::gdn_attention_core', 'vllm::kda_attention', 'vllm::sparse_attn_indexer', 'vllm::rocm_aiter_sparse_attn_indexer', 'vllm::unified_kv_cache_update'], 'compile_mm_encoder': False, 'compile_sizes': [], 'compile_ranges_split_points': [2048], 'inductor_compile_config': {'enable_auto_functionalized_v2': False, 'combo_kernels': True, 'benchmark_combo_kernel': True}, 'inductor_passes': {}, 'cudagraph_mode': <CUDAGraphMode.FULL_AND_PIECEWISE: (2, 1)>, 'cudagraph_num_of_warmups': 1, 'cudagraph_capture_sizes': [1, 2, 4, 8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 88, 96, 104, 112, 120, 128, 136, 144, 152, 160, 168, 176, 184, 192, 200, 208, 216, 224, 232, 240, 248, 256, 272, 288, 304, 320, 336, 352, 368, 384, 400, 416, 432, 448, 464, 480, 496, 512], 'cudagraph_copy_inputs': False, 'cudagraph_specialize_lora': True, 'use_inductor_graph_partition': False, 'pass_config': {'fuse_norm_quant': False, 'fuse_act_quant': False, 'fuse_attn_quant': False, 'eliminate_noops': True, 'enable_sp': False, 'fuse_gemm_comms': False, 'fuse_allreduce_rms': False, 'fuse_act_padding': False}, 'max_cudagraph_capture_size': 512, 'dynamic_shapes_config': {'type': <DynamicShapesType.BACKED: 'backed'>, 'evaluate_guards': False, 'assume_32_bit_indexing': False}, 'local_cache_dir': None, 'fast_moe_cold_start': True, 'static_all_moe_layers': []}
(EngineCore_DP0 pid=363) WARNING 03-02 15:20:21 [multiproc_executor.py:921] Reducing Torch parallelism from 64 threads to 1 to avoid unnecessary CPU contention. Set OMP_NUM_THREADS in the external environment to tune this value as needed.
INFO 03-02 15:20:26 [parallel_state.py:1234] world_size=2 rank=1 local_rank=1 distributed_init_method=tcp://127.0.0.1:60717 backend=nccl
INFO 03-02 15:20:27 [parallel_state.py:1234] world_size=2 rank=0 local_rank=0 distributed_init_method=tcp://127.0.0.1:60717 backend=nccl
INFO 03-02 15:20:27 [pynccl.py:111] vLLM is using nccl==2.27.5
WARNING 03-02 15:20:27 [symm_mem.py:67] SymmMemCommunicator: Device capability 8.9 not supported, communicator is not available.
WARNING 03-02 15:20:27 [symm_mem.py:67] SymmMemCommunicator: Device capability 8.9 not supported, communicator is not available.
INFO 03-02 15:20:28 [parallel_state.py:1445] rank 1 in world size 2 is assigned as DP rank 0, PP rank 0, PCP rank 0, TP rank 1, EP rank N/A
INFO 03-02 15:20:28 [parallel_state.py:1445] rank 0 in world size 2 is assigned as DP rank 0, PP rank 0, PCP rank 0, TP rank 0, EP rank N/A
(Worker_TP0 pid=570) INFO 03-02 15:20:28 [gpu_model_runner.py:4124] Starting to load model /mnt/elita/soundwave/models/llama3-70b-awq...
(Worker_TP0 pid=570) INFO 03-02 15:20:46 [cuda.py:367] Using FLASH_ATTN attention backend out of potential backends: ['FLASH_ATTN', 'FLASHINFER', 'TRITON_ATTN', 'FLEX_ATTENTION'].
Loading safetensors checkpoint shards:   0% Completed | 0/9 [00:00<?, ?it/s]
Loading safetensors checkpoint shards:  11% Completed | 1/9 [00:00<00:05,  1.52it/s]
Loading safetensors checkpoint shards:  22% Completed | 2/9 [00:01<00:05,  1.24it/s]
Loading safetensors checkpoint shards:  33% Completed | 3/9 [00:02<00:05,  1.17it/s]
Loading safetensors checkpoint shards:  44% Completed | 4/9 [00:03<00:04,  1.14it/s]
Loading safetensors checkpoint shards:  56% Completed | 5/9 [00:04<00:03,  1.13it/s]
Loading safetensors checkpoint shards:  67% Completed | 6/9 [00:05<00:02,  1.16it/s]
Loading safetensors checkpoint shards:  78% Completed | 7/9 [00:05<00:01,  1.21it/s]
Loading safetensors checkpoint shards:  89% Completed | 8/9 [00:06<00:00,  1.37it/s]
Loading safetensors checkpoint shards: 100% Completed | 9/9 [00:06<00:00,  1.87it/s]
Loading safetensors checkpoint shards: 100% Completed | 9/9 [00:06<00:00,  1.38it/s]
(Worker_TP0 pid=570)
(Worker_TP0 pid=570) INFO 03-02 15:20:53 [default_loader.py:293] Loading weights took 6.50 seconds
(Worker_TP0 pid=570) INFO 03-02 15:20:55 [gpu_model_runner.py:4221] Model loading took 18.55 GiB memory and 25.717714 seconds
(Worker_TP0 pid=570) INFO 03-02 15:21:05 [backends.py:916] Using cache directory: /root/.cache/vllm/torch_compile_cache/a75a3a1586/rank_0_0/backbone for vLLM's torch.compile
(Worker_TP0 pid=570) INFO 03-02 15:21:05 [backends.py:976] Dynamo bytecode transform time: 10.61 s

(Worker_TP0 pid=570) INFO 03-02 15:21:23 [backends.py:351] Cache the graph of compile range (1, 2048) for later use
(Worker_TP1 pid=571) INFO 03-02 15:21:23 [backends.py:351] Cache the graph of compile range (1, 2048) for later use
(Worker_TP0 pid=570) INFO 03-02 15:21:31 [backends.py:368] Compiling a graph for compile range (1, 2048) takes 15.00 s
(Worker_TP0 pid=570) INFO 03-02 15:21:31 [monitor.py:34] torch.compile takes 25.60 s in total
(Worker_TP0 pid=570) INFO 03-02 15:21:32 [gpu_worker.py:373] Available KV cache memory: 20.16 GiB
(EngineCore_DP0 pid=363) INFO 03-02 15:21:32 [kv_cache_utils.py:1307] GPU KV cache size: 132,128 tokens
(EngineCore_DP0 pid=363) INFO 03-02 15:21:32 [kv_cache_utils.py:1312] Maximum concurrency for 8,192 tokens per request: 16.13x
Capturing CUDA graphs (mixed prefill-decode, PIECEWISE): 100%|██████████| 51/51 [00:07<00:00,  7.12it/s]
Capturing CUDA graphs (decode, FULL): 100%|██████████| 35/35 [00:03<00:00, 10.63it/s]
(Worker_TP0 pid=570) INFO 03-02 15:21:43 [custom_all_reduce.py:216] Registering 13685 cuda graph addresses
(Worker_TP1 pid=571) INFO 03-02 15:21:44 [custom_all_reduce.py:216] Registering 13685 cuda graph addresses
(Worker_TP0 pid=570) INFO 03-02 15:21:44 [gpu_model_runner.py:5246] Graph capturing finished in 12 secs, took 1.22 GiB
(EngineCore_DP0 pid=363) INFO 03-02 15:21:44 [core.py:278] init engine (profile, create kv cache, warmup model) took 49.81 seconds
(EngineCore_DP0 pid=363) INFO 03-02 15:21:45 [vllm.py:689] Asynchronous scheduling is enabled.
(APIServer pid=78) INFO 03-02 15:21:45 [api_server.py:481] Supported tasks: ['generate']
(APIServer pid=78) INFO 03-02 15:21:45 [serving.py:188] Warming up chat template processing...
(APIServer pid=78) INFO 03-02 15:21:46 [hf.py:318] Detected the chat template content format to be 'string'. You can set `--chat-template-content-format` to override this.
(APIServer pid=78) INFO 03-02 15:21:46 [serving.py:213] Chat template warmup completed in 340.4ms
(APIServer pid=78) INFO 03-02 15:21:46 [api_server.py:486] Starting vLLM API server 0 on http://0.0.0.0:8002
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:38] Available routes are:
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /openapi.json, Methods: HEAD, GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /docs, Methods: HEAD, GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /docs/oauth2-redirect, Methods: HEAD, GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /redoc, Methods: HEAD, GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /load, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /version, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /scale_elastic_ep, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /is_scaling_elastic_ep, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /tokenize, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /detokenize, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /inference/v1/generate, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /metrics, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /health, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/models, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /ping, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /ping, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /invocations, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/chat/completions, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/chat/completions/render, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/responses, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/responses/{response_id}, Methods: GET
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/responses/{response_id}/cancel, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/completions, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/completions/render, Methods: POST
(APIServer pid=78) INFO 03-02 15:21:46 [launcher.py:47] Route: /v1/messages, Methods: POST
(APIServer pid=78) INFO:     Started server process [78]
(APIServer pid=78) INFO:     Waiting for application startup.
(APIServer pid=78) INFO:     Application startup complete.
(APIServer pid=78) INFO 03-02 15:21:56 [loggers.py:259] Engine 000: Avg prompt throughput: 2.7 tokens/s, Avg generation throughput: 17.5 tokens/s, Running: 1 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.2%, Prefix cache hit rate: 0.0%
(APIServer pid=78) INFO:     127.0.0.1:50704 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:22:06 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 30.2 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 0.0%
(APIServer pid=78) INFO 03-02 15:22:16 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 0.0 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 0.0%
(APIServer pid=78) INFO:     127.0.0.1:37322 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:22:26 [loggers.py:259] Engine 000: Avg prompt throughput: 2.3 tokens/s, Avg generation throughput: 14.0 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 0.0%
(APIServer pid=78) INFO:     127.0.0.1:56602 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:22:36 [loggers.py:259] Engine 000: Avg prompt throughput: 0.7 tokens/s, Avg generation throughput: 16.9 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 21.3%
(APIServer pid=78) INFO 03-02 15:22:46 [loggers.py:259] Engine 000: Avg prompt throughput: 1.3 tokens/s, Avg generation throughput: 2.0 tokens/s, Running: 1 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 30.8%
(APIServer pid=78) INFO 03-02 15:22:56 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 34.0 tokens/s, Running: 1 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.3%, Prefix cache hit rate: 30.8%
(APIServer pid=78) INFO:     127.0.0.1:55320 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:23:06 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 7.5 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 30.8%
(APIServer pid=78) INFO 03-02 15:23:16 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 0.0 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 30.8%

(APIServer pid=78) INFO:     127.0.0.1:37422 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37372 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37352 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37408 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37348 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37488 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37364 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37470 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37398 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37430 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37318 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37388 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37410 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37480 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37494 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37380 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37442 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37452 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37456 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:37332 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:24:06 [loggers.py:259] Engine 000: Avg prompt throughput: 44.0 tokens/s, Avg generation throughput: 212.9 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 5.9%
(APIServer pid=78) INFO 03-02 15:24:16 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 0.0 tokens/s, Running: 0 reqs, Waiting: 0 reqs, GPU KV cache usage: 0.0%, Prefix cache hit rate: 5.9%
(APIServer pid=78) INFO 03-02 15:24:26 [loggers.py:259] Engine 000: Avg prompt throughput: 54.0 tokens/s, Avg generation throughput: 240.2 tokens/s, Running: 20 reqs, Waiting: 0 reqs, GPU KV cache usage: 2.4%, Prefix cache hit rate: 3.0%
(APIServer pid=78) INFO:     127.0.0.1:51550 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51552 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51556 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51568 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51580 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51584 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51592 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51598 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51614 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51626 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51628 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51636 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51650 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51658 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51670 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51682 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51684 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51694 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51710 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:51722 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:24:36 [loggers.py:259] Engine 000: Avg prompt throughput: 54.0 tokens/s, Avg generation throughput: 542.1 tokens/s, Running: 20 reqs, Waiting: 0 reqs, GPU KV cache usage: 2.7%, Prefix cache hit rate: 2.0%
(APIServer pid=78) INFO:     127.0.0.1:43720 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43728 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43736 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43744 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43750 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43760 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43766 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43778 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43780 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43792 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43794 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43804 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43810 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43820 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43824 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43828 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43830 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43838 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43850 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:43862 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:24:46 [loggers.py:259] Engine 000: Avg prompt throughput: 54.0 tokens/s, Avg generation throughput: 544.7 tokens/s, Running: 20 reqs, Waiting: 0 reqs, GPU KV cache usage: 2.9%, Prefix cache hit rate: 1.5%
(APIServer pid=78) INFO:     127.0.0.1:42812 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42828 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42836 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42840 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42844 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42854 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42860 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42862 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42876 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42886 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42894 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42910 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42920 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42926 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42928 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42944 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42950 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42966 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42968 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:42978 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO 03-02 15:24:56 [loggers.py:259] Engine 000: Avg prompt throughput: 54.0 tokens/s, Avg generation throughput: 544.0 tokens/s, Running: 20 reqs, Waiting: 0 reqs, GPU KV cache usage: 3.1%, Prefix cache hit rate: 1.2%
(APIServer pid=78) INFO:     127.0.0.1:48326 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48340 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48354 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48356 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48358 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48362 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48370 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48372 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48388 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48402 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48406 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48422 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48438 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48444 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48450 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48460 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48472 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48480 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48484 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(APIServer pid=78) INFO:     127.0.0.1:48494 - "POST /v1/chat/completions HTTP/1.1" 200 OK
(Worker_TP1 pid=571) WARNING 03-02 15:25:05 [multiproc_executor.py:797] WorkerProc was terminated
(Worker_TP0 pid=570) WARNING 03-02 15:25:05 [multiproc_executor.py:797] WorkerProc was terminated
(Worker_TP1 pid=571) INFO 03-02 15:25:05 [multiproc_executor.py:732] Parent process exited, terminating worker
(Worker_TP0 pid=570) INFO 03-02 15:25:05 [multiproc_executor.py:732] Parent process exited, terminating worker
(APIServer pid=78) INFO 03-02 15:25:05 [launcher.py:122] Shutting down FastAPI HTTP server.
Collecting data...
Generating '/tmp/nsys-report-4c90.qdstrm'
[1/1] [0%                          ] vllm_tp2_2026-03-02_151959.nsys-rep(APIServer pid=78) INFO:     Shutting down
[1/1] [8%                          ] vllm_tp2_2026-03-02_151959.nsys-rep(APIServer pid=78) INFO 03-02 15:25:08 [loggers.py:259] Engine 000: Avg prompt throughput: 45.2 tokens/s, Avg generation throughput: 422.4 tokens/s, Running: 20 reqs, Waiting: 0 reqs, GPU KV cache usage: 2.9%, Prefix cache hit rate: 1.0%
[1/1] [8%                          ] vllm_tp2_2026-03-02_151959.nsys-rep(APIServer pid=78) INFO:     Waiting for connections to close. (CTRL+C to force quit)
[1/1] [11%                         ] vllm_tp2_2026-03-02_151959.nsys-rep(APIServer pid=78) INFO 03-02 15:25:18 [loggers.py:259] Engine 000: Avg prompt throughput: 0.0 tokens/s, Avg generation throughput: 0.0 tokens/s, Running: 20 reqs, Waiting: 0 reqs, GPU KV cache usage: 2.9%, Prefix cache hit rate: 1.0%
[1/1] [========================100%] vllm_tp2_2026-03-02_151959.nsys-rep
Generated:
	/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.nsys-rep

(APIServer pid=78) INFO:     Waiting for application shutdown.
(APIServer pid=78) INFO:     Application shutdown complete.
```

### Nsight collection  
  
```
rteixeira@elita:/mnt/elita/soundwave$ /opt/nvidia/nsight-systems/2025.5.2/bin/nsys stats \
  /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.nsys-rep
Generating SQLite file /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite from /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.nsys-rep
Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/nvtx_sum.py]...

 ** NVTX Range Summary (nvtx_sum):

 Time (%)  Total Time (ns)  Instances    Avg (ns)      Med (ns)     Min (ns)    Max (ns)   StdDev (ns)   Style                 Range
 --------  ---------------  ---------  ------------  ------------  ----------  ----------  -----------  -------  ----------------------------------
     44.6      170,465,528      3,262      52,258.0      25,762.0      19,794  41,381,780  1,023,730.0  PushPop  NCCL:ncclAllGather
     33.3      127,353,429      1,610      79,101.5      20,161.0      12,248   1,271,030    228,181.3  PushPop  NCCL:ncclAllReduce
     21.9       83,700,201          2  41,850,100.5  41,850,100.5  41,671,744  42,028,457    252,234.2  PushPop  NCCL:ncclCommInitRankConfig
      0.2          805,470          4     201,367.5      49,359.0      44,092     662,660    307,538.8  PushPop  CCCL:cub::DeviceSegmentedRadixSort

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/osrt_sum.py]...

 ** OS Runtime Summary (osrt_sum):

 Time (%)   Total Time (ns)   Num Calls      Avg (ns)          Med (ns)         Min (ns)        Max (ns)       StdDev (ns)             Name
 --------  -----------------  ---------  ----------------  ----------------  --------------  --------------  ---------------  ----------------------
     33.2  4,310,024,289,063     60,168      71,633,165.3     100,104,774.0           1,000   4,000,188,546    199,468,786.3  pthread_cond_timedwait
     26.0  3,376,565,623,895    290,523      11,622,369.4      10,090,974.0           1,000  71,871,494,813    230,715,852.7  epoll_wait
     25.1  3,262,215,427,364     59,065      55,230,939.3      10,083,527.0           1,000  71,870,982,327    385,612,897.8  poll
      6.5    840,010,023,702         84  10,000,119,329.8  10,000,109,241.0  10,000,080,012  10,000,211,080         26,726.8  sem_clockwait
      5.1    663,374,571,457      1,692     392,065,349.6      29,104,061.0         112,195  71,857,879,891  3,654,653,557.4  sem_wait
      3.1    402,722,853,571      6,982      57,680,156.6       2,913,302.5           1,000   1,000,564,922    180,748,084.0  epoll_pwait
      0.5     63,234,303,218      5,606      11,279,754.4         358,201.0           1,547     341,877,813     44,756,422.4  pthread_cond_wait
      0.2     21,268,321,619         65     327,204,948.0     124,170,211.0           7,394  13,044,245,421  1,607,094,139.2  waitpid
      0.1     13,716,060,172        137     100,117,227.5     100,125,729.0     100,068,289     100,401,780         40,364.7  clock_nanosleep
      0.1      9,409,234,526     11,262         835,485.2           3,990.0           1,000   8,415,860,013     79,332,526.7  read
      0.0      5,424,286,630  2,048,560           2,647.9           1,267.0           1,000      19,237,200         42,942.0  munmap
      0.0      4,479,576,851        313      14,311,747.1         165,598.0           4,424      76,911,631     19,096,251.5  pthread_join
      0.0      3,403,717,291  2,173,798           1,565.8           1,417.0           1,235       2,382,841          2,683.9  mmap64
      0.0        493,790,273     12,679          38,945.5           7,741.0           1,000      18,241,705        271,153.2  pthread_mutex_lock
      0.0        460,956,092     18,887          24,406.0           6,435.0           1,000       3,723,371         91,907.8  ioctl
      0.0        163,707,024        529         309,465.1         247,619.0         110,027       1,001,867        162,914.9  pthread_create
      0.0        149,065,627      3,996          37,303.7          12,558.5           1,000         873,249         81,416.8  recv
      0.0        115,027,579     45,903           2,505.9           2,037.0           1,000       1,713,086          8,516.3  pthread_cond_signal
      0.0        107,524,167      1,641          65,523.6          67,522.0          16,990         184,030          7,475.4  usleep
      0.0         91,824,273      5,169          17,764.4           6,205.0           1,005         612,318         31,437.2  write
      0.0         90,868,684      3,397          26,749.7          15,157.0           2,252         686,723         27,784.7  send
      0.0         67,854,461     27,871           2,434.6           1,786.0           1,000          51,822          2,390.6  stat64
      0.0         60,567,522      8,251           7,340.6           2,166.0           1,282         153,820         13,187.4  open64
      0.0         56,641,197     19,719           2,872.4           2,000.0           1,000          60,276          2,432.4  open
      0.0         34,376,716         53         648,617.3         314,244.0          28,887       4,841,407        861,211.6  pthread_rwlock_wrlock
      0.0         32,386,616     20,427           1,585.5           1,373.0           1,000       2,669,250         18,681.9  lstat64
      0.0         27,619,689         36         767,213.6         770,454.5         584,225         904,264         79,562.7  posix_spawn
      0.0         19,068,609      1,801          10,587.8          10,198.0           1,338          89,375          7,629.9  mmap
      0.0         16,480,332         68         242,357.8           9,171.5           4,866      15,684,989      1,900,658.1  accept4
      0.0         15,398,967      2,385           6,456.6           3,049.0           1,000         927,353         37,196.1  close
      0.0         13,493,264      1,289          10,468.0           3,169.0           1,620         112,002         15,075.8  pthread_cond_broadcast
      0.0         11,964,660        527          22,703.3           3,138.0           1,000       3,606,965        220,889.3  fread
      0.0          9,185,579          4       2,296,394.8           2,138.0           1,553       9,179,750      4,588,903.5  fcntl64
      0.0          7,773,997      3,460           2,246.8           1,473.0           1,003          22,042          1,465.2  epoll_ctl
      0.0          7,718,874      1,577           4,894.7           1,269.0           1,101       4,105,003        103,329.8  accept
      0.0          5,385,273         58          92,849.5          64,765.0          15,170       1,202,883        210,498.1  writev
      0.0          4,344,929        231          18,809.2          11,586.0           2,350          68,553         18,453.9  fopen
      0.0          3,276,964        232          14,124.8           2,381.5           1,000       1,864,855        122,328.5  fwrite
      0.0          2,723,827        677           4,023.4           3,254.0           1,020          38,668          3,393.1  stat
      0.0          2,499,648         26          96,140.3          71,269.5          13,195         272,473         73,694.5  connect
      0.0          2,071,268          1       2,071,268.0       2,071,268.0       2,071,268       2,071,268              0.0  nanosleep
      0.0          1,730,187      1,018           1,699.6           1,408.5           1,001          13,037            738.1  fstat64
      0.0          1,478,347         16          92,396.7          93,500.5          21,093         162,187         51,818.9  pthread_rwlock_rdlock
      0.0          1,206,694        273           4,420.1           3,131.0           1,020          65,765          5,604.5  fclose
      0.0            819,961        210           3,904.6           2,160.0           1,025          63,285          7,531.4  fopen64
      0.0            742,682         56          13,262.2           7,321.5           3,599          35,969          9,708.0  socket
      0.0            640,935         98           6,540.2           3,028.5           1,262          30,406          5,384.2  fputs
      0.0            564,802        101           5,592.1           2,943.0           1,232          23,024          5,040.2  putc
      0.0            541,634         27          20,060.5          15,595.0           4,853          63,661         14,730.9  shutdown
      0.0            410,933        137           2,999.5           3,737.0           1,027           5,877          1,294.9  flock
      0.0            348,995        199           1,753.7           1,575.0           1,007          19,097          1,844.7  sigaction
      0.0            343,450         25          13,738.0          16,306.0           1,190          28,493         11,541.4  close_range
      0.0            338,941          4          84,735.3          84,650.5          10,227         159,413         85,919.6  backtrace
      0.0            281,038         28          10,037.1           2,532.0           1,003         135,880         27,072.4  bind
      0.0            262,562         38           6,909.5           5,848.5           1,523          18,457          4,211.9  pipe2
      0.0            248,176         43           5,771.5           5,489.0           1,155          19,855          4,049.4  recvmsg
      0.0            245,703         21          11,700.1           1,831.0           1,034         210,937         45,653.9  statx
      0.0            182,079         93           1,957.8           1,470.0           1,002           3,915            904.3  futex
      0.0            117,493         16           7,343.3           6,130.5           4,769          13,913          3,111.1  sendmsg
      0.0             99,436         63           1,578.3           1,327.0           1,008           3,123            535.1  fcntl
      0.0             62,516          6          10,419.3          10,366.0           4,024          16,897          6,632.0  lstat
      0.0             52,007         25           2,080.3           1,480.0           1,001           4,890          1,315.6  fstat
      0.0             29,439         12           2,453.3           1,903.0           1,526           5,994          1,351.6  listen
      0.0             28,949          3           9,649.7           9,014.0           8,396          11,539          1,665.1  getc
      0.0             21,901         13           1,684.7           1,783.0           1,106           2,278            331.0  dup2
      0.0             15,317          2           7,658.5           7,658.5           7,468           7,849            269.4  memfd_create
      0.0             13,013          9           1,445.9           1,532.0           1,097           1,871            267.2  signal
      0.0              1,073          1           1,073.0           1,073.0           1,073           1,073              0.0  pthread_mutex_trylock

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/cuda_api_sum.py]...

 ** CUDA API Summary (cuda_api_sum):

 Time (%)  Total Time (ns)  Num Calls    Avg (ns)     Med (ns)    Min (ns)    Max (ns)    StdDev (ns)                      Name
 --------  ---------------  ---------  ------------  -----------  ---------  -----------  ------------  ------------------------------------------
     68.2   50,384,782,577      4,893  10,297,319.1      4,837.0      3,146  226,831,279  15,417,444.0  cudaEventSynchronize
     13.6   10,077,075,217      8,610   1,170,392.0     57,654.5      2,353  229,263,802   8,795,237.2  cudaDeviceSynchronize
      5.9    4,390,875,516      4,534     968,433.1  1,015,304.0     14,132    3,622,034     659,398.3  cudaGraphLaunch_v10000
      5.5    4,078,188,282     37,465     108,853.3      6,318.0      1,588   99,317,959   1,150,500.3  cudaMemcpyAsync
      2.6    1,946,857,032    377,105       5,162.6      2,903.0        828  196,734,163     408,835.8  cudaLaunchKernel
      1.7    1,224,689,411      8,332     146,986.2     77,362.5     35,125    7,921,263     528,349.5  cudaGraphInstantiateWithFlags_v11040
      0.4      309,824,420      9,983      31,035.2      8,138.0      1,416      617,518      77,008.4  cudaStreamSynchronize
      0.4      294,617,860    120,122       2,452.7      2,510.0        776      553,713       4,270.7  cuLaunchKernel
      0.3      191,633,352      1,594     120,221.7    103,557.0      7,829    2,104,467     146,897.3  cudaMalloc
      0.2      171,637,667        498     344,653.9    414,548.5      2,607    3,901,958     334,424.2  cudaFree
      0.1      107,450,759      4,872      22,054.8      4,320.5      1,594    1,208,070     132,701.6  cuLaunchKernelEx
      0.1       82,495,374    250,032         329.9        287.0        171       31,915         237.4  cudaStreamGetCaptureInfo_v2_v11030
      0.1       68,547,433     27,927       2,454.5      1,024.0        335       68,439       3,546.0  cudaEventQuery
      0.1       66,792,227        760      87,884.5      1,138.0        911   29,229,732   1,489,251.6  cuKernelGetFunction
      0.1       57,124,719    132,655         430.6        374.0        212      162,819         524.0  cudaStreamIsCapturing_v10000
      0.1       54,038,157      8,332       6,485.6      4,015.0      2,467      410,861      24,303.2  cudaGraphDestroy_v10000
      0.1       53,151,516         28   1,898,268.4  1,560,116.0    262,229    4,879,427   1,637,221.6  cuLibraryLoadData
      0.1       45,223,637     22,296       2,028.3        837.0        323    4,851,056      45,484.7  cudaEventCreateWithFlags
      0.1       41,167,652     22,113       1,861.7      1,896.0        560    1,158,304       9,048.0  cudaEventRecordWithFlags_v11010
      0.1       37,768,316     10,464       3,609.4      1,107.0        227   17,662,826     175,247.2  cudaThreadExchangeStreamCaptureMode_v10010
      0.0       29,342,828     20,342       1,442.5        941.0        282    1,344,735       9,478.4  cudaEventRecord
      0.0       19,356,879         48     403,268.3    190,405.0     76,416    2,885,961     544,839.8  cuMemSetAccess
      0.0       18,776,502     22,227         844.8        485.0        275    1,907,677      13,996.5  cudaEventDestroy
      0.0       17,023,002     13,980       1,217.7      1,128.0        338       54,423       1,136.3  cudaStreamWaitEvent
      0.0       15,944,266      8,332       1,913.6      1,288.0        935      120,673       5,355.6  cudaStreamEndCapture_v10000
      0.0       15,279,055         36     424,418.2    413,144.0    156,997      842,614     167,297.6  cuModuleLoad
      0.0       13,596,648      8,332       1,631.9      1,534.0        994      463,663       5,091.6  cudaStreamBeginCapture_v10000
      0.0       13,047,124         24     543,630.2     59,674.5     31,760    3,755,164   1,034,390.9  cuMemCreate
      0.0       11,221,713          8   1,402,714.1  1,136,205.0    713,130    2,845,312     757,350.8  cuMemExportToShareableHandle
      0.0        6,056,010         52     116,461.7     24,467.0      5,424    2,264,183     418,596.3  cudaHostAlloc
      0.0        5,802,120      8,332         696.4        615.0        461       10,192         455.6  cudaGraphGetNodes_v10000
      0.0        3,603,592          2   1,801,796.0  1,801,796.0  1,761,533    1,842,059      56,940.5  cudaGetDeviceProperties_v12000
      0.0        2,089,718        328       6,371.1      4,912.0      1,963      106,007      10,799.2  cudaStreamCreateWithFlags
      0.0        1,845,366      4,872         378.8        394.0        113        6,908         269.9  cudaGetFuncBySymbol_v11000
      0.0        1,622,282        322       5,038.1      4,815.0      2,726       15,324       1,537.7  cudaStreamDestroy
      0.0        1,274,976          2     637,488.0    637,488.0    463,503      811,473     246,051.9  cudaMemPoolCreate_v11020
      0.0        1,126,368         14      80,454.9     59,204.5     42,365      229,765      59,502.1  cudaIpcOpenMemHandle
      0.0        1,081,909         32      33,809.7     27,746.0      2,169      110,977      30,495.1  cuMemMap
      0.0          860,794          8     107,599.3    107,401.0     37,227      190,732      51,098.2  cuMemImportFromShareableHandle
      0.0          570,862      2,634         216.7        170.5         75        8,916         312.3  cuGetProcAddress_v2
      0.0          542,689        322       1,685.4      1,543.0        790       11,125         997.7  cudaGraphAddEventRecordNode_v11010
      0.0          407,081          8      50,885.1     52,037.0     33,546       68,504      12,929.5  cudaMemGetInfo
      0.0          397,279        806         492.9        412.0        225        6,557         358.9  cudaStreamUpdateCaptureDependencies_v11030
      0.0          387,921         16      24,245.1     10,317.5      7,680       73,520      25,202.0  cudaMemsetAsync
      0.0          328,186         32      10,255.8      5,306.5      3,323       44,818      10,986.1  cuMemAddressReserve
      0.0          200,896        322         623.9        600.5        287        4,796         318.3  cudaGraphAddDependencies_v10000
      0.0          192,473        322         597.7        565.5        275        5,425         325.8  cudaGraphRetainUserObject_v11030
      0.0          170,323          2      85,161.5     85,161.5     81,632       88,691       4,991.5  cudaMemcpy
      0.0          143,609        322         446.0        411.0        225        1,805         159.3  cudaUserObjectCreate_v11030
      0.0           91,586         34       2,693.7      3,224.0        418        7,567       1,780.3  cuLibraryGetKernel
      0.0           81,753         32       2,554.8        383.0        224       38,063       8,256.1  cuMemGetAllocationGranularity
      0.0           19,010         10       1,901.0      1,306.5        671        3,717       1,121.0  cuInit
      0.0            6,435          2       3,217.5      3,217.5      3,134        3,301         118.1  cudaMemPoolSetAttribute_v11020
      0.0            2,675          4         668.8        576.0        353        1,170         386.8  cudaGetDriverEntryPoint_v11030
      0.0              670          4         167.5        156.0        138          220          37.8  cuModuleGetLoadingMode

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/cuda_gpu_kern_sum.py]...

 ** CUDA GPU Kernel Summary (cuda_gpu_kern_sum):

 Time (%)  Total Time (ns)  Instances   Avg (ns)     Med (ns)    Min (ns)    Max (ns)    StdDev (ns)                                                   Name
 --------  ---------------  ---------  -----------  -----------  ---------  -----------  ------------  ----------------------------------------------------------------------------------------------------
     51.9   62,589,830,968    811,520     77,126.7     33,345.0      8,672      200,707      55,913.1  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      9.0   10,817,981,632     61,320    176,418.5    176,803.0    126,722      228,483       8,546.0  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      8.6   10,414,294,544     39,680    262,457.0    164,770.5     45,761    2,379,362     298,383.9  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      7.8    9,385,940,249    183,960     51,021.6     36,224.0     14,944       95,458      26,039.9  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      4.5    5,385,854,019    523,652     10,285.2      4,448.0      4,096  218,945,664     302,958.6  void vllm::cross_device_reduce_1stage<__half, (int)2>(vllm::RankData *, vllm::RankSignals, vllm::Si…
      2.9    3,507,409,103      2,502  1,401,842.2  1,401,812.0  1,400,371    1,408,212         599.6  std::enable_if<!T7, void>::type internal::gemvx::kernel<int, int, __half, __half, __half, float, (b…
      2.9    3,440,388,334      1,288  2,671,109.0    590,665.0    542,760  446,801,273  14,106,619.3  ncclDevKernel_AllReduce_Sum_f16_RING_LL(ncclDevKernelArgsStorage<(unsigned long)4096>)
      1.9    2,250,706,856    199,200     11,298.7     11,264.0     11,136       13,600         209.2  void flash::flash_fwd_splitkv_combine_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (…
      1.6    1,960,511,758    258,760      7,576.6      6,016.0      5,600       23,425       3,583.1  void flash::flash_fwd_splitkv_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (int)4, (…
      1.4    1,638,772,760     13,584    120,639.9    135,010.0        896      141,218      40,457.2  void at::native::vectorized_elementwise_kernel<(int)4, at::native::FillFunctor<int>, std::array<cha…
      1.0    1,189,515,321        724  1,642,977.0  1,642,920.5  1,638,647    1,657,913       1,788.4  void cutlass::Kernel2<cutlass_80_tensorop_f16_s16816gemm_relu_f16_128x256_32x3_tn_align8>(T1::Param…
      0.8    1,001,399,362    271,010      3,695.1      3,360.0      3,232      211,171       4,865.5  triton_red_fused__to_copy_add_marlin_gemm_mean_mul_pow_rsqrt_2
      0.8      948,982,402    275,984      3,438.5      1,152.0      1,056      285,124      21,212.3  triton_poi_fused_marlin_gemm_mul_silu_slice_1
      0.7      860,405,306     13,440     64,018.3     28,672.0     10,368      221,635      59,345.9  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      0.7      850,008,978    274,444      3,097.2      2,944.0      2,688       83,233       1,993.6  triton_red_fused__to_copy_add_marlin_gemm_mean_mul_pow_rsqrt_0
      0.5      630,464,606     11,520     54,727.8     42,177.0     25,440      100,610      26,556.9  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      0.5      570,704,392    282,440      2,020.6      1,888.0      1,728       10,561         551.1  void vllm::reshape_and_cache_flash_kernel<unsigned short, unsigned short, (vllm::Fp8KVCacheDataType…
      0.5      554,908,241      3,360    165,151.3    154,626.0    137,442      194,115      18,669.8  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      0.4      437,568,961    271,858      1,609.5      1,408.0      1,312       53,185       2,052.9  triton_poi_fused_3
      0.3      398,888,893     10,080     39,572.3     27,873.0     19,680       93,537      21,059.8  void marlin::Marlin<(long)1125899906910725, (long)1125899906843648, (long)1125899906910725, (long)1…
      0.2      265,993,506    131,556      2,021.9      1,920.0      1,696      146,690         891.3  void at::native::elementwise_kernel<(int)128, (int)4, void at::native::gpu_kernel_impl_nocast<at::n…
      0.2      237,325,373      3,258     72,843.9     26,432.0     25,536    2,830,537     121,519.0  ncclDevKernel_AllGather_RING_LL(ncclDevKernelArgsStorage<(unsigned long)4096>)
      0.2      216,538,347     57,960      3,736.0      3,712.0      3,552        5,920          64.6  void flash::flash_fwd_splitkv_combine_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (…
      0.1      176,848,506        640    276,325.8    205,666.5     86,721      608,777     207,838.7  void marlin::awq_marlin_repack_kernel<(int)256, (int)4, (bool)0>(const unsigned int *, unsigned int…
      0.1      176,804,311      3,262     54,201.2     44,961.0     43,809      669,609      29,736.1  void at::native::<unnamed>::cunn_SoftMaxForward<(int)4, float, float, float, at::native::<unnamed>:…
      0.1      102,536,388      5,600     18,310.1     18,560.0      5,184       35,904       9,286.8  void flash::flash_fwd_splitkv_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (int)4, (…
      0.1       76,362,298      3,262     23,409.7     23,072.0     22,432      165,890       6,430.0  void at::native::reduce_kernel<(int)512, (int)1, at::native::ReduceOp<float, at::native::ArgMaxOps<…
      0.1       60,850,346        160    380,314.7    382,229.0    293,636      386,309       9,528.0  void at::native::vectorized_elementwise_kernel<(int)4, at::native::FillFunctor<signed char>, std::a…
      0.0       58,672,279      1,600     36,670.2      3,888.5      1,472      162,338      60,467.4  void at::native::elementwise_kernel<(int)128, (int)2, void at::native::gpu_kernel_impl_nocast<at::n…
      0.0       47,143,517      1,914     24,630.9     32,609.0      2,528       53,537      14,942.2  triton_
      0.0       43,883,955      4,282     10,248.5      1,696.0      1,376       47,937      17,112.6  triton_poi_fused_add_all_reduce_bitwise_and_bitwise_not_bitwise_or_embedding_ge_lt_masked_fill_mul_…
      0.0       41,743,469         28  1,490,838.2  1,492,901.0  1,478,519    1,500,437       6,790.0  void cutlass::Kernel2<cutlass_80_wmma_tensorop_f16_s161616gemm_f16_16x16_128x2_tn_align8>(T1::Param…
      0.0       38,241,618      4,282      8,930.8      1,888.0      1,440       53,025      14,227.5  triton_poi_fused_2
      0.0       24,996,190     16,964      1,473.5      1,344.0        864        8,096         658.3  void at::native::vectorized_elementwise_kernel<(int)4, at::native::FillFunctor<c10::Half>, std::arr…
      0.0       23,677,383         16  1,479,836.4  1,457,619.5  1,369,045    1,630,647      94,439.2  void at_cuda_detail::cub::DeviceSegmentedRadixSortKernel<at_cuda_detail::cub::detail::radix::policy…
      0.0       19,778,401      1,600     12,361.5      7,264.0      5,792       25,728       7,976.5  void flash::flash_fwd_splitkv_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (int)4, (…
      0.0       16,438,913     17,407        944.4        928.0        896        2,624          33.0  void at::native::vectorized_elementwise_kernel<(int)2, at::native::FillFunctor<long>, std::array<ch…
      0.0       15,951,572      3,432      4,647.9      4,384.0      3,264      118,305       3,949.4  triton_red_fused__to_copy_add_mean_mul_pow_rsqrt_2
      0.0       14,679,146      3,258      4,505.6      2,400.0      2,112      583,881      20,271.3  void at::native::vectorized_elementwise_kernel<(int)4, at::native::BinaryFunctor<float, float, floa…
      0.0       13,725,055      3,150      4,357.2      3,520.0      2,656       14,400       2,436.6  void at::native::index_elementwise_kernel<(int)128, (int)4, void at::native::gpu_index_kernel<void …
      0.0       12,961,460          8  1,620,182.5  1,619,703.0  1,485,013    1,753,335     139,201.7  void at_cuda_detail::cub::DeviceSegmentedRadixSortKernel<at_cuda_detail::cub::detail::radix::policy…
      0.0       12,136,683      3,258      3,725.2      2,304.0      2,176      300,388      10,519.4  void at::native::unrolled_elementwise_kernel<at::native::direct_copy_kernel_cuda(at::TensorIterator…
      0.0       11,336,746      3,258      3,479.7      1,888.0      1,824      302,244      10,578.9  void at::native::elementwise_kernel<(int)128, (int)2, void at::native::gpu_kernel_impl_nocast<at::n…
      0.0       10,808,495      3,258      3,317.5      2,336.0      2,272      212,162       7,333.2  void at::native::<unnamed>::distribution_elementwise_grid_stride_kernel<float, (int)4, void at::nat…
      0.0       10,544,895      3,434      3,070.7      2,944.0      2,528       21,921       1,054.1  triton_red_fused__to_copy_add_marlin_gemm_mean_mul_pow_rsqrt_1
      0.0       10,193,735      1,944      5,243.7      4,160.0      2,240       17,184       3,019.4  void at::native::index_elementwise_kernel<(int)128, (int)4, void at::native::gpu_index_kernel<void …
      0.0        8,907,385      5,440      1,637.4      1,664.0      1,312        1,888         102.3  void at::native::elementwise_kernel<(int)128, (int)2, void at::native::gpu_kernel_impl_nocast<at::n…
      0.0        7,537,634         82     91,922.4     23,328.5      1,440    2,352,289     360,675.6  void at::native::<unnamed>::distribution_elementwise_grid_stride_kernel<float, (int)4, void at::nat…
      0.0        7,368,358          4  1,842,089.5  1,842,570.0  1,826,232    1,856,986      16,691.8  void cutlass::Kernel2<cutlass_80_tensorop_f16_s16816gemm_relu_f16_256x128_32x3_tn_align8>(T1::Param…
      0.0        6,206,915      3,254      1,907.5      1,888.0      1,664        2,176          45.9  void at::native::unrolled_elementwise_kernel<at::native::direct_copy_kernel_cuda(at::TensorIterator…
      0.0        4,485,492      3,258      1,376.8      1,376.0      1,280        1,696          22.0  void at::native::unrolled_elementwise_kernel<at::native::direct_copy_kernel_cuda(at::TensorIterator…
      0.0        3,829,595      1,120      3,419.3      3,392.0      3,360        5,312         130.7  void flash::flash_fwd_splitkv_combine_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (…
      0.0        3,329,230         28    118,901.1      1,888.0      1,728      828,139     291,899.6  void at::native::_scatter_gather_elementwise_kernel<(int)128, (int)8, void at::native::_cuda_scatte…
      0.0        3,268,110      3,256      1,003.7        992.0        960        1,088          15.8  void at::native::unrolled_elementwise_kernel<at::native::CUDAFunctorOnSelf_add<int>, std::array<cha…
      0.0        3,158,410          8    394,801.3    396,709.0    374,756      414,214      17,681.2  void at::native::vectorized_elementwise_kernel<(int)4, at::native::<unnamed>::masked_fill_kernel(at…
      0.0        2,290,558      1,092      2,097.6      1,632.0      1,280        7,296       1,098.7  void at::native::vectorized_gather_kernel<(int)16, long>(char *, char *, T2 *, int, long, long, lon…
      0.0        2,285,246        320      7,141.4      7,136.0      7,072        9,025         155.4  void flash::flash_fwd_splitkv_combine_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (…
      0.0        2,011,963          8    251,495.4    248,915.0    208,483      297,988      43,378.6  void at::native::elementwise_kernel<(int)128, (int)4, void at::native::gpu_kernel_impl_nocast<at::n…
      0.0        1,414,644          4    353,661.0    353,845.5    351,812      355,141       1,383.2  at::native::<unnamed>::fill_reverse_indices_kernel(long *, int, at::cuda::detail::IntDivider<unsign…
      0.0        1,394,515          4    348,628.8    347,860.5    345,605      353,189       3,305.6  void at::native::tensor_kernel_scan_innermost_dim<float, std::plus<float>>(T1 *, const T1 *, unsign…
      0.0          793,778        160      4,961.1      4,960.0      4,896        5,568          83.9  void flash::flash_fwd_splitkv_combine_kernel<Flash_fwd_kernel_traits<(int)128, (int)64, (int)128, (…
      0.0          616,394        664        928.3        928.0        896        1,600          66.6  void at::native::vectorized_elementwise_kernel<(int)4, at::native::FillFunctor<float>, std::array<c…
      0.0           23,424         26        900.9        896.0        896          928          11.8  void at::native::unrolled_elementwise_kernel<at::native::FillFunctor<long>, std::array<char *, (uns…
      0.0           22,465          4      5,616.3      5,616.5      5,568        5,664          41.2  void at::native::<unnamed>::distribution_elementwise_grid_stride_kernel<float, (int)4, void at::nat…
      0.0           15,233          8      1,904.1      1,936.0      1,792        1,952          64.0  void at::native::vectorized_elementwise_kernel<(int)4, void at::native::compare_scalar_kernel<float…
      0.0           10,848          4      2,712.0      2,688.0      2,592        2,880         131.6  void at::native::_scatter_gather_elementwise_kernel<(int)128, (int)8, void at::native::_cuda_scatte…
      0.0            7,841          4      1,960.3      1,968.5      1,696        2,208         254.1  void at::native::vectorized_elementwise_kernel<(int)4, at::native::CUDAFunctorOnOther_add<float>, s…
      0.0            7,584          4      1,896.0      1,920.0      1,760        1,984         108.9  void at::native::vectorized_elementwise_kernel<(int)2, at::native::CUDAFunctorOnOther_add<long>, st…
      0.0            7,393          4      1,848.3      1,824.0      1,568        2,177         309.9  void at::native::elementwise_kernel<(int)128, (int)4, void at::native::gpu_kernel_impl_nocast<at::n…
      0.0            6,817          4      1,704.3      1,712.5      1,632        1,760          54.8  void at::native::elementwise_kernel<(int)128, (int)2, void at::native::gpu_kernel_impl_nocast<at::n…
      0.0            6,784          4      1,696.0      1,696.0      1,664        1,728          26.1  void at::native::vectorized_elementwise_kernel<(int)2, at::native::<unnamed>::where_kernel_impl(at:…

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/cuda_gpu_mem_time_sum.py]...

 ** CUDA GPU MemOps Summary (by Time) (cuda_gpu_mem_time_sum):

 Time (%)  Total Time (ns)   Count   Avg (ns)   Med (ns)  Min (ns)   Max (ns)   StdDev (ns)            Operation
 --------  ---------------  -------  ---------  --------  --------  ----------  -----------  ------------------------------
     91.1    3,511,442,378   28,996  121,100.9     768.0       352  99,183,773  1,300,513.4  [CUDA memcpy Host-to-Device]
      5.2      200,119,365  202,956      986.0     960.0       928     416,613      2,557.8  [CUDA memcpy Device-to-Device]
      3.7      143,886,430    4,537   31,714.0   1,312.0       480     583,656     99,605.2  [CUDA memcpy Device-to-Host]
      0.0           11,616       16      726.0     576.0       544       1,600        338.8  [CUDA memset]

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/cuda_gpu_mem_size_sum.py]...

 ** CUDA GPU MemOps Summary (by Size) (cuda_gpu_mem_size_sum):

 Total (MB)   Count   Avg (MB)  Med (MB)  Min (MB)  Max (MB)   StdDev (MB)            Operation
 ----------  -------  --------  --------  --------  ---------  -----------  ------------------------------
 42,222.461   28,996     1.456     0.001     0.000  1,050.673       14.521  [CUDA memcpy Host-to-Device]
  4,034.464  202,956     0.020     0.008     0.000    131.334        0.841  [CUDA memcpy Device-to-Device]
  2,406.551    4,537     0.530     0.000     0.000      7.340        1.494  [CUDA memcpy Device-to-Host]
      0.052       16     0.003     0.000     0.000      0.024        0.008  [CUDA memset]

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/openmp_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain OpenMP event data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/opengl_khr_range_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain KHR Extension (KHR_DEBUG) data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/opengl_khr_gpu_range_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain GPU KHR Extension (KHR_DEBUG) data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/vulkan_marker_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain Vulkan Debug Extension (Vulkan Debug Util) data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/vulkan_gpu_marker_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain GPU Vulkan Debug Extension (GPU Vulkan Debug markers) data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/dx11_pix_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain DX11 CPU debug markers.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/dx12_gpu_marker_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain DX12 GPU debug markers.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/dx12_pix_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain DX12 CPU debug markers.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/wddm_queue_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain WDDM context data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/um_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain CUDA Unified Memory CPU page faults data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/um_total_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain CUDA Unified Memory CPU page faults data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/um_cpu_page_faults_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain CUDA Unified Memory CPU page faults data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/openacc_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain OpenACC event data.

Processing [/mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite] with [/opt/nvidia/nsight-systems/2025.5.2/host-linux-x64/reports/syscall_sum.py]...
SKIPPED: /mnt/elita/soundwave/profiles/vllm_tp2_2026-03-02_151959.sqlite does not contain syscall data.
```

---

#### Analisys

1) Interpreting your nsys stats numbers  

- A key thing to remember: most of these “Total Time” values are summed over many threads/streams, so they can be much larger than wall-clock (and the % is only “within that report section”, not your whole app).
  
- NVTX Range Summary (your NCCL markers)
  
    - This section is time inside NVTX “ranges” (here, NCCL labels).

    - NCCL:ncclAllGather — 44.6% (3262 calls)

    - NCCL:ncclAllReduce — 33.3% (1610 calls)
    
        - In your TP=2 run, a big chunk of the NVTX-labeled work is GPU-to-GPU communication (expected for tensor parallel).
        - The Max values (e.g., 41 ms for AllGather) mean some collectives occasionally stall long—often due to synchronization, queueing, or overlap issues.
  
        - ncclCommInitRankConfig — 21.9% (2 calls, ~41.8 ms each)
          That’s just communicator init, not runtime cost (but it shows up because it’s NVTX-marked).
  

- OS Runtime Summary (pthread/epoll/poll)

Top lines:

pthread_cond_timedwait / epoll_wait / poll dominate
This mostly means CPU threads are waiting (on queues, network events, worker coordination). In a server, that’s normal—especially with many threads and “bursty” requests.

Also notice:

sem_clockwait has ~10s median (84 calls)
That’s a timed wait (not necessarily bad), but it can hint at periodic sleeps/timeouts inside worker/scheduler logic.

CUDA API Summary (CPU-side CUDA calls)

Biggest flags:

cudaEventSynchronize (68.2%, ~50.4s) and cudaDeviceSynchronize (13.6%, ~10.1s)
Your CPU is spending lots of time blocking/waiting for GPU work to finish. This often reduces overlap (CPU can’t enqueue more work while waiting). In LLM serving, these sync points can come from scheduling boundaries, graph capture boundaries, stream coordination, or comm barriers.

Good sign:

cudaGraphLaunch (~4.39s, 4534 calls)
CUDA graphs are actually being launched (so graphs are active).

Potential overhead:

cudaGraphInstantiateWithFlags (8332 calls)
Graph instantiation is not free. If this is happening frequently during steady-state, it can be a perf tax (ideally you instantiate less often and reuse).

CUDA GPU Kernel Summary (actual GPU work)

marlin::Marlin... kernels dominate (~52% + more in other Marlin rows)
That matches your model: AWQ + Marlin GEMMs are the main compute.

You also see:

flash::flash_fwd... → FlashAttention is in play

ncclDevKernel_AllReduce... → some GPU-time spent in NCCL kernels (real comm cost)

CUDA MemOps (copies)

Host→Device total ~42,222 MB across 28,996 copies

Median size is 0.001 MB (~1 KB) but max is ~1050 MB
This usually means: tons of tiny H2D transfers + a few very large ones. Tiny copies can be overheady (launch/driver overhead dominates). In the GUI timeline, look for frequent small memcpy H2D bursts.
