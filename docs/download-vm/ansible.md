# 1. Create ansible folder

```
mkdir -p ansible/
```

# 2. Check status of inventory

```
ansible-inventory -i inventory.ini --list

ansible minio -m ping -i inventory.ini
```

# 3. Run playbook 

```
ansible-playbook -i inventory.ini setup-openebs.yaml
```