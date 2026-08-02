# Building Zen Browser

We've taken the time to make building Zen Browser as easy as possible, independent of your operating system or technical knowledge.

---

## Basic Requirements

The following resources are essential for a successful build. Without them, you will encounter unnecessary build failures:

* **Disk Space:** Keep **30GB** of free space on the disk (the build process is resource-intensive).
* **Git** – Required for version control and managing source code.
* **Python 3** – Needed for running build scripts and automation tools.
* **Node.js 21+** – Required for managing dependencies and running JavaScript-based tools.
* **sccache** – A caching tool that speeds up rebuilds by storing compiled objects.
* **Rust and Cargo** – Required to apply Firefox patches.

> [!WARNING]
> If you are using **Windows**, ensure that all basic software requirements are added to your **PATH** variable.
> We cannot provide support if a build fails. Please understand this before proceeding with the following steps.

---

## Windows Configuration: How to Use `sccache` Locally

`sccache` speeds up future rebuilds by caching compiled C++ and Rust code on your local disk. To enable local caching correctly on Windows (and avoid GitHub Actions cloud cache errors):

1. Press `Win + R`, type `sysdm.cpl`, and hit **Enter**.
2. Go to the **Advanced** tab $\rightarrow$ click **Environment Variables...**.
3. Under **User variables**, add/edit the following:

| Variable | Value | Purpose |
| :--- | :--- | :--- |
| **`SCCACHE_DIR`** | `C:\Users\<YourUsername>\.cache\sccache` | Forces `sccache` to use your local disk instead of GitHub cloud cache. |
| **`MOZ_CALL_CACHE`** | `1` | Forces Mozilla's build system (`mach`) to activate `sccache` locally. |

---

## Step 1: Getting Started & Clone the Project

Follow the steps below to set up your environment for building Zen Browser:

1. Install **MozillaBuild** and add it to your `PATH`.
2. Install **7-Zip** and add it to your `PATH`.
3. Ensure you have **Visual Studio** installed with the *"Desktop development with C++"* workload and Windows 10/11 SDK.

Clone the repository locally:

```bash
git clone https://github.com/zen-browser/desktop.git --depth 10

```

> **Note:** `--depth 10` prevents downloading the entire git history, saving bandwidth and time.

---

## Step 2: Install Dependencies

Navigate to the project directory and install the necessary dependencies using `npm`:

```bash
npm i

```

---

## Step 3: Download and Bootstrap the Browser

Prepare the environment and download essential dependencies:

```bash
npm run init

```

> This command handles all bootstrapping tasks. The process may take time and appear inactive at times, but commands are running in the background.

---

## Step 4: Update Language Packs

Update the American English language packs to ensure localization files are current:

```bash
python3 ./scripts/update_en_US_packs.py

```

---

## Step 5: Build the Browser

Compile the source code and build Zen Browser:

```bash
npm run build

```

### Fast Rebuilds (UI Only)

If your changes are strictly within JavaScript or CSS, run:

```bash
npm run build:ui

```

This skips core compilation and rapidly rebuilds the UI components. For C++, Rust, or core engine modifications, always run a full `npm run build`.

---

## Step 6: Run the Browser

Once the build finishes successfully, launch the browser:

```bash
npm start

```

---

## Common Build Errors & Fixes

### Q: `sccache: error: Server startup failed: create gha cache failed`

* **Cause:** `sccache` is trying to access GitHub Actions cache servers on a local machine.
* **Fix:**
1. Remove `SCCACHE_GHA_ENABLED` from your Windows environment variables.
2. Add `SCCACHE_DIR` (e.g., `C:\Users\user\.cache\sccache`) and `MOZ_CALL_CACHE=1`.
3. Kill any running sccache daemon: `taskkill /F /IM sccache.exe`.
4. Restart your terminal and run `npm run build`.



### Q: "mach not found" error?

* Install MozillaBuild, add it to your `PATH`, and restart your terminal.

### Q: "7z" or "7-Zip" missing during build?

* Download 7-Zip, add it to your `PATH`, and restart your terminal.

### Q: "Unsupported Microsoft Visual Studio version" or build failing on Windows?

* Ensure Visual Studio is installed with the **Desktop development with C++** workload and the latest **Windows 10/11 SDK**.

### Q: Build stuck or freezing?

* Restrict the number of CPU threads used for building:
```bash
npm run build -- --jobs 2

```



### Q: "Git submodule" errors after cloning?

* Initialize and update submodules manually:
```bash
git submodule update --init --recursive

```



### Q: "npm run init" fails?

* Manually bootstrap the project:
```bash
npm run bootstrap

```



### Q: "zen.exe" not found after build?

* Perform a clean reset and rebuild:
```bash
npm run reset-ff && npm run init && npm run build

```
