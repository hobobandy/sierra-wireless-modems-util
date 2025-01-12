# Sierra Wireless Models Utility Script

Interact with Sierra Wireless modems, specifically AirPrime EM74xx/MC74xx models, using AT commands.

*Under development, until then, here's additional instructions.*

## Modems Tested

* [MC7455](https://source.sierrawireless.com/resources/airprime/minicard/74xx/airprime_mc7455_product_technical_specification/)

## Auto-start GPS without a SIM card

1. Stop and disable ModemManager to prevent conflicts:  
   `sudo systemctl stop ModemManager && sudo systemctl disable ModemManager`
2. Start ModemManager in debug mode to monitor responses:  
   `sudo /usr/sbin/ModemManager --debug`
3. Send the following commands - adjust the index as required, wait for an OK after each command: 

    * `minicom` can also be used to send the commands.
    
    ```bash
    sudo mmcli -m 0 --command="ATI"
    sudo mmcli -m 0 --command="A710"
    sudo mmcli -m 0 --command='AT!ENTERCND="A710"'
    sudo mmcli -m 0 --command='AT!GPSAUTOSTART=1,1,255,100,1'
    sudo mmcli -m 0 --command='AT!GPSNMEACONFIG=1,1'
    sudo mmcli -m 0 --command='AT+WANT=1'
    sudo mmcli -m 0 --command='AT!CUSTOM="GPSSEL",0'
    sudo mmcli -m 0 --command='AT!CUSTOM="GPSLPM",0'
    sudo mmcli -m 0 --command='AT!RESET'
    ```

### Command breakdown

* `AT!ENTERCND="A710"` - Enable access to password-protected commands
  * `A710` is the default code for MC7455 (maybe others)
* `AT!GPSAUTOSTART=1,1,255,100,1` - Configure GPS auto-start
  * Enable at boot
  * Standalone fix (no MS assistance, use GPS only)
  * Max time to wait for fix (`255` secs)
  * Accuracy of fix (within `100` meters)
  * Time between fixes (`1` sec)
* `AT!GPSNMEACONFIG=1,1` - Configure NMEA sentences
  * Enable NMEA data output
  * Output rate of `1` sec
* `AT+WANT=1` - Enable antenna power (3.3V) for active GPS antennae
* `AT!CUSTOM="GPSSEL",0` - Ensure dedicated GPS antenna port is used (default)
* `AT!CUSTOM="GPSLPM",0` - GPS remains enabled when modem enters low power mode (e.g, no SIM)
* `AT!RESET` - Reset modem to apply new configuration

## Firmware Upgrade

Reference: [Software Integration and Development Guide for Linux](https://source.sierrawireless.com/resources/airprime/software/mbpl/mbpl-software-latest/)

### Flash Tool Download

1. Download the Sierra Wireless's Mobile Broadband Package SDK Lite for Linux:  
   * [MBPL_SDK_Rxx_ENGx-lite.bin.tar](https://source.sierrawireless.com/resources/airprime/software/mbpl/mbpl-software-latest/)
2. Extract the `lite-fw-download` tool specifically - adapt for your platform:  
   `tar --extract --strip-components 3 --file MBPL_SDK_R42_ENG4-lite.bin.tar.gz SampleApps/fw-download-tool/bin/fw-download-toolhostx86_64`

### Firmware Flashing

1. Stop and disable ModemManager to prevent conflicts:  
   `sudo systemctl stop ModemManager && sudo systemctl disable ModemManager`
2. Reboot (may be optional)
3. Download your modem's approved firwmare package:
   * [EM/MC74xx](https://source.sierrawireless.com/resources/airprime/minicard/74xx/em_mc74xx-approved-fw-packages/)
4. Extract the files in an images directory:  
   `unzip SWI9X30C_02.39.00.00_GENERIC_002.085_000.zip -d images/`
5. Run the tool as superuser, the simple one meets requirements - this may take a while:  
   `sudo ./fw-download-toolhostx86_64 -f images/`
6. Monitor the progress by tailing the log file:  
   `tail -f fwdwl.log`
7. Confirm firmware version using `minicom`:
   1. Send the command `AT+GMR`, the response should be:  
   `SWI9X30C_02.39.00.00 rF194F7CA76D79E jenkins 2024/06/05 05:36:47`
8. Start and enable ModemManager - if needed:  
   `sudo systemctl start ModemManager && sudo systemctl enable ModemManager`

## Development Notes

### minicom

1. Install minicom for serial interactions:  
   `sudo apt install minicom`
2. Configure minicom on first run, or if serial port changes:
   1. `sudo minicom -s`
   2. Use the down-arrow key to go to `Serial port setup`
   3. Type `a` to change the `Serial Device` to `/dev/ttyUSB3` - or whatever your AT file handle is
   4. Return to the main menu and go to `Screen and keyboard`
   5. Type `q` to enable `Local echo` so you can see what you're typing!
   6. Return to the main menu and save your config as default by selecting `Save setup as dfl`
   7. Exit by selecting `Exit from Minicom`

* To quit `minicom`, the hotkey sequence is `ctrl-a > q > Enter`

### Monitoring Serial Messages

1. Stop ModemManager:  
   `sudo systemctl stop ModemManager`
2. Start ModemManager in debug:  
   `sudo /usr/sbin/ModemManager --debug`

## Credit

* Thanks to [daniele wood's sierra-wireless-models project](https://github.com/danielewood/sierra-wireless-modems/) for references and a great start point.
