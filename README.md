# WUBlock
**Take back control of Windows Update. Disable it, enable it, check it — your call.**

Tired of Microsoft force-rebooting your PC mid-work? WUBlock is a professional-grade Batch utility that lets you fully disable or re-enable Windows Updates whenever you want. It features a built-in safety diagnostic engine so you always know exactly which defense layers are active.

---

##  What it does
WUBlock doesn't just "pause" updates; it creates a multi-layered barrier that prevents Windows from repairing its own update services:

* **Disable (Hard Block):** Kills Windows Update from 5 critical angles:
    1.  **Services:** Stops and disables `wuauserv`, `UsoSvc`, `bits`, and `dosvc`.
    2.  **Medic Lock:** Hard-locks `WaaSMedicSvc` via the registry to prevent Windows from "self-repairing" the update block.
    3.  **Group Policy:** Applies Registry-level GPO blocks to `NoAutoUpdate`.
    4.  **Network Routing:** Null-routes update server domains in the local `hosts` file.
    5.  **Task Scheduler:** Disables the Update Orchestrator's background scan tasks.
* **Enable (Restore):** Cleanly reverses every single change, restoring services to their default "Automatic" state and cleaning the `hosts` file.
* **Safety Check:** Runs a real-time diagnostic report on your Services, Registry, Network, and Medic-status.

---

##  How to use
1.  **Download** `WUBlock.bat`.
2.  **Right-click** the file and choose **"Run as administrator"** (required to modify system settings).
3.  **Select** an option from the stylized menu.

---

##  Menu Options
* **[1] DISABLE ALL UPDATES:** Full system lockdown.
* **[2] ENABLE ALL UPDATES:** Complete factory restoration of update services.
* **[3] RUN SAFETY CHECK:** View the Diagnostic Safety Report.
* **[4] EXIT:** Close the utility.

---

##  Security Report Breakdown
| Layer | Target | Effect |
| :--- | :--- | :--- |
| **Service** | `wuauserv` | Stops the core update engine. |
| **Registry** | `NoAutoUpdate` | Forces Group Policy to ignore updates. |
| **Network** | `hosts` file | Blocks connection to Microsoft Update servers. |
| **Medic** | `WaaSMedicSvc` | Prevents Windows from automatically re-enabling the engine. |

---

## ⚠️ Requirements & Heads Up
* **OS:** Windows 10 or Windows 11.
* **Privileges:** Must be run as **Administrator**.
* **Security:** Blocking updates means you will not receive security patches. It is recommended to run **Option [2]** once a month to manually check for and install critical security updates.

