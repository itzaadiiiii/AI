```markdown
# 🚀 Complete Guide: Agentic AI IDE Setup on AWS EC2 Ubuntu  
**VS Code + Continue.dev + Ollama (Full-Precision 13–14 GB Models Supported)**

This is a **single, production-ready Markdown guide** that covers **both budget (CPU)** and **full-precision (GPU)** setups.  
Follow the steps exactly as written. All commands are ready to copy-paste.

---

## 🧠 1. EC2 Instance Sizing & Cost (Mumbai Region – ap-south-1)

### Option A: Budget / CPU-only (Quantized Models – Recommended to Start)
| Instance     | vCPU | RAM  | GPU     | Hourly (₹) | Monthly 24/7 (₹) |
|--------------|------|------|---------|------------|------------------|
| **t3.2xlarge** | 8    | 32 GB | None    | ~28        | ~20,000         |

**Best for**: 7B/8B quantized models (llama3, mistral).  
**Model size on disk**: 4–8 GB.  
**Performance**: Good enough for coding/agent tasks.

### Option B: Full-Precision GPU (13–14 GB+ models – What You Asked For)
| Instance      | vCPU | RAM   | GPU                  | VRAM   | Hourly (₹) | Monthly 24/7 (₹) |
|---------------|------|-------|----------------------|--------|------------|------------------|
| **g5.xlarge**   | 4    | 16 GB | 1× NVIDIA A10G       | 24 GB  | ~90        | ~65,000         |
| **g5.2xlarge** (Recommended) | 8 | 32 GB | 1× NVIDIA A10G       | 24 GB  | ~120       | ~90,000         |
| **g5.12xlarge** (Heavy) | 48 | 192 GB | 4× NVIDIA A10G     | 96 GB  | ~1,000+    | 7L+             |

**Why g5.2xlarge?**  
- Handles **full-precision / FP16 13B models** comfortably (~24–28 GB VRAM needed).  
- Future-proof for multi-agent workflows and larger models.  
- **Storage**: 150 GB gp3 (minimum).

**💡 Cost-Saving Tip (Both Options)**  
Stop the instance when not in use → saves **80–90 %**.  
Use `aws ec2 stop-instances --instance-ids i-xxx` or the console.

---

## ⚙️ 2. Launch EC2 Instance (AWS Console)

1. Go to **EC2 → Launch instance**
2. **Name**: `agentic-ai-ide`
3. **AMI**: **Ubuntu Server 22.04 LTS** (64-bit, x86)
4. **Instance type**:
   - Budget → `t3.2xlarge`
   - Full-precision → `g5.2xlarge` (recommended)
5. **Key pair**: Create or use existing (`key.pem`)
6. **Storage**: 
   - Budget → 100 GB gp3
   - Full-precision → **150 GB gp3**
7. **Security Group** (create new):
   - SSH (22) → Anywhere (or your IP)
   - Custom TCP **11434** → Anywhere (Ollama API)
8. Launch → Wait for **Running** status.

---

## 🔐 3. Connect to EC2

```bash
chmod 400 key.pem
ssh -i key.pem ubuntu@<YOUR_EC2_PUBLIC_IP>
```

---

## 🧰 4. Install Base Dependencies (Common)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl build-essential htop
```

---

## 🔧 5. GPU-Only: Install NVIDIA Drivers & CUDA (Full-Precision Only)

```bash
sudo apt update
sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall
sudo reboot
```

**After reboot**:

```bash
nvidia-smi                    # ← Should show A10G GPU
sudo apt install -y nvidia-cuda-toolkit
```

---

## 🦙 6. Install Ollama (Common)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama --version
```

---

## 📦 7. Pull Model

**Budget / CPU** (quantized):
```bash
ollama pull llama3          # ~4–8 GB
# or
ollama pull mistral
```

**Full-Precision / GPU** (13B+):
```bash
ollama pull llama3:13b      # or llama3:70b if you have enough VRAM
```

**Verify GPU usage** (full-precision only):
```bash
watch -n 1 nvidia-smi
```

---

## 🌐 8. Expose Ollama API (So VS Code Can Connect)

```bash
sudo nano /etc/systemd/system/ollama.service
```

**Add / edit** the `[Service]` section:
```ini
Environment="OLLAMA_HOST=0.0.0.0"
```

Save & restart:
```bash
sudo systemctl daemon-reexec
sudo systemctl restart ollama
sudo systemctl enable ollama
```

Check status:
```bash
sudo systemctl status ollama
```

---

## 💻 9. Local Machine – Install VS Code & Extensions

1. Download & install **Visual Studio Code** (https://code.visualstudio.com)
2. Install these two extensions:
   - **Remote - SSH**
   - **Continue** (by Continue.dev)

---

## 🔗 10. Connect VS Code to EC2

1. Press `Ctrl + Shift + P` → type **Remote-SSH: Connect to Host**
2. Click **Add New SSH Host**
3. Paste:
   ```bash
   ssh -i ~/Downloads/key.pem ubuntu@<YOUR_EC2_PUBLIC_IP>
   ```
4. Select the config file → Connect.

---

## 🤖 11. Configure Continue.dev with Ollama

In VS Code, open the Continue config:
- Press `Ctrl + Shift + P` → **Continue: Open Config**

Replace the content with:

```json
{
  "models": [
    {
      "title": "Ollama Full-Precision LLaMA",
      "provider": "ollama",
      "model": "llama3:13b",          // Change to "llama3" for budget CPU
      "apiBase": "http://<YOUR_EC2_PUBLIC_IP>:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Ollama Autocomplete",
    "provider": "ollama",
    "model": "llama3:13b"
  }
}
```

Save (`Ctrl + S`). Continue.dev will auto-reload.

---

## 🧪 12. Test the Full Setup

1. Open any file in the remote workspace.
2. Highlight code → Right-click → **Continue: Ask Continue**
3. Or press `Ctrl + L` and type:
   - “Explain this code”
   - “Write a Kubernetes deployment for this service”
   - “Optimize this Python script for performance”

You are now running a **full agentic AI IDE** powered by your EC2 Ollama model.

---

## 🔁 13. Daily Workflow & Maintenance

- **Start instance** when you begin work.
- **Stop instance** when done:
  ```bash
  aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxx
  ```
- Check Ollama status anytime:
  ```bash
  ollama list
  curl http://localhost:11434/api/tags
  ```
- Monitor GPU (full-precision):
  ```bash
  watch -n 1 nvidia-smi
  ```

---

## 🔥 Architecture Flow

```
Your Laptop (VS Code + Continue.dev)
          ↓ (Remote-SSH)
EC2 Ubuntu (g5.2xlarge or t3.2xlarge)
          ↓ (HTTP)
Ollama API (0.0.0.0:11434)
          ↓
LLM (llama3:13b full-precision on GPU)
          ↓
Responses back to IDE in real-time
```

---

**✅ Done!** You now have a complete, private, agentic AI coding environment running full-precision Ollama models.

Want me to add the next level?
- Multi-agent setup (CrewAI / LangChain)
- Auto-scaling Kubernetes + Ollama
- DeepSeek-Coder or CodeLlama full-precision
- Persistent storage with EFS

Just reply **“Next level”** and I’ll give you the full advanced guide.  

Happy coding! 🚀
```