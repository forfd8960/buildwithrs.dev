---
sidebar_position: 2
---

# How to install rust in MacOS, Linux, and Windows

Here are the steps to install Rust on macOS, Linux, and Windows:

## MacOS and Linux

1. **Open Terminal**.
2. **Run the installation script:** Use the following command in your terminal [1]:

    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```

    This command downloads and executes the `rustup` script, which is the official Rust installer [1].
3. **Follow the on-screen instructions:** The script will guide you through the installation process. It will ask you to confirm the installation and configure your environment.
4. **Configure the `PATH` environment variable:**  `rustup` will attempt to configure your `PATH` environment variable during installation. If it doesn't succeed, you may need to manually add the Rust toolchain directory to your `PATH`. The toolchain is typically located in `~/.cargo/bin` [1]. You can modify your shell's configuration file (e.g., `.bashrc`, `.zshrc`) to include the following line:

    ```bash
    export PATH="$HOME/.cargo/bin:$PATH"
    ```

5. **Verify the installation:** After the installation is complete, open a new terminal and run the following command:

    ```bash
    rustc --version
    ```

    This will display the installed Rust version, confirming that Rust is installed correctly and accessible from your terminal [1].

## Windows

1. **Download the installer:** Download the appropriate `rustup-init.exe` installer (32-bit or 64-bit) from the official Rust website [1].
2. **Run the installer:** Execute the downloaded file and follow the on-screen instructions. You might be prompted to install the Visual Studio C++ Build tools [1].
3. **Configure the `PATH` environment variable:**  `rustup` should automatically configure the `PATH` environment variable during installation. If it doesn't, you'll need to add `%USERPROFILE%\.cargo\bin` to your `PATH` manually [1].
4. **Verify the installation:** Open a new command prompt or PowerShell window and run:

    ```bash
    rustc --version
    ```

    This will display the installed Rust version [1].

## Additional Notes

* **Windows Subsystem for Linux (WSL):** If you are using WSL, you can install Rust using the same command as in macOS and Linux [1]:

    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```
* **Other Installation Methods:**  While `rustup` is the preferred method, Rust can also be installed via other methods [1].
* **Homebrew (macOS):** You can also use Homebrew to install Rust on macOS [5]:

    ```bash
    brew install rust
    ```

* **Uninstalling Rust:** To uninstall Rust, you can run `rustup self uninstall` [1].
