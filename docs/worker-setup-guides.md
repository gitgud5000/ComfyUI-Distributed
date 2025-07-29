## ComfyUI Distributed Worker Setup Guide

ComfyUI-Distributed allows you to distribute your workflows across multiple GPUs, whether they're in the same machine, on your local network, or in the cloud.

**Master**: The main ComfyUI instance that coordinates and distributes work. This is where you load workflows, manage the queue, and view results.

**Worker**: A ComfyUI instance that receives and processes tasks from the master. Workers handle just the GPU computation and send results back to the master. You can have multiple workers connected to a single master, each utilizing their own GPU.

### Types of Workers

- **Local workers**: Additional GPUs on the same machine as the master
- **Remote workers**: GPUs on different computers within your network
- **Cloud workers**: GPUs hosted on cloud services like Runpod

## Local workers
> These are added automatically on first launch, but you can add them manually if you need to.

1. **Open** the Distributed GPU panel.
2. **Click** "Add Worker" in the UI.
3. **Configure** your local worker:
   - **Name**: A descriptive name for the worker (e.g., "Studio PC 1")
   - **Port**: A unique port number for this worker (e.g., 8189, 8190...).
   - **CUDA Device**: The GPU index from `nvidia-smi` (e.g., 0, 1).
   - **Extra Args**: Optional ComfyUI arguments for this specific worker.
4. **Save** and  launch the local worker.

## Remote workers
> ComfyUI instances running on completely different computers on your network. These allow you to harness GPU power from other machines. Remote workers must be manually started on their respective computers and are connected via IP address.

**On the Remote Worker Machine:**
1. **Launch** ComfyUI with the `--listen --enable-cors-header` arguments. ⚠️ **Required!**
   - This ComfyUI instance will serve as a worker for your main master.
2. *Optionally* add additional local workers on this machine if it has multiple GPUs:
   - Access the Distributed GPU panel in this ComfyUI instance
   - Add workers for any additional GPUs (if they haven't been added automatically)
   - Make sure they have `--listen` set in `Extra Args`
   - Launch them
3. **Open** the ComfyUI port (e.g., 8188) and any additional worker ports (e.g., 8189, 8190) in the firewall.
  
**On the Main Machine:**
1. **Launch** ComfyUI with `--enable-cors-header` launch argument.
2. **Open** the Distributed GPU panel (sidebar on the left).
3. **Click** "Add Worker."
4. **Choose** "Remote".
5. **Configure** your remote worker:
   - **Name**: A descriptive name for the worker (e.g., "Server Rack GPU 0")
   - **Host**: The remote worker's IP address.
   - **Port**: The port number used when launching ComfyUI on the remote master/worker (e.g., 8188).
6. **Save** the remote worker configuration.
  
## Cloud workers
> ComfyUI instances running on a cloud service like [Runpod](https://get.runpod.io/0bw29uf3ug0p). 

### Deploy Cloud Worker on Runpod

**On Runpod:**
1. Register a [Runpod](https://get.runpod.io/0bw29uf3ug0p) account.
2. On Runpod, go to Storage > New Network Volume and create a volume that will store the models you need. Start with 40 GB, you can always add more later.
3. Now go to Pods and find a suitable GPU for your workflows. 
4. Choose the [ComfyUI Distributed Pod](https://console.runpod.io/deploy?template=m21ynvo8yo&ref=ak218p52) template and make sure your network drive is mounted.
> To use the ComfyUI Distributed Pod template, you will need to filter instances by CUDA 12.8.
6. Launch your pod.
7. Access your pod using JupyterLabs.
8. Download models into /workspaces/ComfyUI/models/ (these will remain on your network drive even after you terminate the pod).
> You can use [this guide](model-download-script.md) to make this process easy for you. It will generate a shell script that will automatically download the models you need for a given workflow.
8. If using your own template, make sure you launch ComfyUI with the `--enable-cors-header` argument and you git clone ComfyUI-Distributed into custom_nodes. ⚠️ **Required!**
9. Download any additional custom nodes using the ComfyUI Manager.

**On the Main Machine:**
1. Launch a Cloudflare tunnel
   - Download from here: [https://github.com/cloudflare/cloudflared/releases](https://github.com/cloudflare/cloudflared/releases)
	- Then run, for example: `cloudflared-windows-amd64.exe tunnel --url http://localhost:8188`
> Cloudflare tunnels create secure connections without exposing ports directly to the internet
2. Copy the Cloudflare address
3. **Launch** ComfyUI with `--enable-cors-header` launch argument.
4. **Open** the Distributed GPU panel (sidebar on the left).
5. **Edit** the Master's host address and replace it with the Cloudflare address.
   - **Click** "Add Worker."
   - **Choose** "Cloud".
   - **Configure** your cloud worker:
     - **Host**: The Runpod address
     - **Port**: 443
   - **Save** the remote worker configuration.

---

### Deploy Cloud Worker on Other Platforms

**On the Cloud Worker machine:**
   - Your cloud worker container needs to have the same models and custom nodes as the workflow you want to run on your local machine.
   - If your cloud platform doesn't provide a secure connection, use Cloudflare to create a tunnel for the worker. Each GPU needs their own tunnel for their respective port.
	- For example: `./cloudflared tunnel --url http://localhost:8188`
1. **Launch** ComfyUI with the `--listen --enable-cors-header` arguments. ⚠️ **Required!**
2. **Add** workers in the UI panel if the remote machine has more than one GPU.
   - Make sure that they also have `--listen` set in `Extra Args`.
   - Then launch them.
  
**On the Main Machine:**
1. **Launch** a Cloudflare tunnel on your local machine.
   - Download from here: [https://github.com/cloudflare/cloudflared/releases](https://github.com/cloudflare/cloudflared/releases)
   - Then run, for example: `cloudflared-windows-amd64.exe tunnel --url http://localhost:8188`
2. **Copy** the Cloudflare address
3. **Launch** ComfyUI with `--enable-cors-header` launch argument.
4. **Open** the Distributed GPU panel (sidebar on the left).
5. **Edit** the Master's host address and replace it with the Cloudflare address.
6. **Click** "Add Worker."
7. **Choose** "Cloud".
8. **Configure** your cloud worker:
   - **Host**: The remote worker's IP address/domain
   - **Port**: 443
9. **Save** the remote worker configuration.
