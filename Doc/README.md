# User Simulation Script Documentation

## How to Use the Script for a Local Run

1. Copy the folder containing the relevant files into your test system.
2. Modify settings at your convenience:
   - Right-click on `Config.ps1`
   - Select "Open with" and choose "WordPad".
3. Right-click on `RunUserSimulation.ps1` and select "Run with PowerShell".
4. Logs will be stored in the `Logs` folder, named by date and time.

## How to Use the Script for a Multi-Session Run

1. See "Setup the Environment" first.
2. Right-click on `RunUserSimulation.ps1` and select "Run with PowerShell". Accept the UAC prompt.

## Setup the Environment (For Remote/Multi-Users Only)

> If you plan to clone VMs, follow these steps first. If you already have multiple VMs, apply these steps to each one.

### Activate Script Execution

To authorize script execution:

```powershell
Set-ExecutionPolicy -Scope CurrentUser Bypass
```

Answer `A` (yes for all) when prompted.

- Open a PowerShell prompt via `Windows + X` -> "Windows PowerShell".

### Setup Virtual Machines (Network)

1. Connect VMs to a common network.
2. On each VM:
   - Disable the firewall for public networks.
   - Enable network discovery for public networks.
3. Ensure each machine has:
   - A unique name.
   - A unique IP address (use DHCP or configure a static IP).
   - A unique MAC address.
4. Avoid naming PCs as "Local", "All", "First", or "Random" (case insensitive) as these conflict with scheduling features.

#### Setup Network (Static IP Attribution)

In the guest OS:

1. Open Control Panel -> Network & Sharing Center -> Change Adapter Settings.
2. Right-click on the Ethernet adapter and select "Properties".
3. Select IPv4 and click "Properties".
4. Configure the following:
   - IP Address: e.g., `192.168.0.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `192.168.0.1`

#### Setup Network (DHCP with VirtualBox)

On the host OS:

1. Open a shell and navigate to the VirtualBox software directory (e.g., `C:\Program Files (x86)\Oracle\VirtualBox\`).
2. Run the following command:

```cmd
VBoxManage dhcpserver add --netname intnet --ip 10.13.13.100 --netmask 255.255.255.0 --lowerip 10.13.13.101 --upperip 10.13.13.254 --enable
```

### Enable Required Rules (For PowerShell Remoting)

Run the `EnableEverything.ps1` script twice:

1. Right-click on `EnableEverything.ps1`.
2. Select "Run with PowerShell" and accept the UAC prompt.

### Booting the VMs

1. Boot one VM (preferably the one with the lowest static IP if applicable). This will be the master VM.
2. Once the master VM has booted and you have logged in, boot the other VMs.
3. On the master VM, run the `RunUserSimulation` script as an administrator.

### Enable Multi-Session in Configuration File

1. On the master VM:
   - Open the `Config.ps1` file.
   - Locate the `global:global_localonly` option and change its value from `$true` to `$false`.

### Configure the VMs

- Ensure each VM has a copy of the `user simulation script` folder in the same path.
- Global configuration options (e.g., `global_localonly`) apply to all PCs.
- Module configuration options apply only to the local PC (e.g., browsing, mapping shares).

### Notes

- The script can connect to an unlimited number of networked computers. Ensure your hardware can handle the network topology.
- A safe limit is 32 networked PCs.

## Enable the Scheduling Feature

1. Enable `$global_schedule` in the configuration file.
2. Modify the `schedule.txt` file. The first line must always be:

```
Stop All activity on All after 00:01
```

3. Use the following structure for subsequent lines:

```
<Start|Stop> <activity> on <All|Local|First|Random|[PC Name]> after <HH:mm>
```

   - **Start/Stop**: Define the action to perform.
   - **Activity**: Specify the activity (e.g., `All`, `IE`, `MapShare`, `Type`).
   - **On**: Define the target PCs (e.g., `All`, `Local`, specific PC names).
   - **After**: Specify the time to perform the action (24-hour format).

4. Do not repeat the same line in the file.
5. The program may take up to one minute to execute the orders.

## Script Behavior

The script runs in rounds. Each round includes multiple activities (e.g., browsing, mapping network shares). After completing a round, the script waits for an inactivity period specified in the configuration file.

## Run at Startup (Not Currently Working)

1. Create a shortcut to the `RunUserSimulation.ps1` file.
2. Move the shortcut to the startup folder (`Windows + R`, type `shell:startup`).
3. Ensure script execution is activated as described above.

## Important Notes

- This script should run exclusively on the computer to avoid interference with its operations.
- The script forcibly interacts with the environment (e.g., opening/closing applications like Internet Explorer).

You are ready to run the script!