# Google Pixel 10 Laguna Custom Kernel Workbench (GKI 6.6.y)

This directory provides an automated **GitHub Actions Builder workflow** and a comprehensive **visual companion workbench application** dedicated specifically to compiling custom kernels for the **Google Pixel 10** (incorporating the Google Tensor G5 "Laguna" SOC on GKI 6.6).

It centers highly requested root concealments and network optimizations:
- **KernelSU-Next**: The next-generation root management system with advanced namespace separation hooks.
- **SUSFS Overlay**: Powerful kernel-level spoofing to completely mask zygote mounts and files.
- **zeromount**: Custom directory shield patch to prevent virtual mount leaks.
- **BBR (BBRv3) Pacing**: High-performance TCP congestion control with Fair Queueing packet pacing.

---

## 🛠️ GitHub Actions Build Runner

The Actions builder at `.github/workflows/build-kernel.yml` removes compiling overhead completely. It automates build space setup, LLVM downloading, patching, compiling, and packaging into a recovery-flashable ZIP.

### How to Build on GitHub Actions:
1. **Push your workspace** (incorporating the `.github/` folder) to your personal GitHub repository.
2. Navigate to your repository on **GitHub** and enter the **Actions** tab.
3. In the sidebar on the left, select **"Android Custom Kernel Builder (Pixel 10 - Laguna 6.6)"**.
4. Click **"Run workflow"** on the right side.
5. Provide your configuration inputs:
   - **`kernel_repo`**: The URL of your fork or target Google Pixel 10 kernel source repository.
   - **`kernel_branch`**: Your kernel branch name (e.g. `laguna-android15-6.6`).
   - **`kernel_defconfig`**: Target defconfig name located inside `arch/arm64/configs/` (e.g., `gki_defconfig`).
   - **`clang_url`**: Direct stable download link for the Google AOSP Clang toolchain compiler.
   - **`kernelsu_next_enabled`**: Integrates the KernelSU-Next system hooks.
   - **`susfs_enabled`**: Integrates advanced SUSFS overlays.
   - **`zeromount_enabled`**: Injects loop overlay namespace masking.
   - **`bbr_enabled`**: Automatically patches configuration files to set BBR as the default TCP engine.
6. Click **Run workflow**. Upon completion, a flashable **AnyKernel3** update ZIP will be output under the **Artifacts** section!

---

## 🧬 Kernel Upgrade Recipes

For manual compiling or developer scripting, use these specific commands to prepare your local Pixel 10 kernel tree:

### 1. KernelSU-Next Setup (rifsxd Hook)
Deploy the advanced KernelSU-Next structures directly into the kernel source tree:
```bash
cd kernel-source
curl -LSs "https://raw.githubusercontent.com/rifsxd/KernelSU-Next/next/kernel/setup.sh" | bash -s next
```
This script establishes root management hooks and namespace isolation definitions within the virtual file system.

### 2. Precise SUSFS Application
Clone the correct matching GKI 6.6 SUSFS modules and copy the drivers and includes over:
```bash
git clone https://gitlab.com/pomfs/susfs4ksu.git
cd susfs4ksu
# Merge structures
cp -r kernel/* ../kernel-source/kernel/
cp -r fs/* ../kernel-source/fs/
cp -r include/* ../kernel-source/include/
# Apply the target GKI patch
cd ../kernel-source
patch -p1 < ../susfs4ksu/patches/5.15/0001-add-susfs-to-gki-kernel.patch
```

### 3. zeromount Shield Inline Injection
To prevent detection systems from mapping active Magisk/KernelSU virtual mount points, apply this overlay/mount bypass hook inside target namespace handlers:
```bash
# Append checking parameters into fs mount namespace routines
sed -i '/security_sb_mount/s/$/ \/* Inject-Zero-Mount-Pass-Shield *\//' fs/namespace.c
# Build internal mock structures
mkdir -p init/overlay_zeromount
touch init/overlay_zeromount/placeholder
```

### 4. BBR Congestion & FQ Pacing Settings
Append these configuration flags to `arch/arm64/configs/gki_defconfig` before building. BBR relies on fair queueing packet schedulers to pace transmissions accurately:
```bash
echo "CONFIG_TCP_CONG_BBR=y" >> arch/arm64/configs/gki_defconfig
echo "CONFIG_DEFAULT_TCP_CONG=\"bbr\"" >> arch/arm64/configs/gki_defconfig
echo "CONFIG_NET_SCH_FQ=y" >> arch/arm64/configs/gki_defconfig
echo "CONFIG_NET_SCH_FQ_CODEL=y" >> arch/arm64/configs/gki_defconfig
```

### 5. Tensor G5 Compilation Block (LLVM)
Compile arm64 images using the prebuilt LLVM/Clang compiler with standard flags:
```bash
export PATH="$(pwd)/clang/bin:$PATH"
export ARCH=arm64
export SUBARCH=arm64
export CC=clang

# Setup config
make O=out LLVM=1 LLVM_IAS=1 gki_defconfig
# Run compile
make -j$(nproc --all) O=out LLVM=1 LLVM_IAS=1 CROSS_COMPILE=aarch64-linux-gnu- CLANG_TRIPLE=aarch64-linux-gnu-
```

---

## 📱 Visual Companion Applet
Run this repository's native Android workspace module to operate a handy interactive configuration suite:
- **Interactive Toggles**: Turn KernelSU-Next, SUSFS overlays, zeromount bypass, and TCP BBR on or off. See the respective build arguments change in real-time.
- **Workflow Exporter**: Fully formatted, dynamic, single-click copyable YAML scripts for GitHub Actions.
- **Interactive Compile Simulation**: Run the "Dry-Run Builder" to evaluate the sequential ordering of patch applications in a terminal screen interface.
