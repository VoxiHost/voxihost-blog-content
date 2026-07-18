---
image: /assets/images/blog/en/install-python-venv-ubuntu/og-image.png
title: "How to Setup Python Venv on Ubuntu"
description: "Learn how to configure isolated Python virtual environments on Ubuntu using the native venv module. Avoid dependency conflicts and manage your projects."
status: draft
category: Tutorials
tags:
  - python
  - ubuntu
  - linux
  - server
  - vps
date: '2026-07-24'
locale: en
translationKey: install-python-venv-ubuntu
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: []
howto:
  name: "How to Setup Python Venv on Ubuntu"
  steps:
    - name: "Step 1: Install the Python Venv Module"
      url: "step-1-install-the-python-venv-module"
    - name: "Step 2: Create a Project Directory"
      url: "step-2-create-a-project-directory"
    - name: "Step 3: Create and Activate the Virtual Environment"
      url: "step-3-create-and-activate-the-virtual-environment"
    - name: "Step 4: Manage Packages Within the Environment"
      url: "step-4-manage-packages-within-the-environment"
faq:
  - question: "Why does Ubuntu block global pip installs via PEP 668?"
    answer: "Ubuntu enforces <strong>PEP 668</strong> to prevent <code>pip</code> from overwriting system-managed packages, which could break critical OS services on your VoxiHost VPS."
  - question: "Do I need sudo to manage my virtual environment?"
    answer: "No. You should never use <code>sudo</code> when creating or managing a virtual environment, as it changes file ownership and causes permission errors."
  - question: "How can I tell if my virtual environment is currently active?"
    answer: "When active, your shell prompt will typically be prefixed with the name of your environment directory, such as <code>(venv) user@hostname:~$</code>."
  - question: "What should I do if pip is missing after creating the venv?"
    answer: "If <code>pip</code> is not available, run <code>python3 -m ensurepip --upgrade</code> to install it directly into your active virtual environment."
  - question: "How do I deactivate the virtual environment when I am finished?"
    answer: "Simply run the <code>deactivate</code> command in your terminal to return your shell to the system default Python environment."
---

Managing Python dependencies on a production server often leads to conflicts. When you install packages globally using `pip`, you risk breaking system-level tools that rely on specific library versions. On modern Ubuntu distributions, this issue is further addressed by PEP 668, which restricts global package installations to ensure system stability.

The industry-standard solution is the virtual environment, or `venv`. By isolating your project dependencies into a local directory, you ensure that your application has exactly what it needs without interfering with the underlying operating system or other projects hosted on your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/). 

This guide provides a direct, no-nonsense approach to setting up isolated Python environments. We will walk through installing the necessary module, creating a clean workspace, and activating your environment to start installing packages safely. Whether you are running a lightweight microservice on a [Budget VPS](/budget-vps/) or managing complex data pipelines, this setup is the foundation of professional Python development on Linux.

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H1.png", "A terminal window showing a successful Python virtual environment activation on an Ubuntu server", "(max-width: 768px) 100vw, 800px" %}

## Prerequisites

Before beginning, ensure your server meets the minimum requirements for a stable development environment. We recommend a minimum of 512MB of RAM and 1 CPU core, which is standard for our [Budget VPS](/budget-vps/) and [Premium VPS](/premium-vps/) instances. 

You should have access to an Ubuntu 22.04 LTS or newer server with a non-root user that has `sudo` privileges. If you have not yet set up your administrative user, refer to our guide on [How to Create a Sudo User on Ubuntu & Debian: The Complete Server Guide](/add-sudo-user-ubuntu/) to ensure you are not running development tasks as the root user. 

Additionally, confirm that your system clock is synchronized to avoid SSL errors when fetching packages. You should also ensure that your APT package list is current to avoid dependency resolution issues. While no specific Python code is required at this stage, having basic familiarity with the command line is expected. 

Finally, check that you have enough disk space available in your project directory to accommodate the virtual environment structure, which typically consumes a few megabytes for the core files plus the size of any project-specific dependencies you intend to install later.

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H2.png", "A terminal session displaying system resource checks and user validation before starting the Python venv installation", "(max-width: 768px) 100vw, 800px" %}

## Step 1: Install the Python Venv Module

Modern versions of Ubuntu include Python by default, but the module required for creating isolated environments is often excluded from the core installation to save space. To manage your project dependencies without polluting the global system packages, you must install the `python3-venv` package. This ensures you comply with PEP 668, which restricts global pip installations on newer Ubuntu releases to prevent conflicts with system-level software.

Execute the following commands to update your local package index and pull the necessary module into your system:

```bash
## Update the package index and install the virtual environment module
sudo apt update
sudo apt install -y python3-venv
```

This installation provides the `venv` module, which allows you to create lightweight, isolated Python environments. Once the process finishes, verify the installation by checking the Python version. While this doesn't explicitly confirm the module's presence, it ensures your environment is ready for the next steps:

```bash
## Verify Python installation
python3 --version
```

If you are using a <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/premium-vps/), this operation will complete in seconds. You are now prepared to initialize your first virtual environment within a dedicated project directory.

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H3.png", "Terminal output showing the successful installation of the python3-venv package via apt", "(max-width: 768px) 100vw, 800px" %}

## Step 2: Create a Project Directory

Before you initialize your environment, you need a clean workspace to host your application code and its dependencies. Keeping your projects in separate directories prevents file clutter and makes it easier to manage multiple environments on your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> server. 

Navigate to your home directory or a dedicated folder for your development projects, then create a new directory for your specific task:

```bash
## Create a new project directory and move into it
mkdir -p ~/my_python_project
cd ~/my_python_project
```

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H4.png", "Terminal showing the creation of the project directory", "(max-width: 768px) 100vw, 800px" %}

## Step 3: Create and Activate the Virtual Environment

With the project directory ready, you can now initialize the virtual environment. We will use the `venv` module to generate a local directory named `venv`. This folder will contain a standalone Python binary and its own `pip` installer, effectively sandboxing your project from the rest of the operating system.

```bash
## Initialize the virtual environment in the current folder
python3 -m venv venv
```

After running this command, you will notice a new `venv` directory in your current path. This directory holds everything needed to run your project without requiring root privileges. Do not use `sudo` for this step or any subsequent package management within this environment, as it can cause ownership issues that break your project configuration. Next, activate the environment to start using the local binaries:

```bash
## Activate the virtual environment
source venv/bin/activate
```

Once activated, you will notice that your terminal prompt changes to include `(venv)` at the beginning. This visual indicator confirms that any subsequent commands, such as `python` or `pip`, are now pointing to the binaries located inside your project folder rather than the system defaults.

> **Note:** If you find that `pip` is not available after activation, you can ensure it is installed by running `python3 -m ensurepip --upgrade`. This will safely install the package manager directly into your virtual environment without affecting your host system.

At this point, your <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> server is correctly configured for local development. You can now safely install project-specific libraries using `pip install <package_name>`. Because you are using a virtual environment, these packages will remain contained within your `~/my_python_project/venv` directory, keeping your system clean and avoiding conflicts with other applications.

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H5.png", "Terminal showing the activated virtual environment with (venv) prefix in the command prompt", "(max-width: 768px) 100vw, 800px" %}

## Step 4: Manage Packages Within the Environment

Now that your environment is active, you can manage project dependencies without requiring root access. The primary tool for this is `pip`, the standard package installer for Python. Because your shell is currently pointed at the isolated `venv` binaries, any library you install will be saved exclusively to your project folder.

To install a package, such as `requests`, simply run:

```bash
## Install a package within the virtual environment
pip install requests
```

You can verify that the library is installed correctly by listing the currently available packages in your environment:

```bash
## List all installed packages in the current venv
pip list
```

If you are working on a collaborative project or moving your code to a production <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> server, you should generate a requirements file. This allows you to track exactly which versions of libraries your project needs:

```bash
## Export the current environment dependencies to a file
pip freeze > requirements.txt
```

This `requirements.txt` file is the industry standard for maintaining consistent environments across different machines. If you ever need to recreate this environment on another server, you can install all dependencies at once using `pip install -r requirements.txt`. 

> **Warning:** Never use `sudo` when running `pip` inside your virtual environment. Using `sudo` can cause file permission errors and may inadvertently install packages to the system-wide Python directory, which violates the isolation you have just established.

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H6.png", "Terminal showing pip install output and a generated requirements.txt file within the active venv", "(max-width: 768px) 100vw, 800px" %}

## Conclusion

You have successfully established a robust, isolated Python environment on your system. By moving away from global package management, you have eliminated the risk of dependency conflicts and ensured that your projects remain portable and stable. This workflow is particularly vital when deploying applications on your [Premium VPS](/premium-vps/) or [Budget VPS](/budget-vps/), where maintaining a clean system state is essential for long-term server health.

Remember that virtual environments are ephemeral. If you ever need to update your dependencies, simply reactivate the environment using the `source venv/bin/activate` command and run your updates. If you find your project requirements have changed significantly, it is often cleaner to delete the `venv` directory and recreate it from your `requirements.txt` file rather than manually uninstalling dozens of individual packages.

As your projects grow, you might consider containerizing them with Docker or implementing automated deployment scripts to handle these environment setups. For now, you have a solid foundation for local development and production deployment. Keep your server secure, keep your dependencies pinned, and continue building on your infrastructure with confidence.

{% image "/assets/images/blog/en/install-python-venv-ubuntu/H7.png", "A terminal screen showing a deactivated virtual environment and a return to the standard user shell", "(max-width: 768px) 100vw, 800px" %}
