### Installing AIPerf (Replacement of GenAI-Perf) on container:

- Persistent container:
    
```bash
podman run -d --name aiperf-runner --net=host \
  --security-opt=label=disable \
  -v /mnt/elita/soundwave:/mnt/elita/soundwave \
  python:3.12-slim \
  bash -lc "python -m pip install -U pip >/dev/null 2>&1 && pip install --only-binary=:all: aiperf >/dev/null 2>&1 && sleep infinity"
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
root@elita:/#
```





- 1 time running container + installation:
```bash
podman run --rm -it --net=host \
  --security-opt=label=disable \
  --entrypoint bash \
  python:3.12-slim -lc \
  "python -m pip install -U pip && pip install --only-binary=:all: aiperf && aiperf --help"
```

```  
Resolved "python" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/python:3.12-slim...
Getting image source signatures
Copying blob 2d517a4d89f5 done   |
Copying blob a546a81ba946 done   |
Copying blob 206356c42440 done   |
Copying blob 90b93dccd534 done   |
Copying config 5265ced2e3 done   |
Writing manifest to image destination
Requirement already satisfied: pip in /usr/local/lib/python3.12/site-packages (25.0.1)
Collecting pip
  Downloading pip-26.0.1-py3-none-any.whl.metadata (4.7 kB)
Downloading pip-26.0.1-py3-none-any.whl (1.8 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.8/1.8 MB 68.6 MB/s eta 0:00:00
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 25.0.1
    Uninstalling pip-25.0.1:
      Successfully uninstalled pip-25.0.1
Successfully installed pip-26.0.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager, possibly rendering your system unusable. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv. Use the --root-user-action option if you know what you are doing and want to suppress this warning.
Collecting aiperf
  Downloading aiperf-0.5.0-py3-none-any.whl.metadata (13 kB)
Collecting aiofiles~=24.1.0 (from aiperf)
  Downloading aiofiles-24.1.0-py3-none-any.whl.metadata (10 kB)
Collecting aiohttp~=3.13.3 (from aiperf)
  Downloading aiohttp-3.13.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (8.1 kB)
Collecting billiard>=4.2.0 (from aiperf)
  Downloading billiard-4.2.4-py3-none-any.whl.metadata (4.8 kB)
Collecting cyclopts<5,>=4 (from aiperf)
  Downloading cyclopts-4.6.0-py3-none-any.whl.metadata (12 kB)
Collecting dash-bootstrap-components~=2.0.0 (from aiperf)
  Downloading dash_bootstrap_components-2.0.4-py3-none-any.whl.metadata (18 kB)
Collecting dash~=3.1.0 (from aiperf)
  Downloading dash-3.1.1-py3-none-any.whl.metadata (10 kB)
Collecting ffmpeg-python~=0.2.0 (from aiperf)
  Downloading ffmpeg_python-0.2.0-py3-none-any.whl.metadata (1.7 kB)
Collecting jinja2~=3.1.5 (from aiperf)
  Downloading jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
Collecting jmespath~=1.0.1 (from aiperf)
  Downloading jmespath-1.0.1-py3-none-any.whl.metadata (7.6 kB)
Collecting kaleido~=1.2.0 (from aiperf)
  Downloading kaleido-1.2.0-py3-none-any.whl.metadata (5.6 kB)
Collecting matplotlib>=3.10.0 (from aiperf)
  Downloading matplotlib-3.10.8-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (52 kB)
Collecting msgspec<1.0.0,>=0.19.0 (from aiperf)
  Downloading msgspec-0.20.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (5.5 kB)
Collecting numpy~=1.26.4 (from aiperf)
  Downloading numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (61 kB)
Collecting orjson~=3.10.18 (from aiperf)
  Downloading orjson-3.10.18-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (41 kB)
Collecting pandas~=2.3.3 (from aiperf)
  Downloading pandas-2.3.3-cp312-cp312-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl.metadata (91 kB)
Collecting pillow~=11.1.0 (from aiperf)
  Downloading pillow-11.1.0-cp312-cp312-manylinux_2_28_x86_64.whl.metadata (9.1 kB)
Collecting plotly~=6.4.0 (from aiperf)
  Downloading plotly-6.4.0-py3-none-any.whl.metadata (8.5 kB)
Collecting prometheus-client~=0.23.1 (from aiperf)
  Downloading prometheus_client-0.23.1-py3-none-any.whl.metadata (1.9 kB)
Collecting psutil~=7.0.0 (from aiperf)
  Downloading psutil-7.0.0-cp36-abi3-manylinux_2_12_x86_64.manylinux2010_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (22 kB)
Collecting pyarrow>=18.0.0 (from aiperf)
  Downloading pyarrow-23.0.1-cp312-cp312-manylinux_2_28_x86_64.whl.metadata (3.1 kB)
Collecting pydantic-settings<3.0.0,>=2.10.0 (from aiperf)
  Downloading pydantic_settings-2.13.1-py3-none-any.whl.metadata (3.4 kB)
Collecting pydantic<3.0.0,>=2.10.0 (from aiperf)
  Downloading pydantic-2.12.5-py3-none-any.whl.metadata (90 kB)
Collecting pyzmq~=26.4.0 (from aiperf)
  Downloading pyzmq-26.4.0-cp312-cp312-manylinux_2_28_x86_64.whl.metadata (6.0 kB)
Collecting rich~=14.1.0 (from aiperf)
  Downloading rich-14.1.0-py3-none-any.whl.metadata (18 kB)
Collecting ruamel-yaml~=0.18.12 (from aiperf)
  Downloading ruamel_yaml-0.18.17-py3-none-any.whl.metadata (27 kB)
Collecting seaborn~=0.13.2 (from aiperf)
  Downloading seaborn-0.13.2-py3-none-any.whl.metadata (5.4 kB)
Collecting setproctitle~=1.3.6 (from aiperf)
  Downloading setproctitle-1.3.7-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl.metadata (10 kB)
Collecting soundfile~=0.13.1 (from aiperf)
  Downloading soundfile-0.13.1-py2.py3-none-manylinux_2_28_x86_64.whl.metadata (16 kB)
Collecting textual~=5.3.0 (from aiperf)
  Downloading textual-5.3.0-py3-none-any.whl.metadata (9.1 kB)
Collecting tqdm>=4.67.1 (from aiperf)
  Downloading tqdm-4.67.3-py3-none-any.whl.metadata (57 kB)
Collecting transformers>=4.56.0 (from aiperf)
  Downloading transformers-5.2.0-py3-none-any.whl.metadata (32 kB)
Collecting uvloop~=0.21.0 (from aiperf)
  Downloading uvloop-0.21.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (4.9 kB)
Collecting aiohappyeyeballs>=2.5.0 (from aiohttp~=3.13.3->aiperf)
  Downloading aiohappyeyeballs-2.6.1-py3-none-any.whl.metadata (5.9 kB)
Collecting aiosignal>=1.4.0 (from aiohttp~=3.13.3->aiperf)
  Downloading aiosignal-1.4.0-py3-none-any.whl.metadata (3.7 kB)
Collecting attrs>=17.3.0 (from aiohttp~=3.13.3->aiperf)
  Downloading attrs-25.4.0-py3-none-any.whl.metadata (10 kB)
Collecting frozenlist>=1.1.1 (from aiohttp~=3.13.3->aiperf)
  Downloading frozenlist-1.8.0-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl.metadata (20 kB)
Collecting multidict<7.0,>=4.5 (from aiohttp~=3.13.3->aiperf)
  Downloading multidict-6.7.1-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (5.3 kB)
Collecting propcache>=0.2.0 (from aiohttp~=3.13.3->aiperf)
  Downloading propcache-0.4.1-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (13 kB)
Collecting yarl<2.0,>=1.17.0 (from aiohttp~=3.13.3->aiperf)
  Downloading yarl-1.22.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (75 kB)
Collecting docstring-parser<4.0,>=0.15 (from cyclopts<5,>=4->aiperf)
  Downloading docstring_parser-0.17.0-py3-none-any.whl.metadata (3.5 kB)
Collecting rich-rst<2.0.0,>=1.3.1 (from cyclopts<5,>=4->aiperf)
  Downloading rich_rst-1.3.2-py3-none-any.whl.metadata (6.1 kB)
Collecting Flask<3.2,>=1.0.4 (from dash~=3.1.0->aiperf)
  Downloading flask-3.1.3-py3-none-any.whl.metadata (3.2 kB)
Collecting Werkzeug<3.2 (from dash~=3.1.0->aiperf)
  Downloading werkzeug-3.1.6-py3-none-any.whl.metadata (4.0 kB)
Collecting importlib-metadata (from dash~=3.1.0->aiperf)
  Downloading importlib_metadata-8.7.1-py3-none-any.whl.metadata (4.7 kB)
Collecting typing-extensions>=4.1.1 (from dash~=3.1.0->aiperf)
  Downloading typing_extensions-4.15.0-py3-none-any.whl.metadata (3.3 kB)
Collecting requests (from dash~=3.1.0->aiperf)
  Downloading requests-2.32.5-py3-none-any.whl.metadata (4.9 kB)
Collecting retrying (from dash~=3.1.0->aiperf)
  Downloading retrying-1.4.2-py3-none-any.whl.metadata (5.5 kB)
Collecting nest-asyncio (from dash~=3.1.0->aiperf)
  Downloading nest_asyncio-1.6.0-py3-none-any.whl.metadata (2.8 kB)
Collecting setuptools (from dash~=3.1.0->aiperf)
  Downloading setuptools-82.0.0-py3-none-any.whl.metadata (6.6 kB)
Collecting future (from ffmpeg-python~=0.2.0->aiperf)
  Downloading future-1.0.0-py3-none-any.whl.metadata (4.0 kB)
Collecting blinker>=1.9.0 (from Flask<3.2,>=1.0.4->dash~=3.1.0->aiperf)
  Downloading blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting click>=8.1.3 (from Flask<3.2,>=1.0.4->dash~=3.1.0->aiperf)
  Downloading click-8.3.1-py3-none-any.whl.metadata (2.6 kB)
Collecting itsdangerous>=2.2.0 (from Flask<3.2,>=1.0.4->dash~=3.1.0->aiperf)
  Downloading itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting markupsafe>=2.1.1 (from Flask<3.2,>=1.0.4->dash~=3.1.0->aiperf)
  Downloading markupsafe-3.0.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.7 kB)
Collecting choreographer>=1.1.1 (from kaleido~=1.2.0->aiperf)
  Downloading choreographer-1.2.1-py3-none-any.whl.metadata (6.8 kB)
Collecting logistro>=1.0.8 (from kaleido~=1.2.0->aiperf)
  Downloading logistro-2.0.1-py3-none-any.whl.metadata (3.9 kB)
Collecting packaging (from kaleido~=1.2.0->aiperf)
  Downloading packaging-26.0-py3-none-any.whl.metadata (3.3 kB)
Collecting pytest-timeout>=2.4.0 (from kaleido~=1.2.0->aiperf)
  Downloading pytest_timeout-2.4.0-py3-none-any.whl.metadata (20 kB)
Collecting python-dateutil>=2.8.2 (from pandas~=2.3.3->aiperf)
  Downloading python_dateutil-2.9.0.post0-py2.py3-none-any.whl.metadata (8.4 kB)
Collecting pytz>=2020.1 (from pandas~=2.3.3->aiperf)
  Downloading pytz-2025.2-py2.py3-none-any.whl.metadata (22 kB)
Collecting tzdata>=2022.7 (from pandas~=2.3.3->aiperf)
  Downloading tzdata-2025.3-py2.py3-none-any.whl.metadata (1.4 kB)
Collecting narwhals>=1.15.1 (from plotly~=6.4.0->aiperf)
  Downloading narwhals-2.17.0-py3-none-any.whl.metadata (14 kB)
Collecting annotated-types>=0.6.0 (from pydantic<3.0.0,>=2.10.0->aiperf)
  Downloading annotated_types-0.7.0-py3-none-any.whl.metadata (15 kB)
Collecting pydantic-core==2.41.5 (from pydantic<3.0.0,>=2.10.0->aiperf)
  Downloading pydantic_core-2.41.5-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (7.3 kB)
Collecting typing-inspection>=0.4.2 (from pydantic<3.0.0,>=2.10.0->aiperf)
  Downloading typing_inspection-0.4.2-py3-none-any.whl.metadata (2.6 kB)
Collecting python-dotenv>=0.21.0 (from pydantic-settings<3.0.0,>=2.10.0->aiperf)
  Downloading python_dotenv-1.2.1-py3-none-any.whl.metadata (25 kB)
Collecting markdown-it-py>=2.2.0 (from rich~=14.1.0->aiperf)
  Downloading markdown_it_py-4.0.0-py3-none-any.whl.metadata (7.3 kB)
Collecting pygments<3.0.0,>=2.13.0 (from rich~=14.1.0->aiperf)
  Downloading pygments-2.19.2-py3-none-any.whl.metadata (2.5 kB)
Collecting docutils (from rich-rst<2.0.0,>=1.3.1->cyclopts<5,>=4->aiperf)
  Downloading docutils-0.22.4-py3-none-any.whl.metadata (15 kB)
Collecting ruamel.yaml.clib>=0.2.15 (from ruamel-yaml~=0.18.12->aiperf)
  Downloading ruamel_yaml_clib-0.2.15-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (3.5 kB)
Collecting cffi>=1.0 (from soundfile~=0.13.1->aiperf)
  Downloading cffi-2.0.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (2.6 kB)
Collecting platformdirs<5,>=3.6.0 (from textual~=5.3.0->aiperf)
  Downloading platformdirs-4.9.2-py3-none-any.whl.metadata (4.7 kB)
Collecting idna>=2.0 (from yarl<2.0,>=1.17.0->aiohttp~=3.13.3->aiperf)
  Downloading idna-3.11-py3-none-any.whl.metadata (8.4 kB)
Collecting pycparser (from cffi>=1.0->soundfile~=0.13.1->aiperf)
  Downloading pycparser-3.0-py3-none-any.whl.metadata (8.2 kB)
Collecting simplejson>=3.19.3 (from choreographer>=1.1.1->kaleido~=1.2.0->aiperf)
  Downloading simplejson-3.20.2-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.2 kB)
Collecting mdurl~=0.1 (from markdown-it-py>=2.2.0->rich~=14.1.0->aiperf)
  Downloading mdurl-0.1.2-py3-none-any.whl.metadata (1.6 kB)
Collecting linkify-it-py<3,>=1 (from markdown-it-py[linkify,plugins]>=2.1.0->textual~=5.3.0->aiperf)
  Downloading linkify_it_py-2.0.3-py3-none-any.whl.metadata (8.5 kB)
Collecting mdit-py-plugins>=0.5.0 (from markdown-it-py[linkify,plugins]>=2.1.0->textual~=5.3.0->aiperf)
  Downloading mdit_py_plugins-0.5.0-py3-none-any.whl.metadata (2.8 kB)
Collecting uc-micro-py (from linkify-it-py<3,>=1->markdown-it-py[linkify,plugins]>=2.1.0->textual~=5.3.0->aiperf)
  Downloading uc_micro_py-1.0.3-py3-none-any.whl.metadata (2.0 kB)
Collecting contourpy>=1.0.1 (from matplotlib>=3.10.0->aiperf)
  Downloading contourpy-1.3.3-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl.metadata (5.5 kB)
Collecting cycler>=0.10 (from matplotlib>=3.10.0->aiperf)
  Downloading cycler-0.12.1-py3-none-any.whl.metadata (3.8 kB)
Collecting fonttools>=4.22.0 (from matplotlib>=3.10.0->aiperf)
  Downloading fonttools-4.61.1-cp312-cp312-manylinux1_x86_64.manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_5_x86_64.whl.metadata (114 kB)
Collecting kiwisolver>=1.3.1 (from matplotlib>=3.10.0->aiperf)
  Downloading kiwisolver-1.4.9-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (6.3 kB)
Collecting pyparsing>=3 (from matplotlib>=3.10.0->aiperf)
  Downloading pyparsing-3.3.2-py3-none-any.whl.metadata (5.8 kB)
Collecting pytest>=7.0.0 (from pytest-timeout>=2.4.0->kaleido~=1.2.0->aiperf)
  Downloading pytest-9.0.2-py3-none-any.whl.metadata (7.6 kB)
Collecting iniconfig>=1.0.1 (from pytest>=7.0.0->pytest-timeout>=2.4.0->kaleido~=1.2.0->aiperf)
  Downloading iniconfig-2.3.0-py3-none-any.whl.metadata (2.5 kB)
Collecting pluggy<2,>=1.5 (from pytest>=7.0.0->pytest-timeout>=2.4.0->kaleido~=1.2.0->aiperf)
  Downloading pluggy-1.6.0-py3-none-any.whl.metadata (4.8 kB)
Collecting six>=1.5 (from python-dateutil>=2.8.2->pandas~=2.3.3->aiperf)
  Downloading six-1.17.0-py2.py3-none-any.whl.metadata (1.7 kB)
Collecting huggingface-hub<2.0,>=1.3.0 (from transformers>=4.56.0->aiperf)
  Downloading huggingface_hub-1.5.0-py3-none-any.whl.metadata (13 kB)
Collecting pyyaml>=5.1 (from transformers>=4.56.0->aiperf)
  Downloading pyyaml-6.0.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.4 kB)
Collecting regex!=2019.12.17 (from transformers>=4.56.0->aiperf)
  Downloading regex-2026.2.28-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (40 kB)
Collecting tokenizers<=0.23.0,>=0.22.0 (from transformers>=4.56.0->aiperf)
  Downloading tokenizers-0.22.2-cp39-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (7.3 kB)
Collecting typer-slim (from transformers>=4.56.0->aiperf)
  Downloading typer_slim-0.24.0-py3-none-any.whl.metadata (4.2 kB)
Collecting safetensors>=0.4.3 (from transformers>=4.56.0->aiperf)
  Downloading safetensors-0.7.0-cp38-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (4.1 kB)
Collecting filelock>=3.10.0 (from huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading filelock-3.24.3-py3-none-any.whl.metadata (2.0 kB)
Collecting fsspec>=2023.5.0 (from huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading fsspec-2026.2.0-py3-none-any.whl.metadata (10 kB)
Collecting hf-xet<2.0.0,>=1.2.0 (from huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading hf_xet-1.3.2-cp37-abi3-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (4.9 kB)
Collecting httpx<1,>=0.23.0 (from huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading httpx-0.28.1-py3-none-any.whl.metadata (7.1 kB)
Collecting typer (from huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading typer-0.24.1-py3-none-any.whl.metadata (16 kB)
Collecting anyio (from httpx<1,>=0.23.0->huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading anyio-4.12.1-py3-none-any.whl.metadata (4.3 kB)
Collecting certifi (from httpx<1,>=0.23.0->huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading certifi-2026.2.25-py3-none-any.whl.metadata (2.5 kB)
Collecting httpcore==1.* (from httpx<1,>=0.23.0->huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading httpcore-1.0.9-py3-none-any.whl.metadata (21 kB)
Collecting h11>=0.16 (from httpcore==1.*->httpx<1,>=0.23.0->huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading h11-0.16.0-py3-none-any.whl.metadata (8.3 kB)
Collecting zipp>=3.20 (from importlib-metadata->dash~=3.1.0->aiperf)
  Downloading zipp-3.23.0-py3-none-any.whl.metadata (3.6 kB)
Collecting charset_normalizer<4,>=2 (from requests->dash~=3.1.0->aiperf)
  Downloading charset_normalizer-3.4.4-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (37 kB)
Collecting urllib3<3,>=1.21.1 (from requests->dash~=3.1.0->aiperf)
  Downloading urllib3-2.6.3-py3-none-any.whl.metadata (6.9 kB)
Collecting shellingham>=1.3.0 (from typer->huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading shellingham-1.5.4-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting annotated-doc>=0.0.2 (from typer->huggingface-hub<2.0,>=1.3.0->transformers>=4.56.0->aiperf)
  Downloading annotated_doc-0.0.4-py3-none-any.whl.metadata (6.6 kB)
Downloading aiperf-0.5.0-py3-none-any.whl (3.4 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.4/3.4 MB 10.3 MB/s  0:00:00
Downloading aiofiles-24.1.0-py3-none-any.whl (15 kB)
Downloading aiohttp-3.13.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (1.8 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.8/1.8 MB 103.7 MB/s  0:00:00
Downloading cyclopts-4.6.0-py3-none-any.whl (200 kB)
Downloading dash-3.1.1-py3-none-any.whl (7.9 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 7.9/7.9 MB 108.1 MB/s  0:00:00
Downloading dash_bootstrap_components-2.0.4-py3-none-any.whl (204 kB)
Downloading docstring_parser-0.17.0-py3-none-any.whl (36 kB)
Downloading ffmpeg_python-0.2.0-py3-none-any.whl (25 kB)
Downloading flask-3.1.3-py3-none-any.whl (103 kB)
Downloading jinja2-3.1.6-py3-none-any.whl (134 kB)
Downloading jmespath-1.0.1-py3-none-any.whl (20 kB)
Downloading kaleido-1.2.0-py3-none-any.whl (68 kB)
Downloading msgspec-0.20.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (224 kB)
Downloading multidict-6.7.1-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (256 kB)
Downloading numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (18.0 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 18.0/18.0 MB 109.5 MB/s  0:00:00
Downloading orjson-3.10.18-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (133 kB)
Downloading pandas-2.3.3-cp312-cp312-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl (12.4 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 12.4/12.4 MB 109.5 MB/s  0:00:00
Downloading pillow-11.1.0-cp312-cp312-manylinux_2_28_x86_64.whl (4.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.5/4.5 MB 106.3 MB/s  0:00:00
Downloading plotly-6.4.0-py3-none-any.whl (9.9 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 9.9/9.9 MB 109.1 MB/s  0:00:00
Downloading prometheus_client-0.23.1-py3-none-any.whl (61 kB)
Downloading psutil-7.0.0-cp36-abi3-manylinux_2_12_x86_64.manylinux2010_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl (277 kB)
Downloading pydantic-2.12.5-py3-none-any.whl (463 kB)
Downloading pydantic_core-2.41.5-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.1 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.1/2.1 MB 99.2 MB/s  0:00:00
Downloading pydantic_settings-2.13.1-py3-none-any.whl (58 kB)
Downloading pyzmq-26.4.0-cp312-cp312-manylinux_2_28_x86_64.whl (855 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 855.9/855.9 kB 93.1 MB/s  0:00:00
Downloading rich-14.1.0-py3-none-any.whl (243 kB)
Downloading pygments-2.19.2-py3-none-any.whl (1.2 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.2/1.2 MB 94.3 MB/s  0:00:00
Downloading rich_rst-1.3.2-py3-none-any.whl (12 kB)
Downloading ruamel_yaml-0.18.17-py3-none-any.whl (121 kB)
Downloading seaborn-0.13.2-py3-none-any.whl (294 kB)
Downloading setproctitle-1.3.7-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl (32 kB)
Downloading soundfile-0.13.1-py2.py3-none-manylinux_2_28_x86_64.whl (1.3 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.3/1.3 MB 93.4 MB/s  0:00:00
Downloading textual-5.3.0-py3-none-any.whl (702 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 702.7/702.7 kB 89.2 MB/s  0:00:00
Downloading platformdirs-4.9.2-py3-none-any.whl (21 kB)
Downloading typing_extensions-4.15.0-py3-none-any.whl (44 kB)
Downloading uvloop-0.21.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (4.7 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.7/4.7 MB 104.6 MB/s  0:00:00
Downloading werkzeug-3.1.6-py3-none-any.whl (225 kB)
Downloading yarl-1.22.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (377 kB)
Downloading aiohappyeyeballs-2.6.1-py3-none-any.whl (15 kB)
Downloading aiosignal-1.4.0-py3-none-any.whl (7.5 kB)
Downloading annotated_types-0.7.0-py3-none-any.whl (13 kB)
Downloading attrs-25.4.0-py3-none-any.whl (67 kB)
Downloading billiard-4.2.4-py3-none-any.whl (87 kB)
Downloading blinker-1.9.0-py3-none-any.whl (8.5 kB)
Downloading cffi-2.0.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (219 kB)
Downloading choreographer-1.2.1-py3-none-any.whl (49 kB)
Downloading click-8.3.1-py3-none-any.whl (108 kB)
Downloading frozenlist-1.8.0-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl (242 kB)
Downloading idna-3.11-py3-none-any.whl (71 kB)
Downloading itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Downloading logistro-2.0.1-py3-none-any.whl (8.6 kB)
Downloading markdown_it_py-4.0.0-py3-none-any.whl (87 kB)
Downloading mdurl-0.1.2-py3-none-any.whl (10.0 kB)
Downloading linkify_it_py-2.0.3-py3-none-any.whl (19 kB)
Downloading markupsafe-3.0.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (22 kB)
Downloading matplotlib-3.10.8-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (8.7 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 8.7/8.7 MB 106.7 MB/s  0:00:00
Downloading contourpy-1.3.3-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl (362 kB)
Downloading cycler-0.12.1-py3-none-any.whl (8.3 kB)
Downloading fonttools-4.61.1-cp312-cp312-manylinux1_x86_64.manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_5_x86_64.whl (5.0 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.0/5.0 MB 105.7 MB/s  0:00:00
Downloading kiwisolver-1.4.9-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (1.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.5/1.5 MB 97.8 MB/s  0:00:00
Downloading mdit_py_plugins-0.5.0-py3-none-any.whl (57 kB)
Downloading narwhals-2.17.0-py3-none-any.whl (444 kB)
Downloading packaging-26.0-py3-none-any.whl (74 kB)
Downloading propcache-0.4.1-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (221 kB)
Downloading pyarrow-23.0.1-cp312-cp312-manylinux_2_28_x86_64.whl (47.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 47.6/47.6 MB 94.7 MB/s  0:00:00
Downloading pyparsing-3.3.2-py3-none-any.whl (122 kB)
Downloading pytest_timeout-2.4.0-py3-none-any.whl (14 kB)
Downloading pytest-9.0.2-py3-none-any.whl (374 kB)
Downloading pluggy-1.6.0-py3-none-any.whl (20 kB)
Downloading iniconfig-2.3.0-py3-none-any.whl (7.5 kB)
Downloading python_dateutil-2.9.0.post0-py2.py3-none-any.whl (229 kB)
Downloading python_dotenv-1.2.1-py3-none-any.whl (21 kB)
Downloading pytz-2025.2-py2.py3-none-any.whl (509 kB)
Downloading ruamel_yaml_clib-0.2.15-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (788 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 788.2/788.2 kB 68.9 MB/s  0:00:00
Downloading simplejson-3.20.2-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl (152 kB)
Downloading six-1.17.0-py2.py3-none-any.whl (11 kB)
Downloading tqdm-4.67.3-py3-none-any.whl (78 kB)
Downloading transformers-5.2.0-py3-none-any.whl (10.4 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 10.4/10.4 MB 107.2 MB/s  0:00:00
Downloading huggingface_hub-1.5.0-py3-none-any.whl (596 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 596.3/596.3 kB 48.2 MB/s  0:00:00
Downloading hf_xet-1.3.2-cp37-abi3-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (4.2 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.2/4.2 MB 59.5 MB/s  0:00:00
Downloading httpx-0.28.1-py3-none-any.whl (73 kB)
Downloading httpcore-1.0.9-py3-none-any.whl (78 kB)
Downloading tokenizers-0.22.2-cp39-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (3.3 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.3/3.3 MB 86.6 MB/s  0:00:00
Downloading filelock-3.24.3-py3-none-any.whl (24 kB)
Downloading fsspec-2026.2.0-py3-none-any.whl (202 kB)
Downloading h11-0.16.0-py3-none-any.whl (37 kB)
Downloading pyyaml-6.0.3-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (807 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 807.9/807.9 kB 74.7 MB/s  0:00:00
Downloading regex-2026.2.28-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (802 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 802.0/802.0 kB 77.9 MB/s  0:00:00
Downloading safetensors-0.7.0-cp38-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (507 kB)
Downloading typing_inspection-0.4.2-py3-none-any.whl (14 kB)
Downloading tzdata-2025.3-py2.py3-none-any.whl (348 kB)
Downloading anyio-4.12.1-py3-none-any.whl (113 kB)
Downloading certifi-2026.2.25-py3-none-any.whl (153 kB)
Downloading docutils-0.22.4-py3-none-any.whl (633 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 633.2/633.2 kB 38.8 MB/s  0:00:00
Downloading future-1.0.0-py3-none-any.whl (491 kB)
Downloading importlib_metadata-8.7.1-py3-none-any.whl (27 kB)
Downloading zipp-3.23.0-py3-none-any.whl (10 kB)
Downloading nest_asyncio-1.6.0-py3-none-any.whl (5.2 kB)
Downloading pycparser-3.0-py3-none-any.whl (48 kB)
Downloading requests-2.32.5-py3-none-any.whl (64 kB)
Downloading charset_normalizer-3.4.4-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (153 kB)
Downloading urllib3-2.6.3-py3-none-any.whl (131 kB)
Downloading retrying-1.4.2-py3-none-any.whl (10 kB)
Downloading setuptools-82.0.0-py3-none-any.whl (1.0 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.0/1.0 MB 88.8 MB/s  0:00:00
Downloading typer-0.24.1-py3-none-any.whl (56 kB)
Downloading annotated_doc-0.0.4-py3-none-any.whl (5.3 kB)
Downloading shellingham-1.5.4-py2.py3-none-any.whl (9.8 kB)
Downloading typer_slim-0.24.0-py3-none-any.whl (3.4 kB)
Downloading uc_micro_py-1.0.3-py3-none-any.whl (6.2 kB)
Installing collected packages: pytz, zipp, uvloop, urllib3, uc-micro-py, tzdata, typing-extensions, tqdm, six, simplejson, shellingham, setuptools, setproctitle, safetensors, ruamel.yaml.clib, retrying, regex, pyzmq, pyyaml, python-dotenv, pyparsing, pygments, pycparser, pyarrow, psutil, propcache, prometheus-client, pluggy, platformdirs, pillow, packaging, orjson, numpy, nest-asyncio, narwhals, multidict, msgspec, mdurl, markupsafe, logistro, kiwisolver, jmespath, itsdangerous, iniconfig, idna, hf-xet, h11, future, fsspec, frozenlist, fonttools, filelock, docutils, docstring-parser, cycler, click, charset_normalizer, certifi, blinker, billiard, attrs, annotated-types, annotated-doc, aiohappyeyeballs, aiofiles, yarl, Werkzeug, typing-inspection, ruamel-yaml, requests, python-dateutil, pytest, pydantic-core, plotly, markdown-it-py, linkify-it-py, jinja2, importlib-metadata, httpcore, ffmpeg-python, contourpy, choreographer, cffi, anyio, aiosignal, soundfile, rich, pytest-timeout, pydantic, pandas, mdit-py-plugins, matplotlib, httpx, Flask, aiohttp, typer, seaborn, rich-rst, pydantic-settings, kaleido, dash, typer-slim, textual, huggingface-hub, dash-bootstrap-components, cyclopts, tokenizers, transformers, aiperf
Successfully installed Flask-3.1.3 Werkzeug-3.1.6 aiofiles-24.1.0 aiohappyeyeballs-2.6.1 aiohttp-3.13.3 aiosignal-1.4.0 aiperf-0.5.0 annotated-doc-0.0.4 annotated-types-0.7.0 anyio-4.12.1 attrs-25.4.0 billiard-4.2.4 blinker-1.9.0 certifi-2026.2.25 cffi-2.0.0 charset_normalizer-3.4.4 choreographer-1.2.1 click-8.3.1 contourpy-1.3.3 cycler-0.12.1 cyclopts-4.6.0 dash-3.1.1 dash-bootstrap-components-2.0.4 docstring-parser-0.17.0 docutils-0.22.4 ffmpeg-python-0.2.0 filelock-3.24.3 fonttools-4.61.1 frozenlist-1.8.0 fsspec-2026.2.0 future-1.0.0 h11-0.16.0 hf-xet-1.3.2 httpcore-1.0.9 httpx-0.28.1 huggingface-hub-1.5.0 idna-3.11 importlib-metadata-8.7.1 iniconfig-2.3.0 itsdangerous-2.2.0 jinja2-3.1.6 jmespath-1.0.1 kaleido-1.2.0 kiwisolver-1.4.9 linkify-it-py-2.0.3 logistro-2.0.1 markdown-it-py-4.0.0 markupsafe-3.0.3 matplotlib-3.10.8 mdit-py-plugins-0.5.0 mdurl-0.1.2 msgspec-0.20.0 multidict-6.7.1 narwhals-2.17.0 nest-asyncio-1.6.0 numpy-1.26.4 orjson-3.10.18 packaging-26.0 pandas-2.3.3 pillow-11.1.0 platformdirs-4.9.2 plotly-6.4.0 pluggy-1.6.0 prometheus-client-0.23.1 propcache-0.4.1 psutil-7.0.0 pyarrow-23.0.1 pycparser-3.0 pydantic-2.12.5 pydantic-core-2.41.5 pydantic-settings-2.13.1 pygments-2.19.2 pyparsing-3.3.2 pytest-9.0.2 pytest-timeout-2.4.0 python-dateutil-2.9.0.post0 python-dotenv-1.2.1 pytz-2025.2 pyyaml-6.0.3 pyzmq-26.4.0 regex-2026.2.28 requests-2.32.5 retrying-1.4.2 rich-14.1.0 rich-rst-1.3.2 ruamel-yaml-0.18.17 ruamel.yaml.clib-0.2.15 safetensors-0.7.0 seaborn-0.13.2 setproctitle-1.3.7 setuptools-82.0.0 shellingham-1.5.4 simplejson-3.20.2 six-1.17.0 soundfile-0.13.1 textual-5.3.0 tokenizers-0.22.2 tqdm-4.67.3 transformers-5.2.0 typer-0.24.1 typer-slim-0.24.0 typing-extensions-4.15.0 typing-inspection-0.4.2 tzdata-2025.3 uc-micro-py-1.0.3 urllib3-2.6.3 uvloop-0.21.0 yarl-1.22.0 zipp-3.23.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager, possibly rendering your system unusable. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv. Use the --root-user-action option if you know what you are doing and want to suppress this warning.
Usage: aiperf COMMAND
```

### NVIDIA AIPerf

```
╭─ Commands ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ analyze-trace  Analyze mooncake trace for prefix statistics                                                                                                                                                     │
│ plot           Generate visualizations from AIPerf profiling data.                                                                                                                                              │
│ profile        Run the Profile subcommand.                                                                                                                                                                      │
│ --help (-h)    Display this message and exit.                                                                                                                                                                   │
│ --version      Display application version.                                                                                                                                                                     │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```
