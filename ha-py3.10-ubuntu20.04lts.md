# Installing Home Assistant 2022.07 (or latest) over python 3.10 on Ubuntu 20.04 LTS

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt install python3.10 python3.10-dev python3.10-venvpython3.10 -m venv 
python3.10 -c 'print("hello world!")'

cd
mkdir homeassistant-py3.10
cd    homeassistant-py3.10
python3.10 -m venv .
source bin/activate

# from now on, command operate on the venv
python3 -m pip install wheel
pip3 install homeassistant

```
