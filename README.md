# install_AlphaFold2_on_GCP_VM

```
# Create GCP VM with following spec
# GPU: A100, 40GB
# Disk: 4096 GB, SSD permanent
# Image: Google, Debian 10 based Deep Learning VM with , M108, Base CUDA 11.3, Deep Learning VM Image with CUDA 11.3 preinstalled.

# SSH to the VM

# some preparatory work
sudo apt remove docker-desktop
rm -r $HOME/.docker/desktop
sudo rm /usr/local/bin/com.docker.cli
sudo apt purge docker-desktop
sudo apt install gnome-terminal
sudo apt-get update

# Download docker image from https://www.docker.com/

# Install docker
sudo apt-get install ./docker-desktop-4.19.0-amd64.deb 
systemctl --user start docker-desktop
docker compose version
docker --version
docker version

# Add user to docker
sudo groupadd docker
sudo usermod -aG docker $USER

# Restart VM (i.e. stop it on GCP, start it again)

# test docker
docker run hello-world
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

# Install nvidia-container-toolkit
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# test docker can call cuda + GPU
sudo docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi

# Git clone AlphaFold2
mkdir script; cd script
git clone https://github.com/deepmind/alphafold.git
cd ./alphafold

# Download AlphaFold2 databases
mkdir ~/script/alphafold_database/
scripts/download_all_data.sh ~/script/alphafold_database/

# Build AlphaFold2 docker
docker build -f docker/Dockerfile -t alphafold .

# Create conda env for AlphaFold2
conda create -n AlphaFold python=3.8
conda activate AlphaFold
pip3 install -r docker/requirements.txt

# Restart VM (i.e. stop it on GCP, start it again)

# Test AlphaFold2
mkdir ~/test; mkdir ~/test/test_af2; cd ~/test/test_af2
touch example.fasta
echo \>example >> example.fasta
echo MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQRESTLHLVLRLRGG >> example.fasta

python3 /home/chunlai_tam/script/alphafold/docker/run_docker.py \
  --fasta_paths=/home/chunlai_tam/test/test_af2/example.fasta \
  --max_template_date=2022-01-01 \
  --data_dir=/home/chunlai_tam/script/alphafold_database/ \
  --output_dir=/home/chunlai_tam/test/test_af2/
```
