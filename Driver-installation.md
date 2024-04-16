# Driver installation

Remember to be cautious when installing or updating drivers, as improper installation or incompatible drivers can cause system instability. Make sure you have the correct driver for your hardware, and back up your data before making any significant changes to your system.

## Variant 1: using the Installation Wizard

This is the best option for regular users that just want to install the drivers on their VM

1. **Download Virtio-Win ISO:**
   - Visit the official Virtio-Win repository (https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso) and download the latest Virtio-Win ISO file to your host machine.

2. **Attach the Virtio-Win ISO to the VM:**
   - Ensure your VM is powered on and running.
   - In the VM management software (e.g., VirtualBox, VMware, QEMU, etc.), locate the option to attach the Virtio-Win ISO as a virtual CD/DVD drive to the VM. The specific steps may vary depending on the virtualization software you're using.

3. **Inside the VM:**
   - Access the VM's desktop or file system.

4. **Open File Explorer:**
   - Open File Explorer (Windows Explorer) within the VM.

5. **Navigate to the Virtio-Win ISO:**
   - In File Explorer, locate and click on the virtual CD/DVD drive where the Virtio-Win ISO is mounted. It should be listed under "This PC" or "Computer."

6. **Launch the Installation Wizard:**
   - Inside the Virtio-Win ISO, locate the `virtio-win-guest-tools-xxx.exe` (where "xxx" represents the version number) file and run it by double-clicking it. This file contains the Virtio-Win drivers and the Installation Wizard.

7. **Select Components:**
   - The Virtio-Win Installation Wizard will launch. It will prompt you to select the components you want to install, such as network drivers, storage drivers, and balloon drivers. You can choose to install all available drivers by default or select specific components based on your requirements. Click "Install" to proceed.

8. **Driver Installation:**
   - The Installation Wizard will begin to install the selected drivers. You will see progress bars for each driver component being installed.

9. **Reboot the VM (if prompted):**
   - After the drivers are installed, the Installation Wizard may prompt you to reboot the VM for the changes to take effect. If prompted, save your work and restart the VM.

10. **Verify Driver Installation:**
    - After the VM restarts, check Device Manager to ensure that the Virtio-Win drivers are correctly installed. There should be no unknown devices or driver-related errors.

## Installing drivers required for Windows installation, such as hard disk drivers

1. **Prepare Driver Files:**
   - Before you begin the Windows installation, you'll need to have the necessary driver files for your hardware component ready. This usually includes the hard disk or storage controller drivers. Download these drivers from the manufacturer's website.

2. **Insert Windows Installation Media:**
   - Insert the Windows installation media (DVD or USB) into your computer and boot from it. You may need to access the BIOS/UEFI settings to set the boot order to prioritize the installation media.

3. **Start the Windows Installation:**
   - Power on or restart your computer to initiate the Windows installation from the installation media. If it doesn't start automatically, you may need to press a key (e.g., F2, F12, Delete) to access the boot menu and select the installation media.

4. **Language and Region Settings:**
   - In the initial Windows setup screen, select your preferred language, time format, keyboard layout, and click "Next."

5. **Click "Install Now":**
   - On the next screen, click "Install Now" to start the Windows installation.

6. **Accept License Terms:**
   - Read and accept the Microsoft Software License Terms.

7. **Choose Installation Type:**
   - Select "Custom: Install Windows only (advanced)" as you will be specifying driver installation manually.

8. **Partitioning and Drive Selection:**
   - In the "Where do you want to install Windows?" window, you may not see your hard disk or storage device listed if it's not recognized. Click "Load Driver."

9. **Load Drivers:**
   - The "Load Driver" window will open. Insert the USB flash drive or attach the Virtio-Win ISO containing the driver files you prepared earlier.

10. **Browse for Drivers:**
    - Click "Browse" and navigate to the USB flash drive to find the driver files you downloaded. Select the appropriate driver and click "Next."

11. **Install the Driver:**
    - Windows will load the driver. Once the driver is loaded successfully, you should see your hard disk or storage device listed in the installation window.

12. **Select the Drive:**
    - Choose your hard disk or storage device and click "Next."

13. **Complete Windows Setup:**
    - Continue with the Windows setup, which includes creating or signing in with a Microsoft account, setting up your preferences, and configuring your system settings.

By following these steps, you can successfully install the required drivers, such as hard disk drivers, during the Windows installation process. This ensures that your storage devices are recognized, allowing you to proceed with the installation and use your computer as intended.

## Variant 2: using Device Manager

1. **Plug in the Device:** First, make sure the device for which you want to install the driver is connected to your computer. It should be recognized as an unknown device or have an error indicator in Device Manager.

2. **Open Device Manager:**
   - Press the Windows key.
   - Type "Device Manager" into the search bar.
   - Click on "Device Manager" in the search results.

3. **Locate the Device:**
   In Device Manager, find and expand the category that corresponds to the device you want to install a driver for. The device may appear under "Other devices" with a yellow triangle icon, indicating a problem with the driver.

4. **Install the Driver:**
   Right-click on the device with the missing driver and select "Update driver."

5. **Choose How to Search for Drivers:**
   - Select "Search automatically for updated driver software" if you want Windows to search for the driver online. This option requires an active internet connection.
   - Select "Browse my computer for drivers" if you have already downloaded the driver and have it saved on your computer. Then, click "Next."

6. **Specify the Driver Location (if needed):**
   If you selected "Browse my computer for drivers," browse to the folder containing the downloaded driver files, or select the appropriate folder where the driver is located. Click "Next" to continue.

7. **Confirm Driver Installation:**
   Windows will analyze the driver package and confirm whether it's compatible with your device. If it is, you'll see a message confirming that the driver will be installed. Click "Install" to proceed.

8. **Wait for Installation:** 
   Windows will install the driver. This may take a moment, and you may see a progress bar.

9. **Complete the Installation:** 
   Once the driver installation is complete, you'll see a message indicating the successful installation. Click "Close" or "Finish" to complete the process.

10. **Verify Installation:** 
    After the installation, the device should no longer have an error indicator in Device Manager. It should be listed under the appropriate category without any warnings.

11. **Reboot (if necessary):** 
    In some cases, Windows may prompt you to restart your computer to finalize the driver installation. If prompted, save your work and restart your computer.

## Variant 3: via an INF (Information) file 

1. **Download the Correct INF File:**
   - Visit the manufacturer's website or a trusted source to download the correct INF file for the driver that matches your hardware device and Windows version. Ensure that you have the latest version of the driver if available.

2. **Locate the Downloaded INF File:**
   - Once the INF file is downloaded, locate it in your computer's file system. It is usually in the Downloads folder unless you specified a different location during the download.

3. **Right-Click on the INF File:**
   - Right-click on the downloaded INF file.

4. **Select "Install" from the Context Menu:**
   - In the context menu that appears when you right-click the INF file, choose "Install." This action will initiate the installation process.

5. **Driver Installation Process:**
   - Windows will begin the installation process for the driver based on the information provided in the INF file. During this process, you may see a series of dialogs or prompts, depending on the driver. These prompts may ask for confirmation to install the driver.

6. **Follow On-Screen Prompts:**
   - Follow any on-screen prompts or instructions provided during the installation. These may include agreeing to the driver's terms and conditions, confirming the installation, or selecting installation options.

7. **Complete the Installation:**
   - Once the installation is complete, you'll receive a message indicating the successful installation of the driver.

8. **Reboot (if necessary):**
   - In some cases, Windows may prompt you to restart your computer to finalize the driver installation. If prompted, save your work and restart your computer.

9. **Verify Installation:**
   - After installation, check Device Manager to ensure the driver is listed under the appropriate category without any warnings or errors. If the device is listed correctly, the driver installation was successful.


## Variant 4: using the `pnputil` command line utility

1. Open a Command Prompt with administrative privileges:
   - Press the Windows key.
   - Type "cmd" or "Command Prompt" into the search bar.
   - Right-click on "Command Prompt" and select "Run as administrator."

1. Type the following command to list all the installed drivers and their package names:
   ```bash
   pnputil /enum-drivers
   ```
   
   This will provide a list of installed drivers along with their published names, which you will need to identify the correct driver package.

1. Locate the INF file for the driver you want to install. INF files are typically located in the driver package folder or on the manufacturer's website.

1. Install the driver using the `pnputil` utility. Replace `path\to\driver.inf` with the actual path to the INF file of the driver you want to install:
   ```bash
   pnputil /add-driver "path\to\driver.inf"
   ```
   
   If you want to specify a package name (use the published name from step 2), you can do so with the `/package` option:
   ```bash
   pnputil /add-driver "path\to\driver.inf" /package PackageName
   ```
   
   If the driver package contains multiple INF files, you should specify the INF file that corresponds to the specific hardware component you want to update.

1. Windows may prompt you to confirm the installation of the driver. Confirm and proceed with the installation if prompted.

1. After a successful installation, you should see a message indicating that the driver has been added.

1. You may need to restart your computer for the changes to take effect. Use the following command to check the status of the driver installation:
   ```bash
   pnputil /e
   ```

   This command will show a list of installed driver packages.
