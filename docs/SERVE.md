# Serving LLM With FastChat (by ISL)

## FastChat Architecture

![server arch](../assets/server_arch.png)


## Serve With Docker

Check out to `dev` branch:
```shell
git checkout dev
```

### Serving

Our `dev` image is hosted and stored in our own `ghcr` (GitHub container registry) at [`ghcr.io/intelligent-systems-lab/fastchat`](https://github.com/orgs/Intelligent-Systems-Lab/packages/container/package/fastchat).

To serve the `controll plane`, run `docker/docker-compose-controll.yml`:
```shell
# Pull the image
docker pull ghcr.io/intelligent-systems-lab/fastchat:latest

# Run with docker compose
docker compose -f docker-compose-controll.yml up
```
> :bulb: If bump denied to access this image, you need to login to ghcr.io by `docker login ghcr.io`.

<!-- To serve the `model worker`, run:
```shell
docker run -d ghcr.io/intelligent-systems-lab/fastchat python3.9 -m fastchat.serve.model_worker --model-names
``` -->

### Developing

If you wish to rebuild the image, run:
```shell
# Build the image
docker build -t ghcr.io/intelligent-systems-lab/fastchat -f Dockerfile.dev .

# Push the image
docker push ghcr.io/intelligent-systems-lab/fastchat
```


## Serving With Host OS

There are essentially **3** components to start in order to serve models with FastChat, see the above system architecture for more details:
- Controller
- Model Worker
- Web Server (Gradio or OpenAI API)

> :bulb: It is recommended to run in `screen`. If you are not familiar with `screen`, see this [tutorial](https://linuxize.com/post/how-to-use-linux-screen/).

### Installation

1. Clone this repo:  
   ```
   git clone https://github.com/Intelligent-Systems-Lab/FastChat.git
   ```
2. Create a virtual environment:  
   For example, with anaconda: `conda create -n fastchat python==3.10.11`
3. Install dependencies:
   ```shell
   # Go to the FastChat folder
   cd <fastchat_location>

   # Upgrade pip
   pip install --upgrade pip
   
   # Install with dev mode
   pip install -e .
   ```

You're all set.

### Controller

Available arguments:
- `--host`: Where to host the controller, default `localhost`.
- `--port`: Which port to host the controller, default `21001`.
- `--dispatch-method`: How to return `worker_address` (does not impact our application), default `shorteset_queue`.

To serve (set the `--host` if you want to serve FastChat distributedly):
```shell
python -m fastchat.serve.controller --host 0.0.0.0
```


### Web Server (Gradio)

Available arguments:
- `--host`: Where to host the web server, default `0.0.0.0`.
- `--port`: Which port to host the web server, default set to gradio's default port `7860`.
- `--controller-url`: The address of the controller (should be in full format, e.g. `http://<host>:<port>`), default `http://localhost:21001`.
- `--model-list-mode`: Whether to load the model list once or reload the model list every time, default `once`.

To serve (it is recommeded to set model-list-mode to reload when experimenting):
```shell
python -m fastchat.serve.gradio_web_server --controller-url <controller_url> --model-list-mode reload
```


### OpenAI API Server

Available arguments:
- `--host`: Where to host the api server, default `localhost`.
- `--port`: Which port to host the api server, default `8000`.
- `--controller-url`: The address of the controller (should be in full format, e.g. `http://<host>:<port>`), default `http://localhost:21001`.

To serve:
```shell
python -m fastchat.serve.openai_api_server --host 0.0.0.0 --controller-address http://hc2.isl.lab.nycu.edu.tw:21001
```


### Model Worker

Available arguments:
- `--host`: Where to host the model, default `0.0.0.0`.
- `--port`: Which port to host the model, default `21002`.
- `--model-path`: The model to serve (follows HF conventions).
- `--worker-address`: Worker address is the key for controller to store available model workers (should be in full format, e.g. `http://<host>:<port>`), default `http://localhost:21002`.
- `--controller-address`: The address of the controller, default `http://localhost:21001`.
- `--num-gpus`: Number of GPU to use for inferencing, default `1`.

To serve (set `--host` to 0.0.0.0 if you want to serve FastChat distributedly):
```shell
python -m fastchat.serve.model_worker --model-path lmsys/vicuna-13b-v1.3 --controller-address <controller_address> --host 0.0.0.0 --port <port> --worker-address http://<worker_address>:<worker_port> --num-gpus 2
```

> :warning: Be careful with the port if you are serving more than 1 model in 1 server.  
> :warning: Be careful with `--worker_address`, you need to specify the right value for others to find the right endpoint.
