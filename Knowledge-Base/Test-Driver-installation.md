# Test-signed driver installation

Remember to be cautious when installing or updating drivers. Improper installation or incompatible drivers can cause system instability. Make sure you have the correct driver for your hardware and back up your data before making any significant changes to your system.

This guide explains how to install a test-signed driver on Windows.

## Step 1: Disable Secure Boot Mode

The Test-Signing Mode is incompatible with Secure Boot Mode. Therefore, you must disable Secure Boot Mode before enabling Test-Signing Mode.
To disable Secure Boot Mode, you need to refer to the QEMU or OpenShift documentation for the specific instructions.

## Step 2: Enable Test-Signing Mode

Before you can install a test-signed driver, you must enable test-signing mode.

1.  **Open Command Prompt as Administrator:**
    - Press the **Windows key**.
    - Type `cmd` into the search bar.
    - Right-click on **Command Prompt** in the search results and select **Run as administrator**.

2.  **Enable Test-Signing:**
    - In the Command Prompt window, type the following command and press **Enter**:
      ```
      bcdedit /set testsigning on
      ```
    - You should see the message: "The operation completed successfully."

3.  **Reboot Your Computer:**
    - For the change to take effect, you must restart your computer. After rebooting, you will see a "Test Mode" watermark on your desktop, which indicates that test-signing mode is active.

## Step 3: Install driver

### Variant 1: using Device Manager

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
   - Select "Browse my computer for drivers" and click "Next".

6. **Specify the Driver Location:**
   Browse to the folder containing the downloaded driver files, or select the appropriate folder where the driver is located. Click "Next" to continue.

7. **Confirm Driver Installation:**
   Windows will analyze the driver package and confirm whether it's compatible with your device. If it is, you'll see a message confirming that the driver will be installed. Click "Install" to proceed.

8. **Confirm driver installation warning:**
   Proceed with the installation prompts. You may see a warning that Windows cannot verify the publisher or that the driver's test signature is not trusted by default. Confirm that you want to install it anyway.

9. **Wait for Installation:**
   Windows will install the driver. This may take a moment, and you may see a progress bar.

10. **Complete the Installation:**
   Once the driver installation is complete, you'll see a message indicating the successful installation. Click "Close" or "Finish" to complete the process.

11. **Verify Installation:**
    After the installation, the device should no longer have an error indicator in Device Manager. It should be listed under the appropriate category without any warnings.

12. **Reboot (if necessary):**
    In some cases, Windows may prompt you to restart your computer to finalize the driver installation. If prompted, save your work and restart your computer.
