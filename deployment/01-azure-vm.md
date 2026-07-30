# Azure Virtual Machine Deployment

## 1. Azure Virtual Machine

The first step was to deploy a virtual machine in Microsoft Azure to act as the main virtualization host.

The Azure VM runs Ubuntu Server and provides the resources required to run the EVE-NG virtual machine and the network devices inside it.

### 1.1 Azure VM Configuration

The virtualization host was deployed with the following configuration:

| Component        | Configuration           |
| ---------------- | ----------------------- |
| Cloud Provider   | Microsoft Azure         |
| Operating System | Ubuntu Server 22.04 LTS |
| VM Size          | Standard E4as v5        |
| vCPU             | 4                       |
| RAM              | 32 GB                   |
| Architecture     | x86_64                  |
| Virtualization   | Nested virtualization   |
| Access           | SSH                     |
| Public IP        | `20.244.33.221`     |

The Azure VM must support **nested virtualization**, since KVM will be running inside the Azure virtual machine.

---

### 1.2 Create the Azure VM

The virtual machine was created from the **Microsoft Azure Portal**.

The following steps were performed:

1. Open **Virtual Machines**.
2. Select **Create → Azure virtual machine**.
3. Select the required **Subscription**.
4. Create or select a **Resource Group**.
5. Configure the **Virtual Machine name**.
6. Select the required **Azure Region**.
7. Select **Ubuntu Server 22.04 LTS** as the operating system image.
8. Select the **Standard E4as v5** VM size.
9. Configure **SSH authentication**.
10. Allow SSH access on **TCP port 22**.
11. Review the configuration.
12. Create the virtual machine.

After deployment, Azure provides a public IP address that can be used to establish an SSH connection to the Ubuntu server.

![Azure VM overview](../screenshots/01-azure-vm-overview.png)

**Figure 1 – Azure virtual machine overview and configuration**

---

## 2. Network Security Group Configuration

A **Network Security Group (NSG)** was configured to control inbound traffic to the Azure virtual machine.

The following inbound security rules were configured:

| Priority | Name          |  Port | Protocol | Source          | Destination | Action |
| -------: | ------------- | ----: | -------- | --------------- | ----------- | ------ |
|      300 | SSH           |    22 | TCP      | `105.69.199.56` | Any         | Allow  |
|      320 | HTTPS         |   443 | TCP      | Any             | Any         | Allow  |
|      340 | HTTP          |    80 | TCP      | Any             | Any         | Allow  |
|      350 | cockpit-port  |  9090 | TCP      | Any             | Any         | Allow  |
|      370 | wireguard-vpn | 51820 | UDP      | Any             | Any         | Allow  |
|      380 | ultra-vnc     | 32771 | TCP      | Any             | Any         | Allow  |

These rules provide access to the services required by the infrastructure:

* **SSH – TCP 22:** remote administration of the Ubuntu server. Access is restricted to the administrator's public IP address.
* **HTTPS – TCP 443:** HTTPS-based services.
* **HTTP – TCP 80:** HTTP-based services.
* **Cockpit – TCP 9090:** access to the Cockpit web management interface.
* **WireGuard – UDP 51820:** VPN connections to the virtualization environment.
* **UltraVNC – TCP 32771:** remote graphical access for EVE-NG servers when required.

![Azure Network Security](../screenshots/02-azure-network-security.png)

**Figure 2 – Azure Network Security Group inbound rules**

> **Security consideration:** SSH access is restricted to the administrator's public IP address instead of being exposed to all sources. Other services that do not require public access should also be restricted whenever possible.

---

## 3. Connect to the Azure VM

After the deployment is completed, the Ubuntu server can be accessed remotely using SSH.

### 3.1 Linux / macOS

```bash
ssh -i <private-key> <username>@<AZURE_PUBLIC_IP>
```

### 3.2 Windows PowerShell

```powershell
ssh -i C:\Users\<USER>\.ssh\<private-key> <username>@<AZURE_PUBLIC_IP>
```

Where:

* `<private-key>` is the SSH private key used during VM creation.
* `<username>` is the administrator username configured for the VM.
* `<AZURE_PUBLIC_IP>` is the public IP address assigned to the Azure VM.

After successful authentication, the administrator obtains terminal access to the Ubuntu virtualization host.

---

## 4. Update Ubuntu Server

After connecting to the Azure VM, the Ubuntu system was updated before installing the virtualization components.

First, update the package repositories:

```bash
sudo apt update
```

Upgrade the installed packages:

```bash
sudo apt upgrade -y
```

If the system requests a reboot, restart the server:

```bash
sudo reboot
```

After the reboot, reconnect to the server through SSH:

```bash
ssh -i <private-key> <username>@<AZURE_PUBLIC_IP>
```

---

## 5. Verify the Ubuntu Host

After reconnecting to the server, the host configuration was verified using `hostnamectl`.

```bash
hostnamectl
```

This command was used to verify the Ubuntu host information and confirm that the server was correctly deployed before proceeding with the virtualization installation.

![Ubuntu Host](../screenshots/03-ubuntu-host.png)

**Figure 3 – Ubuntu Server host after deployment and initial configuration**

---

## 6. Network Services

The Azure VM serves as the main infrastructure host for several services used in the project.

The resulting network access is:

```text
                         Internet
                            │
                            ▼
                  Azure Public IP
                            │
                            ▼
                Network Security Group
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       SSH :22          Cockpit :9090    WireGuard :51820
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                     Ubuntu Server
                            │
                            ▼
                       QEMU / KVM
                            │
                            ▼
                         EVE-NG
```

The Azure NSG therefore provides the first network-access control layer before traffic reaches the Ubuntu virtualization host.

---

## 7. Deployment Result

At this stage, the Azure virtual machine is ready to serve as the virtualization host.

The resulting infrastructure is:

```text
                    Microsoft Azure
                          │
                          ▼
              ┌───────────────────────┐
              │       Azure VM         │
              │                        │
              │ Ubuntu Server 22.04    │
              │ 4 vCPU / 32 GB RAM     │
              │                        │
              │ Nested Virtualization  │
              └───────────┬────────────┘
                          │
                          ▼
                    QEMU / KVM
                          │
                          ▼
                       EVE-NG VM
```

The Azure VM is now ready for the installation of the virtualization stack.

The next stage is documented in:

```text
deployment/02-qemu-kvm-cockpit.md
```

This stage covers the installation and configuration of:

* QEMU
* KVM
* libvirt
* Cockpit
* EVE-NG virtual machine
