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

%cd /home/jovyan/workspace/ui
!python launch.py --listen --xformers --enable-insecure-extension-access --theme dark --gradio-queue --disable-safe-unpickle --port 8000
```