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
| Access           | SSH / WireGuard VPN     |
| Public IP        | `20.244.33.221`         |

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

The NSG was configured according to the principle of **least privilege**. Administrative services such as SSH and Cockpit are not exposed unnecessarily to the public Internet.

The following inbound security rules were configured:

| Priority | Name          |  Port | Protocol | Source                 | Destination | Action |
| -------: | ------------- | ----: | -------- | ---------------------- | ----------- | ------ |
|      300 | SSH-Admin     |    22 | TCP      | `<ADMIN_PUBLIC_IP>/32` | Any         | Allow  |
|      320 | HTTPS         |   443 | TCP      | Any                    | Any         | Allow  |
|      340 | HTTP          |    80 | TCP      | Any                    | Any         | Allow  |
|      350 | WireGuard-VPN | 51820 | UDP      | Any                    | Any         | Allow  |
|      360 | Cockpit-VPN   |  9090 | TCP      | `10.200.200.0/24`      | Any         | Allow  |

### Security considerations

* **SSH – TCP 22:** remote administration of the Ubuntu server. Access is restricted to the administrator's public IP address using a `/32` source range.
* **HTTPS – TCP 443:** allows access to services that require HTTPS.
* **HTTP – TCP 80:** allows HTTP traffic where required, for example HTTP-to-HTTPS redirection or web services.
* **WireGuard – UDP 51820:** provides secure VPN connectivity to the virtualization environment. This port must remain publicly reachable so VPN clients can establish a connection.
* **Cockpit – TCP 9090:** Cockpit is **not exposed to the public Internet**. Access is restricted to the WireGuard VPN subnet `10.200.200.0/24`.
* **UltraVNC:** the public NSG rule was removed. Remote graphical access is no longer directly exposed through the Azure public IP.

The `Source port ranges` for the SSH rule remains **Any**, because the restriction is applied using the source IP address. The destination port is restricted to TCP `22`.

![Azure Network Security](../screenshots/02-azure-network-security.png)

**Figure 2 – Azure Network Security Group inbound rules**

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

Administrative access is separated into two paths:

* **SSH:** restricted to the administrator's public IP address.
* **Cockpit:** accessible through the WireGuard VPN using the private VPN network.
* **WireGuard:** publicly accessible on UDP port `51820` to establish the VPN connection.
* **HTTP/HTTPS:** publicly accessible for services that require web connectivity.

The resulting network access is:

```text
                              Internet
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             SSH TCP/22                  WireGuard UDP/51820
          Admin Public IP                       │
                    │                           │
                    │                           ▼
                    │                    WireGuard VPN
                    │                    10.200.200.0/24
                    │                           │
                    │                  ┌────────┴────────┐
                    │                  │                 │
                    │                  ▼                 ▼
                    │            Cockpit :9090      Other Services
                    │                  │
                    └──────────────────┼─────────────────┐
                                       ▼                 │
                                Ubuntu Server            │
                                       │                 │
                                       ▼                 │
                                  QEMU / KVM             │
                                       │                 │
                                       ▼                 │
                                    EVE-NG               │
```

Cockpit is therefore accessed through the VPN rather than directly through the Azure public IP.

For example, when connected to WireGuard, the Cockpit interface can be accessed using the VPN address:

```text
https://10.200.200.1:9090
```

This approach prevents the Cockpit management interface from being directly exposed to the Internet.

---

## Deployment Result

At this stage, the Azure virtual machine is ready to serve as the virtualization host.

The resulting infrastructure is:

```text
                    Microsoft Azure
                          │
                          ▼
              ┌────────────────────────┐
              │        Azure VM         │
              │                         │
              │ Ubuntu Server 22.04     │
              │ 4 vCPU / 32 GB RAM      │
              │                         │
              │ Nested Virtualization   │
              └────────────┬────────────┘
                           │
                           ▼
                     QEMU / KVM
                           │
                           ▼
                        EVE-NG VM
```

The administrative access architecture is:

```text
                 Internet
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
     SSH TCP/22         WireGuard UDP/51820
   Admin IP only              │
                              ▼
                       VPN 10.200.200.0/24
                              │
                              ▼
                       Cockpit TCP/9090
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
