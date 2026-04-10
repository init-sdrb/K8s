# 🚢 Harbor Registry on Kubernetes with GitOps

Automated deployment of Harbor private container registry on a bare-metal Kubernetes cluster using GitHub Actions, Ansible, and MetalLB.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Infrastructure](#infrastructure)
- [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)
- [Quick Start](#quick-start)
- [Deployment Workflows](#deployment-workflows)
- [Accessing Harbor](#accessing-harbor)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)

# Architecture

Kubernetes expert insights and solutions
Fix SSH libcrypto key error
S3FS Mount Freeze Troubleshooting
Remote Utilities Installation Guide
LLM and AI Platform Components Explained
Types and Uses of Virtual Private Networks
Cloud Init Workflow Timing
oVirt Engine Deployment Network Connectivity Issue
Groovy Script Deployment Error Fix Guide
Monitor SSL Certificate Expiry with Zabbix
SSL expiry alert setup for multiple domains
Zero Trust Tools in DevOps and SysAdmin
Serverless Architecture Explained for DevOps
Responding to Work-from-Home Approval Details
HAProxy Host Visibility in CrowdSec Setup
Understanding AWS Virtual Private Cloud (VPC)
Master VMware Cloud Foundation Job Role Essentials
Mango Seed Business Opportunities and Challenges
MSSQL Backup Job Fails Randomly: Network Issues
Business Opportunities in Nepal's Geography and Culture
Deploy CrowdSec WAF on Ubuntu 22 Server
Open-Source WAF Solutions for Server Filtering
GitLab PostgreSQL Initialization Issue Fix
Proxmox VE Installation on Dell R940
BenQ GW2790 vs GW2791 Monitor Comparison
Monthly Salary Breakdown in Portrait Mode
Maven Dependency Resolution Failure Issue
Convert 12TB to MB Calculation
Senior System Administrator Role and Key Skills
Reformatting and Summarizing Task Instructions
Kubernetes Architecture and Beyond for DevOps
Paraphrasing IT System and Network Details
VMware vMotion Configuration for Host Migration
Assisting with Image Manipulation Techniques
Daily Jenkins Backup Script Creation Guide
Jenkins Deployment Rollback Strategies Guide
GitLab MFA Options and Security Practices
Secure Document Upload with Open-Source Tools
Regular Security Updates and Policy Review
Open-Source Tools for Resource Alerts
Jenkins Pipeline for EC2 and Ansible Configuration
Jenkins EC2 Creation and Deployment Integration
Jenkins Pipeline Error Fix and Update
TDS Calculation for Monthly Salary Breakdown
These are all my environment var
Jenkins Pipeline Code Corrections and Structure
Terraform EC2 Configuration Review and Improvements
Diagnosing Deadlocks in Tomcat/Java Apps
Fix Docker Swarm MongoDB Connection Issue
Companies Offering System Admin Jobs with Sponsorship
Check Image Layers Using Docker Commands
Adding SSH Credentials in Jenkins for File Transfer
Fixing CORS in Nginx Docker Container
Best Platforms for Writing README.md Files
DR Restoration Drill Document Format Request
Identify Storage Type in Windows Server
Modify Python Script for Rate-Limited Requests
setrlimit(rlimit_nofile): Operat
HIPAA vs SOC 2 Compliance Differences
Solving Intermittent CORS Issues on Web Server
Dockerfile Analysis and Security Improvements
Optimizing WildFly Docker Image Setup
Resolving WildFly Module Dependency Issues
Creating Tar File on Remote Server Guide
ModSecurity for NGINX Alternatives and Status
Professional Job Description Reformatting Assistance
Fixing Nginx 404 Error Configuration
Testing Kubernetes Master Node High Availability
Harbor Webhook Integration and Use Cases Explained
Product Key Version Mismatch Fix
Jenkins Pipeline for Kubernetes Deployment Guide
Kubernetes Deployment Pipeline Setup Guide
CI/CD Workflow for Node.js on AWS Docker
Terraform Xen Orchestra VM Provisioning Configuration
Visualize NGINX Logs in Kibana Guide
Ansible Playbook for Kubernetes Cluster Setup
Removing Specific Entry from Linux Login Log
Kubernetes expert insights and solutions

The issue is clear: The summary job has a dependency on uninstall-harbor, but that job was removed from the workflow I provided in the previous response.
The Problem
yaml

summary:
  name: Deployment Summary
  runs-on: self-hosted
  needs: [deploy-harbor, uninstall-harbor]  # ← uninstall-harbor doesn't exist!
  if: always()

The uninstall-harbor job was omitted in the corrected version, but the summary job still references it.
Quick Fix Options
Option 1: Remove the Dependency (Simplest)

Change the summary job to only depend on jobs that exist:
yaml

summary:
  name: Deployment Summary
  runs-on: self-hosted
  needs: [deploy-harbor]  # ← Remove uninstall-harbor
  if: always()
  
  steps:
    - name: Generate Summary
      run: |
        echo "========================================="
        echo "         DEPLOYMENT SUMMARY"
        echo "========================================="
        echo ""
        echo "Workflow Status: ${{ job.status }}"
        echo "Deploy MetalLB: ${{ github.event.inputs.deploy_metallb || 'true' }}"
        echo "Deploy Harbor: ${{ github.event.inputs.deploy_harbor || 'true' }}"
        echo "Harbor Action: ${{ github.event.inputs.harbor_action || 'install' }}"
        echo ""
        echo "Workflow URL:"
        echo "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
        echo "========================================="

Option 2: Add the Missing uninstall-harbor Job

Add this job before the summary job:
yaml

uninstall-harbor:
  name: Uninstall Harbor
  runs-on: self-hosted
  if: github.event.inputs.harbor_action == 'uninstall'
  
  steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Setup Environment
      run: |
        export PATH="$HOME/.local/bin:$PATH"
        echo "$HOME/.local/bin" >> $GITHUB_PATH
        
        # Fetch kubeconfig
        ssh test@$MASTER_IP "sudo cat /etc/kubernetes/admin.conf" > $KUBECONFIG_PATH
        export KUBECONFIG=$KUBECONFIG_PATH
        echo "KUBECONFIG=$KUBECONFIG_PATH" >> $GITHUB_ENV

    - name: Uninstall Harbor
      run: |
        export PATH="$HOME/.local/bin:$PATH"
        export KUBECONFIG=$KUBECONFIG_PATH
        
        echo "=== Uninstalling Harbor ==="
        helm uninstall harbor -n $HARBOR_NAMESPACE || true
        
        echo -e "\n=== Removing PVCs ==="
        kubectl delete pvc --all -n $HARBOR_NAMESPACE || true
        
        echo -e "\n=== Removing PVs ==="
        kubectl delete pv -l app=harbor || true
        
        echo "✅ Harbor uninstalled"

Option 3: Make Summary Independent (Most Flexible)

Make the summary job not depend on any other jobs, but still run regardless of success/failure:
yaml

summary:
  name: Deployment Summary
  runs-on: self-hosted
  if: always()  # ← Always runs, regardless of previous job status
  
  steps:
    - name: Generate Summary
      run: |
        echo "========================================="
        echo "         DEPLOYMENT SUMMARY"
        echo "========================================="
        echo ""
        echo "Deploy MetalLB: ${{ github.event.inputs.deploy_metallb || 'true' }}"
        echo "Deploy Harbor: ${{ github.event.inputs.deploy_harbor || 'true' }}"
        echo "Harbor Action: ${{ github.event.inputs.harbor_action || 'install' }}"
        echo ""
        echo "Workflow URL:"
        echo "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
        echo ""
        echo "Note: Check individual job logs for detailed status"
        echo "========================================="

Complete Fixed Summary Section

Here's the complete corrected end of your workflow file:
yaml

  # Optional: Uninstall job (only if you want uninstall capability)
  uninstall-harbor:
    name: Uninstall Harbor
    runs-on: self-hosted
    if: github.event.inputs.harbor_action == 'uninstall'
    
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Environment
        run: |
          export PATH="$HOME/.local/bin:$PATH"
          echo "$HOME/.local/bin" >> $GITHUB_PATH
          ssh test@$MASTER_IP "sudo cat /etc/kubernetes/admin.conf" > $KUBECONFIG_PATH
          export KUBECONFIG=$KUBECONFIG_PATH
          echo "KUBECONFIG=$KUBECONFIG_PATH" >> $GITHUB_ENV

      - name: Uninstall Harbor
        run: |
          export PATH="$HOME/.local/bin:$PATH"
          export KUBECONFIG=$KUBECONFIG_PATH
          
          echo "=== Uninstalling Harbor ==="
          helm uninstall harbor -n $HARBOR_NAMESPACE || true
          
          echo -e "\n=== Removing PVCs ==="
          kubectl delete pvc --all -n $HARBOR_NAMESPACE || true
          
          echo -e "\n=== Removing PVs ==="
          kubectl delete pv -l app=harbor || true
          
          echo "✅ Harbor uninstalled"

  summary:
    name: Deployment Summary
    runs-on: self-hosted
    needs: [deploy-harbor, uninstall-harbor]
    if: always()
    
    steps:
      - name: Generate Summary
        run: |
          echo "========================================="
          echo "         DEPLOYMENT SUMMARY"
          echo "========================================="
          echo ""
          echo "Deploy Harbor Job: ${{ needs.deploy-harbor.result }}"
          echo "Uninstall Harbor Job: ${{ needs.uninstall-harbor.result }}"
          echo ""
          echo "Deploy MetalLB: ${{ github.event.inputs.deploy_metallb || 'true' }}"
          echo "Deploy Harbor: ${{ github.event.inputs.deploy_harbor || 'true' }}"
          echo "Harbor Action: ${{ github.event.inputs.harbor_action || 'install' }}"
          echo ""
          echo "Workflow URL:"
          echo "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
          echo "========================================="

The Simplest Fix

If you just want to get the workflow working immediately, use this for the summary job:
yaml

summary:
  name: Deployment Summary
  runs-on: self-hosted
  if: always()
  
  steps:
    - name: Generate Summary
      run: |
        echo "========================================="
        echo "         DEPLOYMENT SUMMARY"
        echo "========================================="
        echo ""
        echo "Workflow Status: ${{ job.status }}"
        echo "Deploy MetalLB: ${{ github.event.inputs.deploy_metallb || 'true' }}"
        echo "Deploy Harbor: ${{ github.event.inputs.deploy_harbor || 'true' }}"
        echo "Harbor Action: ${{ github.event.inputs.harbor_action || 'install' }}"
        echo ""
        echo "Workflow URL:"
        echo "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
        echo "========================================="

Choose Option 1 (remove the dependency) for the quickest fix. The workflow will validate and run correctly.
create a readme file for this setup

Here's a comprehensive README file for your complete Kubernetes, MetalLB, and Harbor setup with GitOps automation:

Create README.md in your repository root:
markdown

# 🚢 Harbor Registry on Kubernetes with GitOps

Automated deployment of Harbor private container registry on a bare-metal Kubernetes cluster using GitHub Actions, Ansible, and MetalLB.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Infrastructure](#infrastructure)
- [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)
- [Quick Start](#quick-start)
- [Deployment Workflows](#deployment-workflows)
- [Accessing Harbor](#accessing-harbor)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)

## 🏗 Architecture Overview

┌─────────────────────────────────────────────────────────────────┐
│ GitHub Repository                                               │
│           ↓ Push                                                │
│ GitHub Actions (CI/CD)                                          │
│           ↓ Trigger                                             │
│ Self-Hosted Runner (192.168.21.21)                              │
│ AlmaLinux - Ansible Controller                                  │
│               ↓ SSH                                             |
│ ┌──────────────────+──────────────────┐                         │
│ ↓                  ↓                  ↓                         │
│ Master Node     Worker Node 1     Worker Node 2                 │
│ 192.168.21.120 192.168.21.121 192.168.21.122                    │
│ (Ubuntu)          (Ubuntu)        (Ubuntu)                      │
│                                                                 │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ Kubernetes Cluster                                          │ │
│ │ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐     │ │
│ │ │ MetalLB     │ │   Harbor    │ │      Future Apps    │     │ │
│ │ │ LoadBalancer│ │ Registry    │ │                     │     │ │
│ │ │ 192.168.21. │ │192.168.21.  │ │ 192.168.21.202      │     │ │
│ │ │ 200-210     │ │ 200         │ │                     │     │ │
│ │ └─────────────┘ └─────────────┘ └─────────────────────┘     │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘


## 🖥 Infrastructure

| Hostname | IP Address | Role | OS | User |
|----------|------------|------|-----|------|
| runner.deep.local | 192.168.21.21 | GitHub Runner + Ansible Controller | AlmaLinux
| master.deep.local | 192.168.21.120 | Kubernetes Master | Ubuntu
| worker01.deep.local | 192.168.21.121 | Kubernetes Worker | Ubuntu 
| worker02.deep.local | 192.168.21.122 | Kubernetes Worker | Ubuntu 

### Network Configuration

- **Pod Network CIDR**: 192.168.0.0/16 (Calico)
- **Service CIDR**: 10.96.0.0/12
- **MetalLB IP Pool**: 192.168.21.200-192.168.21.210
- **Harbor Portal IP**: 192.168.21.200
- **Harbor Registry Port**: 5000

