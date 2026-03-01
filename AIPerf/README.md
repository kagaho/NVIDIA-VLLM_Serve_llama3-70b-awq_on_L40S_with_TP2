### AIPERF  
  
- Reference: https://github.com/ai-dynamo/aiperf

```
$ podman run -d --name aiperf-runner --net=host \
  --security-opt=label=disable \
  -v /mnt/elita/soundwave:/mnt/elita/soundwave \
  python:3.12-slim \
  bash -lc "python -m pip install -U pip >/dev/null 2>&1 && pip install --only-binary=:all: aiperf >/dev/null 2>&1 && sleep infinity"
```  
7f721931cfe5d621581c5c25875cea556f24fba1f63b604aff4d2f183b9e08e8

$ podman ps
```
CONTAINER ID  IMAGE                               COMMAND               CREATED            STATUS            PORTS       NAMES
962ccd197586  docker.io/vllm/vllm-openai:latest   -lc vllm serve /m...  About an hour ago  Up About an hour              priceless_sammet
7f721931cfe5  docker.io/library/python:3.12-slim  bash -lc python -...  8 seconds ago      Up 8 seconds                  aiperf-runner
```

$ podman exec -it aiperf-runner bash
```
root@elita:/# aiperf --help
Usage: aiperf COMMAND

NVIDIA AIPerf

╭─ Commands ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ analyze-trace  Analyze mooncake trace for prefix statistics                                                                                                                                                     │
│ plot           Generate visualizations from AIPerf profiling data.                                                                                                                                              │
│ profile        Run the Profile subcommand.                                                                                                                                                                      │
│ --help (-h)    Display this message and exit.                                                                                                                                                                   │
│ --version      Display application version.                                                                                                                                                                     │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```
```
$ aiperf profile \
  --model /mnt/elita/soundwave/models/llama3-70b-awq \
  --url http://localhost:8002 \
  --endpoint-type chat \
  --concurrency 10 \
  --request-count 100 \
  --streaming
```

``` bash

                                               NVIDIA AIPerf | LLM Metrics
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━━┳━━━━━━━━━━━┳━━━━━━━━━━━┳━━━━━━━━━━━┳━━━━━━━━━━┓
┃                               Metric ┃       avg ┃      min ┃       max ┃       p99 ┃       p90 ┃       p50 ┃      std ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━━╇━━━━━━━━━━━╇━━━━━━━━━━━╇━━━━━━━━━━━╇━━━━━━━━━━┩
│             Time to First Token (ms) │    561.30 │   351.43 │  2,973.53 │  2,973.45 │    674.81 │    368.61 │   561.54 │
│            Time to Second Token (ms) │     93.96 │    28.27 │  1,047.14 │  1,047.08 │    310.31 │     33.71 │   197.70 │
│      Time to First Output Token (ms) │    561.30 │   351.43 │  2,973.53 │  2,973.45 │    674.81 │    368.61 │   561.54 │
│                 Request Latency (ms) │ 13,889.91 │ 8,535.88 │ 23,284.99 │ 22,103.86 │ 17,204.72 │ 13,510.24 │ 2,676.80 │
│             Inter Token Latency (ms) │     40.45 │    31.53 │     44.93 │     43.51 │     42.50 │     40.85 │     2.23 │
│     Output Token Throughput Per User │     24.81 │    22.26 │     31.71 │     30.76 │     26.27 │     24.48 │     1.54 │
│                    (tokens/sec/user) │           │          │           │           │           │           │          │
│      Output Sequence Length (tokens) │    330.31 │   197.00 │    558.00 │    513.45 │    398.50 │    322.50 │    60.20 │
│       Input Sequence Length (tokens) │    550.00 │   550.00 │    550.00 │    550.00 │    550.00 │    550.00 │     0.00 │
│ Output Token Throughput (tokens/sec) │    228.50 │      N/A │       N/A │       N/A │       N/A │       N/A │      N/A │
│    Request Throughput (requests/sec) │      0.69 │      N/A │       N/A │       N/A │       N/A │       N/A │      N/A │
│             Request Count (requests) │    100.00 │      N/A │       N/A │       N/A │       N/A │       N/A │      N/A │
└──────────────────────────────────────┴───────────┴──────────┴───────────┴───────────┴───────────┴───────────┴──────────┘

CLI Command: aiperf profile --model '/mnt/elita/soundwave/models/llama3-70b-awq' --url 'http://localhost:8002' --endpoint-type 'chat' --concurrency 10 --request-count 100 --streaming
Benchmark Duration: 144.56 sec
CSV Export: /artifacts/_mnt_elita_soundwave_models_llama3-70b-awq-openai-chat-concurrency10/profile_export_aiperf.csv
JSON Export: /artifacts/_mnt_elita_soundwave_models_llama3-70b-awq-openai-chat-concurrency10/profile_export_aiperf.json
Server Metrics CSV Export: /artifacts/_mnt_elita_soundwave_models_llama3-70b-awq-openai-chat-concurrency10/server_metrics_export.csv
Server Metrics JSON Export: /artifacts/_mnt_elita_soundwave_models_llama3-70b-awq-openai-chat-concurrency10/server_metrics_export.json
Log File: /artifacts/_mnt_elita_soundwave_models_llama3-70b-awq-openai-chat-concurrency10/logs/aiperf.log
```


