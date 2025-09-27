# 🔮 Ollama Private Translator (AWS/Ollama/Gemma3N)
<img width="1074" height="684" alt="image" src="https://github.com/user-attachments/assets/6e808f86-3249-4c81-bfc8-0aad609b4c60" />

Hey there! 👋 Welcome to my project—a secure, private translation tool built primarily for me (and maybe a few internal folks) to handle sensitive text without sending it to big public APIs. Think of it as **"Google Translate, but running on my own cloud GPU."**

The whole point is privacy: **what you translate stays on my server and is never stored.**

---

## ✨ The Vibe & Core Features

* **Privacy First:** All translation happens on my own AWS instance. The application is totally stateless—I don't save any translation history, ever.
* **Speed Demon:** Runs on an NVIDIA-accelerated EC2 instance, making inference super fast.
* **Model of Choice:** Currently running the mighty **`gemma3n`** model via Ollama.
* **Clean UI:** Simple, Google Translate-style interface with a live character count (max 1000 chars) and a dark mode toggle for late-night hacking.
* **Developer Friendly:** Frontend uses a sweet little JS hack to chunk large text, ensuring the model doesn't choke on long documents.

---

## 🏗️ Architecture: From Hobby to Hero

This project is currently in the process of leveling up its infrastructure game!

### Phase 1: The Bootstrap Setup (Current State) 🚀

Right now, everything is happily living together on one box:

* **The Brains:** A single **EC2 `g4dn.xlarge`** instance (that sweet GPU!).
* **The Party:** **Nginx** (serving the UI) and **Ollama** (serving the API) are co-located.
* **The Good News:** Since it's internal-use-only, access is locked down at the network level via Security Groups, so no random internet traffic gets in.

### Phase 2: The Production Dream (Future Goal) 🌟

The goal is to move to a proper, resilient architecture using core AWS services:

| Component | Goal | Why? |
| :--- | :--- | :--- |
| **Frontend** | Move to **S3 + CloudFront** | Better caching, faster loading globally, and automatic HTTPS security. |
| **Load Balancing** | Introduce **ALB (Application Load Balancer)** | Distribute translation requests, handle SSL termination, and act as a secure gateway. |
| **Scalability** | Introduce **Auto Scaling Group (ASG)** | If a lot of people hit the translator at once, the ASG will spin up more `g4dn.xlarge` instances and automatically manage the capacity. |
| **Network** | **Private Subnets** | Lock the Ollama servers away where they can only talk to the ALB, not the scary public internet. |

---

## 🔒 Security & Maintenance Notes

The most important rule for this project: **TRUST NO ONE (except my own VPC).**

* **Access Lock-down:** The entire application is protected by a network firewall. Only traffic coming from known, authorized internal IPs (like my VPN) will be allowed to hit the load balancer.
* **No Data Spills:** The Ollama servers are behind the ALB and are placed in **Private Subnets**. This is key to preventing any accidental data exposure.
* **Patching:** Gotta keep the OS, NVIDIA drivers, and Ollama itself updated. I'll be baking these updates into new AMIs/Launch Templates for the ASG to use.
* **Cost Control:** The **ASG** will be configured to scale down to zero (or a minimum of one) when not in use, ensuring I only pay for that pricey GPU when I absolutely need it!

---

### Contribution

Feel free to check out the code, especially the JavaScript frontend logic for chunking large text and handling the character count—it was a fun little challenge! Any ideas on optimizing the Modelfile for better `gemma3n` performance are always welcome.

Happy translating! 🚀
