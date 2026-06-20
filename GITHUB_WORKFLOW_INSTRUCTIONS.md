# 🚀 GitHub Actions GKI Kernel Runner Quickstart

This is a developer-focused, zero-fluff manual for executing GKI kernel builds (5.10 / 5.15 / 6.1 / 6.6 / Laguna) with custom security patches directly on GitHub's infrastructure.

---

## ⚡ Setup & Workflow Execution

### 1. Repository Push
- Commit and push your local workshop files (ensuring the `.github/workflows/` directory remains intact) to your personal GitHub repository.

### 2. Launching the Build Runner
1. Head to your repository on **GitHub** and click the **Actions** tab.
2. On the left sidebar, click **"Android Custom Kernel Builder"**.
3. Locate and click the **"Run workflow"** button on the right side.
4. Set your target parameter overrides in the form fields:

   | Field Input | Default Reference Example | Notes / Purpose |
   | :--- | :--- | :--- |
   | **`kernel_repo`** | `https://github.com/google/aosp-kernel-gki-laguna` | Repo link to the Android common or SoC tree. |
   | **`kernel_branch`** | `laguna-android15-6.6` | The active branch (e.g. `common-android15-6.6`, `common-android14-6.1`). |
   | **`kernel_defconfig`** | `gki_defconfig` | Target config file inside `arch/arm64/configs/`. |
   | **`clang_url`** | *Pre-filled Google hosting link* | Active stable Clang compiler archive. |
   | **`kernelsu_next_enabled`** | `true` | Embed next-generation KernelSU-Next hooks. |
   | **`susfs_enabled`** | `true` | Inject matching SUSFS vfs and path masks. |
   | **`zeromount_enabled`** | `true` | Hide mount namespaces using inline overlays. |
   | **`bbr_enabled`** | `true` | Change default TCP engine & scheduler to BBR. |

5. Click the green **Run workflow** button.

---

## 📦 Fetching and Flashing the Build Output

1. Track progress inside the execution log. Patch logs will stream real-time.
2. Once complete, scroll to the bottom of the finished workflow run page.
3. Download the zipped asset package under the **Artifacts** segment containing:
   - Your compiled `Image` file.
   - A recovery-flashable **AnyKernel3 Android Update ZIP**.
4. Sideload, push via TWRP/Kernel Flasher, or flash directly using fastboot.
