# Isambard-AI
Setup Isambard-AI access

## First thing First
1. Ask your supervisor to give access to ISAMBARD-AI Supercluster.
2. Your supervisor will send you an invitation via your manchester student EmailID and you have to "Accept" the invitation.
3. Then go to this link [https://portal.isambard.ac.uk/]
4. When prompted to choose access request type, choose: **University Login (MyAccessID)** and use your Manchester EmailID and password to gain access.
5. You will  see something like this when you have access to a project: ![Login Page](/images/login_portal.png) 

## Connect via Terminal
[BriCS facilities](https://docs.isambard.ac.uk/specs/) provides a command line tool called [Clifton](https://github.com/isambard-sc/clifton) for obtaining SSH certificates and configuring your SSH client to use these. The certificates are valid for 12 hours.
1. For Apple Silicon:
```bash
curl -L https://github.com/isambard-sc/clifton/releases/latest/download/clifton-macos-aarch64 -o clifton
chmod u+x clifton
sudo mv clifton /usr/local/bin/
```
2. For Linux Distro (including WSL)
```bash
curl -L https://github.com/isambard-sc/clifton/releases/latest/download/clifton-linux-musl-x86_64 -o clifton
chmod u+x clifton
mkdir -p ~/.local/bin
mv clifton ~/.local/bin/
```
3. Then create a ssh key using [GitHub's Documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
[Note]: They only support modern SSH keys (i.e. RSA keys of 3072 bits or more, or Ed25519, or any other modern, post-2014 key type). They do not support DSA keys, or RSA keys of less than 3072 bits. Any key generated using a modern version of ssh-keygen should be fine, e.g. the private key must contain `-----BEGIN OPENSSH PRIVATE KEY-----`
4. Next to Authenticate run `clifton auth --identity /path/to/ssh_key`
5. This will redirect you to web browser and you have to again use **University Login (MyAccessID)** to authenticate access to login node.
6. After successful login this will print something like this:
```
Successfully authenticated as YOUR_EMAIL_ADDRESS (YOUR_SHORT_NAME) and downloaded SSH certificate for projects:
 - PROJECT_NAME

Certificate file written to ~/.ssh/id_ed25519-cert.pub
Certificate valid for 11 hours and 59 minutes.
You may now want to run `clifton ssh-config write` to configure your SSH config aliases.
```
6. Then write the shh key to your local workspace using `clifton ssh-config write`
7. Finally to connect to the login node run `ssh <PROJECT_ID>.aip2.isambard`.

## Connect VSCode via Remote Tunnel
1. Connect to a loging node and install VSCode:
```shell
curl --location --output vscode_cli.tar.gz "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64"
mkdir -p ~/opt/vscode_cli
tar -C ~/opt/vscode_cli --extract --verbose --file vscode_cli.tar.gz
```
2. Submit a jobscript with the following details:
##### vscode_code_tunnel.sh
```shell
#!/bin/bash
#SBATCH --job-name=code_tunnel
#SBATCH --gpus=1            # this also allocates 72 CPU cores and 115GB memory
#SBATCH --time=1:00:00
#SBATCH --output=code_tunnel_%j.out

module load brics/nano
# Start named VS Code tunnel for remote connection to compute node
~/opt/vscode_cli/code tunnel --name "i-ai_compute"
```
3. Then run it via sbatch like you used to work with CSF Cluster: `sbatch vscode_code_tunnel.sh`
4. Then use this link to setup remote connection via GitHub [https://github.com/login/device].
5. For the first time, it will ask you an "eight digit code" which you can find it inside your output file of the jobscript. The file will be named something like this `vs_code_tunnel_1182526.out`.
6. Once done open your VSCode, click on the bottom left corner and then use "Connect to Tunnel", then "GitHub" and then you are ready to go!
7. For subsequent Remote Tunnel connection, just submit the jobscript and you can directly connect you VSCode via "Connect to Tunnel" button.

## Some Final Steps:
1. Install mniforge so that pytorch can be run on GPU. Follow the steps provided in this link to [install python environment](https://docs.isambard.ac.uk/user-documentation/guides/python/#conda-installing-and-using-miniforge).
2. Take help of the following tutorial to setup a [Distributed Environment for training in PyTorch](https://docs.isambard.ac.uk/user-documentation/tutorials/distributed-training/)
3. Freqently visit the following link to familiarize yourself with the [SLURM Management System of Isambard](https://docs.isambard.ac.uk/user-documentation/guides/slurm/). (PS its almost similar to our new CSF SLURM system)