# Data Analysis Engine

## Python Project Setup Script

## Overview
This Python script automates the setup of a structured data analysis project. It creates a project directory with a predefined structure, sets up a virtual environment, and generates necessary configuration files. This setup is designed to streamline the process of starting new data analysis projects and ensure consistency across projects.

## Features
- **Directory Structure**: Creates a well-organized directory layout for your project, including folders for raw and processed data, notebooks, scripts, and configuration files.
- **YAML Configuration**: Sets up a configuration file (`config.yaml`) to store API-related settings, making it easy to manage and update.
- **File Management**: Includes functions to create files, check for existing content, and delete unlisted files based on a predefined list.
- **Virtual Environment**: Automatically creates and activates a virtual environment for Python dependencies.
- **User Interaction**: Prompts the user for the project name, overwrite options, and whether to install dependencies.

## Requirements
- Python 3.x
- `venv` module (included with Python)
- `pip` for installing dependencies

## Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/data-analysis.git
   cd data-analysis

## Install any required dependencies (if applicable):
```bash
pip install -r requirements.txt
```

## Usage
Run the script:
```bash
python setup_script.py
```

## Follow the prompts to enter the project name, choose whether to overwrite existing projects and decide if you want to install dependencies.

After execution, the project structure will be created in the specified directory.

## Directory Structure
The script creates the following directory structure:
```
data-analysis/
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
├── scripts/
├── config/
└── README.md
```
## Author
Table0

## License
This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments
Inspired by best practices in project organization and automation.
Contributions and feedback are welcome!

---

# Making the Script Executable

## To make your Python script executable, you can follow these steps:

Add a Shebang Line:

At the top of your Python script, add a shebang line to specify the interpreter. This allows the script to be run directly from the command line.
```
#!/usr/bin/env python3
```

## Change File Permissions (Linux/macOS):

After saving your script, you can change the file permissions to make it executable. Open your terminal and run:
```
chmod +x your_script_name.py
```
## Packaging for Distribution:

If you want to distribute your script as a standalone executable, you can use tools like PyInstaller or cx_Freeze to package your Python script into an executable file.
Example with PyInstaller:
```
pip install pyinstaller
pyinstaller --onefile your_script_name.py
```

## This will create a standalone executable in the dist directory.

## Conclusion
By making your script executable, you enhance its usability and make it easier for others to run it without needing to set up a Python environment manually.

## Python Project Setup Script
```
import os
import hashlib
import subprocess
import venv
import json

def create_file(filepath, content="", overwrite=False):
    try:
        with open(filepath, "r") as f:
            if overwrite:
                existing_content = f.read()
                existing_hash = hashlib.sha256(existing_content.encode()).hexdigest()
                new_hash = hashlib.sha256(content.encode()).hexdigest()
                if existing_hash == new_hash:
                    print(f"File '{filepath}' already exists and content is the same. Skipping.")
                    return
    except Exception as e:
        print(f"Error checking file '{filepath}': {e}")
        return

    print(f"Updating or creating file '{filepath}'.")
    with open(filepath, "w") as f:
        f.write(content)

def run_npm_command(project_path, command):
    """Runs an npm command in the specified project directory."""
    try:
        subprocess.run(["npm", command], check=True, cwd=project_path)
    except subprocess.CalledProcessError as e:
        print(f"Error running npm command: {e}")

def create_and_activate_venv(project_path):
    """Creates and activates a virtual environment."""
    venv_dir = os.path.join(project_path, "venv")
    venv.create(venv_dir, with_pip=True)
    print(f"Virtual environment created at {venv_dir}")

    # Activation instructions (platform-specific)
    if os.name == "nt":  # Windows
        activate_script = os.path.join(venv_dir, "Scripts", "activate")
        print(f"Activate venv: {activate_script}")
    else:  # Linux/macOS (Posix)
        activate_script = os.path.join(venv_dir, "bin", "activate")
        print(f"Activate venv: source {activate_script}")

def create_directories(root_dir, folders):
    for folder in folders:
        os.makedirs(os.path.join(root_dir, folder), exist_ok=True)

def create_files(root_dir, files, overwrite=False):
    for filepath, content in files.items():
        create_file(os.path.join(root_dir, filepath), content, overwrite)

def delete_unlisted_files(root_dir, allowed_files):  # Clean up function
    for root, _, files in os.walk(root_dir):
        for file in files:
            filepath = os.path.join(root, file)
            relative_path = os.path.relpath(filepath, root_dir)
            if relative_path not in allowed_files:
                try:
                    os.remove(filepath)
                    print(f"Deleted unlisted file: {relative_path}")
                except OSError as e:
                    print(f"Error deleting file {relative_path}: {e}")

def create_project_structure(project_name, overwrite=False, install_deps=True):
    root_dir = os.path.join(os.getcwd(), project_name)

    create_and_activate_venv(root_dir)  # Create venv

    folders = ["data/raw", "data/processed", "notebooks", "scripts", "config"]
    create_directories(root_dir, folders)

    files = {  # Example files
        "README.md": "# Project Title\n\nProject description goes here.",
        "config/config.yaml": "gemini:\n  api_url: 'https://api.gemini.com/v2/campaigns'\n  access_token: 'YOUR_ACCESS_TOKEN'",
    }
    create_files(root_dir, files, overwrite)

    # Create additional files for data analysis
    for group, filenames in {
        "notebooks": ["data_analysis.ipynb"],
        "scripts": ["fetch_data.py", "process_data.py"],
    }.items():
        for filename in filenames:
            filepath = os.path.join(root_dir, group, filename)
            create_file(filepath, f"# {filename} - Logic here", overwrite)

    try:
        with open('files_to_keep.json', 'r') as f:
            files_to_keep = json.load(f)
    except FileNotFoundError:
        print("Error: files_to_keep.json not found. Using default list.")
        files_to_keep = [  # Default files (update this list)
            "README.md",
            "config/config.yaml",
            "notebooks/data_analysis.ipynb",
            "scripts/fetch_data.py",
            "scripts/process_data.py",
        ]

    delete_unlisted_files(root_dir, files_to_keep)  # Clean up

    if install_deps:
        print("\nRemember to activate your virtual environment and install any necessary dependencies.")

    print(f"\nProject structure created for '{project_name}' in {root_dir}.")

if __name__ == "__main__":
    project_name = input("Enter project name: ")
    overwrite = input("Overwrite existing project? (y/n): ").lower() == "y"
    install_deps = input("Install dependencies? (y/n): ").lower() == "y"
    create_project_structure(project_name, overwrite, install_deps)
```
