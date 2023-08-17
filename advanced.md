## Advanced Tutorial
https://www.youtube.com/watch?v=6zixxc-4_bA <br />

## Pip requirements:
```py
--extra-index-url https://download.pytorch.org/whl/cu118
torch==2.0.1+cu118
torchvision==0.15.2+cu118
torchaudio==2.0.2+cu118
torchtext==0.15.2
torchdata==0.6.1
xformers==0.0.20
triton==2.0.0
```

## Apt Packages:
```py
aria2
libgl1
libglib2.0-0
git-lfs
```

## Jupyter Notebook With CloudFlared Tunnel
```py
%cd /home/jovyan/workspace
%env TF_CPP_MIN_LOG_LEVEL=1

!git clone -b v2.5 https://dagshub.com/camenduru/ui
!aria2c --console-log-level=error -c -x 16 -s 16 -k 1M https://huggingface.co/embed/upscale/resolve/main/4x-UltraSharp.pth -d /home/jovyan/workspace/ui/models/ESRGAN -o 4x-UltraSharp.pth
!git clone https://github.com/camenduru/tunnels /home/jovyan/workspace/ui/extensions/tunnels
!git clone https://github.com/etherealxx/batchlinks-webui /home/jovyan/workspace/ui/extensions/batchlinks-webui
%cd /home/jovyan/workspace/ui
!git reset --hard
!git -C /home/jovyan/workspace/ui/repositories/stable-diffusion-stability-ai reset --hard

!aria2c --console-log-level=error -c -x 16 -s 16 -k 1M https://huggingface.co/ckpt/CyberRealistic/resolve/main/cyberrealistic_v32.safetensors -d /home/jovyan/workspace/ui/models/Stable-diffusion -o cyberrealistic_v32.safetensors

!sed -i -e '''/from modules import launch_utils/a\import os''' /home/jovyan/workspace/ui/launch.py
!sed -i -e '''/        prepare_environment()/a\        os.system\(f\"""sed -i -e ''\"s/dict()))/dict())).cuda()/g\"'' /home/jovyan/workspace/ui/repositories/stable-diffusion-stability-ai/ldm/util.py""")''' /home/jovyan/workspace/ui/launch.py
!sed -i -e 's/\["sd_model_checkpoint"\]/\["sd_model_checkpoint","sd_vae","CLIP_stop_at_last_layers"\]/g' /home/jovyan/workspace/ui/modules/shared.py

!wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -O /home/jovyan/workspace/cloudflared-linux-amd64 && chmod 777 /home/jovyan/workspace/cloudflared-linux-amd64
import atexit, requests, subprocess, time, re, os
from random import randint
from threading import Timer
from queue import Queue
def cloudflared(port, metrics_port, output_queue):
    atexit.register(lambda p: p.terminate(), subprocess.Popen(['/home/jovyan/workspace/cloudflared-linux-amd64', 'tunnel', '--url', f'http://127.0.0.1:{port}', '--metrics', f'127.0.0.1:{metrics_port}'], stdout=subprocess.DEVNULL, stderr=subprocess.STDOUT))
    attempts, tunnel_url = 0, None
    while attempts < 10 and not tunnel_url:
        attempts += 1
        time.sleep(3)
        try:
            tunnel_url = re.search("(?P<url>https?:\/\/[^\s]+.trycloudflare.com)", requests.get(f'http://127.0.0.1:{metrics_port}/metrics').text).group("url")
        except:
            pass
    if not tunnel_url:
        raise Exception("Can't connect to Cloudflare Edge")
    output_queue.put(tunnel_url)
output_queue, metrics_port = Queue(), randint(8100, 9000)
thread = Timer(2, cloudflared, args=(7860, metrics_port, output_queue))
thread.start()
thread.join()
tunnel_url = output_queue.get()
os.environ['webui_url'] = tunnel_url
print(tunnel_url)

%cd /home/jovyan/workspace/ui
!python launch.py --listen --xformers --enable-insecure-extension-access --theme dark --gradio-queue --disable-safe-unpickle
```