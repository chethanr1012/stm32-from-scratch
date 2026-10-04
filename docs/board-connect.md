# Connecting the board (Windows + WSL)

The NUCLEO-F401RE is plugged into Windows, but the tools run in Ubuntu (WSL).
The ST-LINK must be passed through to WSL each time with usbipd-win.

## Every time you plug in the board or restart WSL

**1. Make sure WSL is running.** Having VS Code open in WSL, or any Ubuntu
terminal open, is enough.

**2. In Windows PowerShell** (normal window, no admin needed):

```powershell
usbipd list
```

Find the ST-Link line (VID:PID `0483:374b`) and note its BUSID, for example `2-3`.

```powershell
usbipd attach --wsl --busid <BUSID>
```

**3. In the Ubuntu terminal**, confirm it arrived:

```bash
lsusb
```

Expected line: `ID 0483:374b STMicroelectronics ST-LINK/V2.1`

**4. Test the debug connection** (optional):

```bash
openocd -f board/st_nucleo_f4.cfg
```

Expected: `stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints` and
`Listening on port 3333 for gdb connections`. Stop it with Ctrl+C.

## My BUSID

BUSID on this laptop: `______` (fill in from `usbipd list`)

The BUSID stays the same as long as you use the same USB port. A different
port gives a different BUSID.

## One-time setup (already done)

Only needed again on a new PC, or if the BUSID changes and `attach` complains
the device is not shared. Run in PowerShell **as Administrator**:

```powershell
usbipd bind --busid <BUSID>
```

## When you are done

Unplugging the board is enough. To give the board back to Windows without
unplugging:

```powershell
usbipd detach --busid <BUSID>
```

## Troubleshooting

| Symptom | Fix |
|---|---|
| `usbipd` not recognized | Open a new PowerShell window. If still missing, reinstall usbipd-win from github.com/dorssel/usbipd-win/releases |
| `attach` says device is not shared | Run `usbipd bind --busid <BUSID>` in an Administrator PowerShell, then attach again |
| `attach` says no WSL distribution is running | Open an Ubuntu terminal or VS Code in WSL first, then attach again |
| `lsusb` does not show the ST-LINK | Run `usbipd list` and check the state says Attached. If not, attach again |
| ST-Link missing from `usbipd list` | Check the USB cable (some are charge-only) and that LD1 on the board is lit |
| OpenOCD: `LIBUSB_ERROR_ACCESS` | Run `sudo openocd -f board/st_nucleo_f4.cfg` |
| OpenOCD: `Error: open failed` | Board not attached to WSL. Repeat steps 2 and 3 |
| Board works in WSL but Windows tools cannot see it | Expected while attached. Run the detach command |

## Notes

- While attached to WSL, the board's serial port appears in Ubuntu as
  `/dev/ttyACM0`, not as a COM port in Windows.
- Attach does not survive a replug, a WSL restart, or a Windows reboot.
