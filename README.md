# Playbook play_hashicorp_vault

```bash

rm -rf venv/
rm -rf /root/.cache/pip/

mkdir -p venv
python3 -m venv venv
source venv/bin/activate

pip3 install -r requirements.txt --upgrade

# ansible-galaxy install -v -f -r roles/requirements.yml

export ANSIBLE_LOG_PATH="ansible_$(date +"%y%m%d%H%M%S").log"
echo "Logdatei: ${ANSIBLE_LOG_PATH}"

ansible --version | head -n 1
ansible --version | grep "python version"

ansible-playbook -i inventories/test/hosts.ini -v playbook-build.yml
ansible-playbook -i inventories/test/hosts.ini -v playbook-destroy.yml

ansible-playbook -i inventories/test/hosts.ini -v playbook-rebuild.yml
ansible-playbook -i inventories/test/hosts.ini -v -e "rl_hashicorp_vault_check_api_reachable_ignore_errors=true" playbook-rebuild.yml
```