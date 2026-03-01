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




