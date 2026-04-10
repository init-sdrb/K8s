# 🚢 Harbor Registry on Kubernetes

Automated deployment of Harbor private container registry on a bare-metal Kubernetes cluster using GitHub Actions, Ansible, and MetalLB.

## Infrastructure

| Hostname | IP Address | Role | OS | User |
|----------|------------|------|-----|------|
| runner.deep.local | 192.168.21.21 | GitHub Runner + Ansible Controller | AlmaLinux
| master.deep.local | 192.168.21.120 | Kubernetes Master | Ubuntu
| worker01.deep.local | 192.168.21.121 | Kubernetes Worker | Ubuntu 
| worker02.deep.local | 192.168.21.122 | Kubernetes Worker | Ubuntu 



