# How to Upload This to GitHub

## Step 1 — Create the repo on GitHub
1. Go to https://github.com/new
2. Name it: `nsm-lab` (or `network-security-monitoring-lab`)
3. Set to **Public**
4. ❌ Do NOT check "Add a README file" (you already have one)
5. Click **Create repository**

## Step 2 — On your local machine, copy these files into a folder
Put all files from this zip into a folder called `nsm-lab/`

## Step 3 — Initialize and push

```bash
cd nsm-lab
git init
git add .
git commit -m "Initial commit: NSM lab with TShark, Zeek, and Suricata"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/nsm-lab.git
git push -u origin main
```

Replace YOUR_USERNAME with your actual GitHub username.

## Step 4 — Add screenshots (important for portfolio!)
After pushing, go back to your VM and take screenshots of:
- Suricata `fast.log` showing your custom rules firing
- Zeek `conn.log` output
- Nmap scan triggering the port scan alert
- curl triggering the User-Agent alert

Add them to the `screenshots/` folder and push again:
```bash
git add screenshots/
git commit -m "Add evidence screenshots"
git push
```

## Step 5 — Add a GitHub repo description
On your GitHub repo page, click the gear ⚙️ next to "About" and add:
> "3-layer NSM stack: TShark + Zeek + Suricata with custom IDS rules on a virtualized network"

Add topics: `cybersecurity`, `soc`, `network-security`, `suricata`, `zeek`, `ids`, `homelab`
