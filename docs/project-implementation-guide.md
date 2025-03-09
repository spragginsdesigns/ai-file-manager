# AI File Organizer: Project Implementation Guide

This document provides a comprehensive guide for implementing the AI File Organizer project, focusing on project structure, functionality, and development phases.

## Project Overview

The AI File Organizer uses Claude 3.7 Sonnet to intelligently analyze and organize files based on their types, content, and relationships. The application provides both CLI and GUI interfaces, allowing users to organize their files using AI-powered suggestions.

### Key Capabilities

- Directory content analysis
- AI-powered organization suggestions
- Automatic file categorization
- Both CLI and GUI interfaces
- Content-based file grouping

## Project Structure

```
ai-file-organizer/
├── .env                  # For API key
├── README.md
├── requirements.txt
└── src/
    ├── core/             # Core functionality
    │   ├── claude_api.py # Claude API integration
    │   ├── file_ops.py   # File operations
    │   └── organizer.py  # Organization logic
    ├── cli/              # Command-line interface
    │   └── cli_app.py
    ├── gui/              # Graphical interface
    │   └── gui_app.py
    └── main.py           # Entry point
```

## Development Phases

### Phase 1: Environment Setup and Core Functionality

#### Environment Setup

1. Create a virtual environment and install dependencies:

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install anthropic pysimplegui python-dotenv
```

2. Create a `.env` file for storing the Anthropic API key:

```
ANTHROPIC_API_KEY=your_api_key_here
```

#### Core Functionality Implementation

1. Create the Claude API integration module (`src/core/claude_api.py`):

```python
import os
from anthropic import Anthropic
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

class ClaudeClient:
    """Client for interacting with Claude API"""

    def __init__(self, api_key=None):
        """Initialize Claude client with API key"""
        # Use provided API key or load from environment
        self.api_key = api_key or os.getenv("ANTHROPIC_API_KEY")
        if not self.api_key:
            raise ValueError("Anthropic API key is required")

        # Initialize client
        self.client = Anthropic(api_key=self.api_key)

        # Default model
        self.model = "claude-3-7-sonnet-20250219"

        # Default thinking mode settings
        self.thinking_enabled = True
        self.thinking_budget = 1000

    def set_model(self, model):
        """Set Claude model to use"""
        self.model = model

    def set_thinking_mode(self, enabled, budget=1000):
        """Configure thinking mode"""
        self.thinking_enabled = enabled
        self.thinking_budget = budget

    def analyze_directory(self, files_info):
        """
        Get Claude's analysis of directory contents

        Args:
            files_info: List of dictionaries containing file metadata

        Returns:
            Claude's analysis and organization suggestions
        """
        # Configure thinking mode
        thinking = None
        if self.thinking_enabled:
            thinking = {
                "type": "enabled",
                "budget_tokens": self.thinking_budget
            }

        # Create prompt for Claude
        prompt = f"""
        Analyze these files and suggest an organization strategy.
        Provide a summary of file types, potential categories, and recommendations for organization:

        {files_info}
        """

        # Call Claude API
        response = self.client.messages.create(
            model=self.model,
            max_tokens=2048,
            thinking=thinking,
            messages=[
                {"role": "user", "content": prompt}
            ]
        )

        return response.content[0].text

    def categorize_file(self, file_content, file_name):
        """
        Categorize a file based on its content

        Args:
            file_content: Content of the file
            file_name: Name of the file

        Returns:
            Suggested category for the file
        """
        # Configure thinking mode
        thinking = None
        if self.thinking_enabled:
            thinking = {
                "type": "enabled",
                "budget_tokens": 500  # Use smaller budget for single file
            }

        # Create prompt for Claude
        prompt = f"""
        Analyze this file and suggest a single category name for organizing it.
        Respond with just the category name, nothing else.

        File name: {file_name}
        File content: {file_content[:10000]}  # Limit content size
        """

        # Call Claude API
        response = self.client.messages.create(
            model=self.model,
            max_tokens=100,
            thinking=thinking,
            messages=[
                {"role": "user", "content": prompt}
            ]
        )

        # Clean up category name
        import re
        category = response.content[0].text.strip().lower()
        category = re.sub(r'[^\w\s]', '', category)
        category = re.sub(r'\s+', '_', category)

        return category
```

2. Create the file operations module (`src/core/file_ops.py`):

```python
import os
import shutil
import datetime
import hashlib

def scan_directory(directory):
    """
    Scan directory and return file metadata

    Args:
        directory: Path to directory to scan

    Returns:
        List of dictionaries containing file metadata
    """
    files_info = []

    for root, dirs, files in os.walk(directory):
        for file in files:
            file_path = os.path.join(root, file)
            try:
                stats = os.stat(file_path)

                # Get file extension
                _, extension = os.path.splitext(file)
                extension = extension[1:].lower() if extension else ""  # Remove the dot

                # Collect file metadata
                files_info.append({
                    'path': file_path,
                    'name': file,
                    'size': stats.st_size,
                    'created': datetime.datetime.fromtimestamp(stats.st_ctime).strftime('%Y-%m-%d %H:%M:%S'),
                    'modified': datetime.datetime.fromtimestamp(stats.st_mtime).strftime('%Y-%m-%d %H:%M:%S'),
                    'extension': extension
                })
            except Exception as e:
                print(f"Error processing {file_path}: {str(e)}")

    return files_info

def read_file_content(file_path, max_size=1024*1024):
    """
    Read the content of a file safely

    Args:
        file_path: Path to the file
        max_size: Maximum file size to read (in bytes)

    Returns:
        File content as string or None if file can't be read
    """
    # Check if file exists and is not too large
    if not os.path.isfile(file_path):
        return None

    if os.path.getsize(file_path) > max_size:
        return f"File too large: {os.path.getsize(file_path)} bytes"

    # Try to read the file
    try:
        with open(file_path, 'r', errors='ignore') as f:
            return f.read()
    except Exception as e:
        return f"Error reading file: {str(e)}"

def create_category_folders(base_dir, categories):
    """
    Create folders for each category if they don't exist

    Args:
        base_dir: Base directory where folders will be created
        categories: List of category names
    """
    for category in categories:
        category_dir = os.path.join(base_dir, category)
        os.makedirs(category_dir, exist_ok=True)

def move_file(source, destination):
    """
    Move a file with error handling and logging

    Args:
        source: Source file path
        destination: Destination file path

    Returns:
        True if successful, False otherwise
    """
    try:
        # Create destination directory if it doesn't exist
        dest_dir = os.path.dirname(destination)
        os.makedirs(dest_dir, exist_ok=True)

        # Move the file
        shutil.move(source, destination)
        return True
    except Exception as e:
        print(f"Error moving {source} to {destination}: {str(e)}")
        return False

def copy_file(source, destination):
    """
    Copy a file with error handling and logging

    Args:
        source: Source file path
        destination: Destination file path

    Returns:
        True if successful, False otherwise
    """
    try:
        # Create destination directory if it doesn't exist
        dest_dir = os.path.dirname(destination)
        os.makedirs(dest_dir, exist_ok=True)

        # Copy the file
        shutil.copy2(source, destination)
        return True
    except Exception as e:
        print(f"Error copying {source} to {destination}: {str(e)}")
        return False

def get_file_hash(file_path):
    """
    Calculate MD5 hash of a file

    Args:
        file_path: Path to the file

    Returns:
        MD5 hash of the file
    """
    hash_md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest()

def find_duplicates(directory):
    """
    Find duplicate files in a directory

    Args:
        directory: Directory to search for duplicates

    Returns:
        Dictionary mapping file hashes to lists of file paths
    """
    hash_dict = {}

    for root, dirs, files in os.walk(directory):
        for filename in files:
            file_path = os.path.join(root, filename)
            try:
                file_hash = get_file_hash(file_path)

                if file_hash in hash_dict:
                    hash_dict[file_hash].append(file_path)
                else:
                    hash_dict[file_hash] = [file_path]
            except Exception as e:
                print(f"Error processing {file_path}: {str(e)}")

    # Return only the duplicate files
    return {hash_val: paths for hash_val, paths in hash_dict.items() if len(paths) > 1}
```

3. Create the organizer module (`src/core/organizer.py`):

```python
import os
import shutil
import datetime
import re
from .file_ops import scan_directory, read_file_content, create_category_folders, copy_file

class FileOrganizer:
    """Class for organizing files"""

    def __init__(self, claude_client=None):
        """Initialize organizer with optional Claude client"""
        self.claude_client = claude_client

    def organize_by_extension(self, source_dir, dest_dir):
        """
        Organize files by their extension

        Args:
            source_dir: Directory containing files to organize
            dest_dir: Base directory where organized files will be placed

        Returns:
            Dictionary with results
        """
        # Create destination directory if it doesn't exist
        os.makedirs(dest_dir, exist_ok=True)

        # Common extension categories
        categories = {
            'documents': ['pdf', 'doc', 'docx', 'txt', 'rtf', 'odt', 'md'],
            'images': ['jpg', 'jpeg', 'png', 'gif', 'bmp', 'tiff', 'svg', 'webp'],
            'audio': ['mp3', 'wav', 'ogg', 'flac', 'm4a', 'aac'],
            'video': ['mp4', 'avi', 'mkv', 'mov', 'wmv', 'flv', 'webm'],
            'archives': ['zip', 'rar', '7z', 'tar', 'gz', 'bz2'],
            'code': ['py', 'js', 'html', 'css', 'java', 'cpp', 'c', 'h', 'php', 'rb']
        }

        # Create category folders
        for category in categories:
            os.makedirs(os.path.join(dest_dir, category), exist_ok=True)

        # Create 'other' folder for uncategorized files
        os.makedirs(os.path.join(dest_dir, 'other'), exist_ok=True)

        # Process each file in the source directory
        results = {
            'total_files': 0,
            'organized_files': 0,
            'failed_files': 0,
            'categories': {}
        }

        for root, dirs, files in os.walk(source_dir):
            for filename in files:
                file_path = os.path.join(root, filename)
                results['total_files'] += 1

                # Get file extension
                _, extension = os.path.splitext(filename)
                extension = extension[1:].lower()  # Remove the dot and convert to lowercase

                # Determine category
                category = 'other'
                for cat, exts in categories.items():
                    if extension in exts:
                        category = cat
                        break

                # Update category count
                if category not in results['categories']:
                    results['categories'][category] = 0
                results['categories'][category] += 1

                # Move file to appropriate category folder
                dest_path = os.path.join(dest_dir, category, filename)
                if copy_file(file_path, dest_path):
                    results['organized_files'] += 1
                else:
                    results['failed_files'] += 1

        return results

    def organize_by_date(self, source_dir, dest_dir):
        """
        Organize files by their creation date

        Args:
            source_dir: Directory containing files to organize
            dest_dir: Base directory where organized files will be placed

        Returns:
            Dictionary with results
        """
        # Create destination directory if it doesn't exist
        os.makedirs(dest_dir, exist_ok=True)

        # Process each file in the source directory
        results = {
            'total_files': 0,
            'organized_files': 0,
            'failed_files': 0,
            'years': {}
        }

        for root, dirs, files in os.walk(source_dir):
            for filename in files:
                file_path = os.path.join(root, filename)
                results['total_files'] += 1

                # Get file creation time
                creation_time = os.path.getctime(file_path)
                date = datetime.datetime.fromtimestamp(creation_time)

                # Create year and month folders
                year_folder = os.path.join(dest_dir, str(date.year))
                month_folder = os.path.join(year_folder, f"{date.month:02d}")

                os.makedirs(year_folder, exist_ok=True)
                os.makedirs(month_folder, exist_ok=True)

                # Update year count
                if str(date.year) not in results['years']:
                    results['years'][str(date.year)] = 0
                results['years'][str(date.year)] += 1

                # Move file to appropriate date folder
                dest_path = os.path.join(month_folder, filename)
                if copy_file(file_path, dest_path):
                    results['organized_files'] += 1
                else:
                    results['failed_files'] += 1

        return results

    def organize_by_content(self, source_dir, dest_dir):
        """
        Organize files by their content using Claude

        Args:
            source_dir: Directory containing files to organize
            dest_dir: Base directory where organized files will be placed

        Returns:
            Dictionary with results
        """
        # Check if Claude client is available
        if not self.claude_client:
            raise ValueError("Claude client is required for content-based organization")

        # Create destination directory if it doesn't exist
        os.makedirs(dest_dir, exist_ok=True)

        # Process each file in the source directory
        results = {
            'total_files': 0,
            'organized_files': 0,
            'failed_files': 0,
            'categories': {}
        }

        for root, dirs, files in os.walk(source_dir):
            for filename in files:
                file_path = os.path.join(root, filename)
                results['total_files'] += 1

                # Read file content
                content = read_file_content(file_path)
                if not content or content.startswith("Error") or content.startswith("File too large"):
                    results['failed_files'] += 1
                    continue

                # Get category from Claude
                try:
                    category = self.claude_client.categorize_file(content, filename)

                    # Create category folder
                    os.makedirs(os.path.join(dest_dir, category), exist_ok=True)

                    # Update category count
                    if category not in results['categories']:
                        results['categories'][category] = 0
                    results['categories'][category] += 1

                    # Move file to category folder
                    dest_path = os.path.join(dest_dir, category, filename)
                    if copy_file(file_path, dest_path):
                        results['organized_files'] += 1
                    else:
                        results['failed_files'] += 1

                except Exception as e:
                    print(f"Error categorizing {filename}: {str(e)}")
                    results['failed_files'] += 1

        return results
```

### Phase 2: Command-Line Interface

Create the CLI application (`src/cli/cli_app.py`):

```python
import os
import argparse
import json
from dotenv import load_dotenv
from ..core.claude_api import ClaudeClient
from ..core.file_ops import scan_directory
from ..core.organizer import FileOrganizer

# Load environment variables
load_dotenv()

def setup_argparse():
    """Set up command-line argument parser"""
    parser = argparse.ArgumentParser(description="AI File Organizer CLI")

    # Create subparsers for different commands
    subparsers = parser.add_subparsers(dest="command", help="Command to execute")

    # List command
    list_parser = subparsers.add_parser("list", help="List files in a directory")
    list_parser.add_argument("--dir", required=True, help="Directory to list files from")

    # Analyze command
    analyze_parser = subparsers.add_parser("analyze", help="Analyze files in a directory")
    analyze_parser.add_argument("--dir", required=True, help="Directory to analyze")

    # Organize command
    organize_parser = subparsers.add_parser("organize", help="Organize files in a directory")
    organize_parser.add_argument("--dir", required=True, help="Directory to organize")
    organize_parser.add_argument("--dest", required=True, help="Destination directory for organized files")
    organize_parser.add_argument("--method", choices=["extension", "date", "content"], default="extension",
                                help="Organization method (default: extension)")
    organize_parser.add_argument("--copy", action="store_true", help="Copy files instead of moving them")

    # Settings command
    settings_parser = subparsers.add_parser("settings", help="Manage settings")
    settings_parser.add_argument("--api-key", help="Set Anthropic API key")
    settings_parser.add_argument("--model", help="Set Claude model")
    settings_parser.add_argument("--thinking", choices=["on", "off"], help="Enable or disable thinking mode")

    return parser

def handle_list_command(args):
    """Handle the list command"""
    directory = args.dir

    if not os.path.isdir(directory):
        print(f"Error: {directory} is not a valid directory")
        return

    # Scan directory
    files_info = scan_directory(directory)

    # Print summary
    print(f"Found {len(files_info)} files in {directory}")

    # Group by extension
    extensions = {}
    for file_info in files_info:
        ext = file_info["extension"] or "no_extension"
        if ext not in extensions:
            extensions[ext] = 0
        extensions[ext] += 1

    # Print extension summary
    print("\nFile types:")
    for ext, count in sorted(extensions.items(), key=lambda x: x[1], reverse=True):
        print(f"  .{ext}: {count} files")

    # Print first 10 files
    print("\nFirst 10 files:")
    for i, file_info in enumerate(files_info[:10]):
        print(f"  {file_info['name']} ({file_info['size']} bytes)")

def handle_analyze_command(args):
    """Handle the analyze command"""
    directory = args.dir

    if not os.path.isdir(directory):
        print(f"Error: {directory} is not a valid directory")
        return

    # Get API key
    api_key = os.getenv("ANTHROPIC_API_KEY")
    if not api_key:
        print("Error: Anthropic API key not found. Set it using the ANTHROPIC_API_KEY environment variable.")
        return

    # Initialize Claude client
    try:
        claude_client = ClaudeClient(api_key)
    except Exception as e:
        print(f"Error initializing Claude client: {str(e)}")
        return

    # Scan directory
    print(f"Scanning directory {directory}...")
    files_info = scan_directory(directory)
    print(f"Found {len(files_info)} files")

    # Analyze with Claude
    print("Analyzing files with Claude...")
    analysis = claude_client.analyze_directory(files_info[:50])  # Limit to 50 files

    # Print analysis
    print("\nAnalysis:")
    print(analysis)

def handle_organize_command(args):
    """Handle the organize command"""
    source_dir = args.dir
    dest_dir = args.dest
    method = args.method

    if not os.path.isdir(source_dir):
        print(f"Error: {source_dir} is not a valid directory")
        return

    # Create organizer
    organizer = FileOrganizer()

    # If using content-based organization, initialize Claude client
    if method == "content":
        # Get API key
        api_key = os.getenv("ANTHROPIC_API_KEY")
        if not api_key:
            print("Error: Anthropic API key not found. Set it using the ANTHROPIC_API_KEY environment variable.")
            return

        # Initialize Claude client
        try:
            claude_client = ClaudeClient(api_key)
            organizer.claude_client = claude_client
        except Exception as e:
            print(f"Error initializing Claude client: {str(e)}")
            return

    # Organize files
    print(f"Organizing files from {source_dir} to {dest_dir} using {method} method...")

    try:
        if method == "extension":
            results = organizer.organize_by_extension(source_dir, dest_dir)
        elif method == "date":
            results = organizer.organize_by_date(source_dir, dest_dir)
        elif method == "content":
            results = organizer.organize_by_content(source_dir, dest_dir)

        # Print results
        print(f"\nOrganized {results['organized_files']} of {results['total_files']} files")
        print(f"Failed to organize {results['failed_files']} files")

        if method == "extension" or method == "content":
            print("\nCategories:")
            for category, count in sorted(results['categories'].items(), key=lambda x: x[1], reverse=True):
                print(f"  {category}: {count} files")
        elif method == "date":
            print("\nYears:")
            for year, count in sorted(results['years'].items()):
                print(f"  {year}: {count} files")

    except Exception as e:
        print(f"Error organizing files: {str(e)}")

def handle_settings_command(args):
    """Handle the settings command"""
    # Load current settings
    settings_file = os.path.join(os.path.expanduser("~"), ".ai_file_organizer", "settings.json")
    settings = {}

    if os.path.exists(settings_file):
        try:
            with open(settings_file, 'r') as f:
                settings = json.load(f)
        except Exception as e:
            print(f"Error loading settings: {str(e)}")

    # Update settings
    if args.api_key:
        settings["api_key"] = args.api_key

    if args.model:
        settings["model"] = args.model

    if args.thinking:
        settings["thinking_mode"] = args.thinking == "on"

    # Save settings
    if args.api_key or args.model or args.thinking:
        try:
            os.makedirs(os.path.dirname(settings_file), exist_ok=True)
            with open(settings_file, 'w') as f:
                json.dump(settings, f, indent=4)
            print("Settings saved successfully")
        except Exception as e:
            print(f"Error saving settings: {str(e)}")

    # Print current settings
    print("\nCurrent settings:")
    for key, value in settings.items():
        if key == "api_key" and value:
            print(f"  {key}: {'*' * 10}")
        else:
            print(f"  {key}: {value}")

def main():
    """Main entry point for CLI application"""
    parser = setup_argparse()
    args = parser.parse_args()

    if args.command == "list":
        handle_list_command(args)
    elif args.command == "analyze":
        handle_analyze_command(args)
    elif args.command == "organize":
        handle_organize_command(args)
    elif args.command == "settings":
        handle_settings_command(args)
    else:
        parser.print_help()

if __name__ == "__main__":
    main()
```

### Phase 3: Graphical User Interface

Create the GUI application (`src/gui/gui_app.py`):

```python
import os
import threading
import json
import PySimpleGUI as sg
from dotenv import load_dotenv
from ..core.claude_api import ClaudeClient
from ..core.file_ops import scan_directory
from ..core.organizer import FileOrganizer

# Load environment variables
load_dotenv()

def setup_dark_mode():
    """Configure PySimpleGUI for Windows 11 dark mode"""
    # Set dark theme
    sg.theme('DarkGrey13')

    # Further customize colors for Windows 11 style
    sg.set_options(window_color='#202020')
    sg.set_options(text_color='#ffffff')
    sg.set_options(button_color=('#ffffff', '#323232'))
    sg.set_options(font=('Segoe UI', 10))

    # Set global options
    sg.set_options(
        element_padding=(10, 5),
        margins=(15, 15),
        border_width=0,
        auto_size_buttons=True
    )

def create_win11_button(text, key=None):
    """Create a button with Windows 11 styling"""
    return sg.Button(
        text,
        key=key or text,
        border_width=0,
        button_color=('#ffffff', '#0067C0'),
        mouseover_colors=('#ffffff', '#0078D7'),
        font=('Segoe UI', 10)
    )

def create_win11_input(key, default_text='', size=(25, 1)):
    """Create an input field with Windows 11 styling"""
    return sg.Input(
        default_text=default_text,
        key=key,
        size=size,
        border_width=1,
        background_color='#333333',
        text_color='#ffffff'
    )

def create_win11_titlebar(title):
    """Create a custom titlebar with Windows 11 styling"""
    return [
        sg.Column(
            [[sg.Text(title, font=('Segoe UI', 12), text_color='#ffffff', background_color='#1F1F1F', pad=(10, 5))]],
            background_color='#1F1F1F',
            pad=(0, 0),
            expand_x=True
        ),
        sg.Column(
            [[sg.Button('✕', key='Exit', button_color=('#ffffff', '#1F1F1F'), border_width=0, font=('Segoe UI', 12))]],
            background_color='#1F1F1F',
            pad=(0, 0)
        )
    ]

def create_main_layout():
    """Create the main application layout with tabs"""
    # Tab 1: File Analysis
    analysis_tab = [
        [sg.Text("Directory:", font=('Segoe UI', 10))],
        [create_win11_input("-ANALYSIS-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],
        [create_win11_button("Analyze Files", key="-RUN-ANALYSIS-")],
        [sg.ProgressBar(100, orientation='h', size=(20, 20), key='-PROGRESS-ANALYSIS-', visible=False)],
        [sg.Text("", key="-STATUS-ANALYSIS-")],
        [sg.Multiline(size=(70, 15), key="-ANALYSIS-OUTPUT-", background_color='#333333', text_color='#ffffff')]
    ]

    # Tab 2: File Organization
    organization_tab = [
        [sg.Text("Source Directory:", font=('Segoe UI', 10))],
        [create_win11_input("-ORG-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],
        [sg.Text("Destination Directory:", font=('Segoe UI', 10))],
        [create_win11_input("-DEST-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],
        [sg.Text("Organization Method:", font=('Segoe UI', 10))],
        [
            sg.Radio("By Extension", "ORG_METHOD", key="-ORG-EXT-", background_color='#202020', text_color='#ffffff', default=True),
            sg.Radio("By Date", "ORG_METHOD", key="-ORG-DATE-", background_color='#202020', text_color='#ffffff'),
            sg.Radio("By Content (AI)", "ORG_METHOD", key="-ORG-AI-", background_color='#202020', text_color='#ffffff')
        ],
        [create_win11_button("Organize Files", key="-RUN-ORGANIZE-")],
        [sg.ProgressBar(100, orientation='h', size=(20, 20), key='-PROGRESS-ORG-', visible=False)],
        [sg.Text("", key="-STATUS-ORG-")],
        [sg.Multiline(size=(70, 10), key="-ORG-OUTPUT-", background_color='#333333', text_color='#ffffff')]
    ]

    # Tab 3: Settings
    settings_tab = [
        [sg.Text("Anthropic API Key:", font=('Segoe UI', 10))],
        [create_win11_input("-API-KEY-", size=(40, 1))],
        [sg.Text("Claude Model:", font=('Segoe UI', 10))],
        [
            sg.Combo(
                ["claude-3-7-sonnet-20250219", "claude-3-opus-20240229", "claude-3-haiku-20240307"],
                default_value="claude-3-7-sonnet-20250219",
                key="-MODEL-",
                background_color='#333333',
                text_color='#ffffff',
                button_background_color='#0067C0'
            )
        ],
        [sg.Text("Thinking Mode:", font=('Segoe UI', 10))],
        [
            sg.Checkbox("Enable Thinking Mode", key="-THINKING-MODE-", background_color='#202020', text_color='#ffffff', default=True),
            sg.Text("Budget (tokens):", font=('Segoe UI', 10)),
            create_win11_input("-THINKING-BUDGET-", default_text="1000", size=(10, 1))
        ],
        [sg.Text("File Size Limit (MB):", font=('Segoe UI', 10))],
        [create_win11_input("-FILE-SIZE-LIMIT-", default_text="1", size=(10, 1))],
        [create_win11_button("Save Settings", key="-SAVE-SETTINGS-")],
        [sg.Text("", key="-SETTINGS-STATUS-")]
    ]

    # Combine tabs
    layout = [
        # Title bar
        create_win11_titlebar("AI File Organizer"),

        # Tabs
        [sg.TabGroup(
            [[
                sg.Tab("Analysis", analysis_tab, background_color='#202020', key='-TAB1-'),
                sg.Tab("Organization", organization_tab, background_color='#202020', key='-TAB2-'),
                sg.Tab("Settings", settings_tab, background_color='#202020', key='-TAB3-')
            ]],
            tab_background_color='#202020',
            selected_background_color='#323232',
            tab_location='topleft',
            title_color='#ffffff',
            selected_title_color='#ffffff',
            background_color='#202020',
            border_width=0,
            pad=(0, 0)
        )]
    ]

    return layout

def load_settings():
    """Load settings from file"""
    # Default settings
    default_settings = {
        "api_key": os.getenv("ANTHROPIC_API_KEY", ""),
        "model": "claude-3-7-sonnet-20250219",
        "thinking_mode": True,
        "thinking_budget": 1000,
        "file_size_limit": 1.0
    }

    # Try to load settings from file
    settings_dir = os.path.join(os.path.expanduser("~"), ".ai_file_organizer")
    settings_file = os.path.join(settings_dir, "settings.json")

    try:
        if os.path.exists(settings_file):
            with open(settings_file, 'r') as f:
                settings = json.load(f)

            # Merge with default settings to ensure all keys exist
            for key, value in default_settings.items():
                if key not in settings:
                    settings[key] = value

            return settings
    except Exception:
        pass

    # Return default settings if loading fails
    return default_settings

def save_settings(settings):
    """Save settings to file"""
    settings_dir = os.path.join(os.path.expanduser("~"), ".ai_file_organizer")
    os.makedirs(settings_dir, exist_ok=True)

    settings_file = os.path.join(settings_dir, "settings.json")
    with open(settings_file, 'w') as f:
        json.dump(settings, f, indent=4)

def thread_analyze_directory(directory, window):
    """Analyze directory in a separate thread"""
    try:
        # Load settings
        settings = load_settings()
        api_key = settings.get("api_key", "")
        model = settings.get("model", "claude-3-7-sonnet-20250219")
        thinking_mode = settings.get("thinking_mode", True)
        thinking_budget = settings.get("thinking_budget", 1000)

        if not api_key:
            window.write_event_value('-THREAD-DONE-', ("analysis", "Error: API key not found in settings. Please go to Settings tab and enter your Anthropic API key."))
            return

        # Initialize Claude client
        client = ClaudeClient(api_key)
        client.set_model(model)
        client.set_thinking_mode(thinking_mode, thinking_budget)

        # Update status
        window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ANALYSIS-", "Scanning directory..."))

        # Scan directory
        files_info = scan_directory(directory)

        # Update status
        window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ANALYSIS-", f"Found {len(files_info)} files. Analyzing with Claude..."))

        # Limit to first 50 files to avoid overwhelming Claude
        files_info = files_info[:50]

        # Use Claude to analyze directory
        analysis = client.analyze_directory(files_info)

        # Return analysis result
        window.write_event_value('-THREAD-DONE-', ("analysis", analysis))

    except Exception as e:
        window.write_event_value('-THREAD-DONE-', ("analysis", f"Error: {str(e)}"))

def thread_organize_directory(directory, dest_dir, method, window):
    """Organize directory in a separate thread"""
    try:
        # Load settings
        settings = load_settings()

        # Create organizer
        organizer = FileOrganizer()

        # If using content-based organization, initialize Claude client
        if method == "content":
            api_key = settings.get("api_key", "")
            model = settings.get("model", "claude-3-7-sonnet-20250219")
            thinking_mode = settings.get("thinking_mode", True)
            thinking_budget = settings.get("thinking_budget", 1000)

            if not api_key:
                window.write_event_value('-THREAD-DONE-', ("organization", "Error: API key not found in settings. Please go to Settings tab and enter your Anthropic API key."))
                return

            # Initialize Claude client
            client = ClaudeClient(api_key)
            client.set_model(model)
            client.set_thinking_mode(thinking_mode, thinking_budget)
            organizer.claude_client = client

        # Update status
        window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ORG-", f"Organizing files using {method} method..."))

        # Organize files
        if method == "extension":
            results = organizer.organize_by_extension(directory, dest_dir)
        elif method == "date":
            results = organizer.organize_by_date(directory, dest_dir)
        elif method == "content":
            results = organizer.organize_by_content(directory, dest_dir)

        # Format results
        output = f"Organized {results['organized_files']} of {results['total_files']} files\n"
        output += f"Failed to organize {results['failed_files']} files\n\n"

        if method == "extension" or method == "content":
            output += "Categories:\n"
            for category, count in sorted(results['categories'].items(), key=lambda x: x[1], reverse=True):
                output += f"  {category}: {count} files\n"
        elif method == "date":
            output += "Years:\n"
            for year, count in sorted(results['years'].items()):
                output += f"  {year}: {count} files\n"

        # Return results
        window.write_event_value('-THREAD-DONE-', ("organization", output))

    except Exception as e:
        window.write_event_value('-THREAD-DONE-', ("organization", f"Error: {str(e)}"))

def thread_save_settings(api_key, model, thinking_mode, thinking_budget, file_size_limit, window):
    """Save settings in a separate thread"""
    try:
        # Create settings object
        settings = {
            "api_key": api_key,
            "model": model,
            "thinking_mode": thinking_mode,
            "thinking_budget": int(thinking_budget),
            "file_size_limit": float(file_size_limit)
        }

        # Save settings to file
        save_settings(settings)

        # Return success
        window.write_event_value('-THREAD-DONE-', ("settings", "Settings saved successfully."))

    except Exception as e:
        window.write_event_value('-THREAD-DONE-', ("settings", f"Error saving settings: {str(e)}"))

def main():
    """Main entry point for GUI application"""
    # Setup
    setup_dark_mode()
    layout = create_main_layout()
    window = sg.Window(
        "AI File Organizer",
        layout,
        finalize=True,
        use_custom_titlebar=True,
        keep_on_top=False,
        no_titlebar=True,
        grab_anywhere=True,
        margins=(0, 0),
        background_color='#202020'
    )

    # Load settings
    settings = load_settings()
    window["-API-KEY-"].update(settings.get("api_key", ""))
    window["-MODEL-"].update(settings.get("model", "claude-3-7-sonnet-20250219"))
    window["-THINKING-MODE-"].update(settings.get("thinking_mode", True))
    window["-THINKING-BUDGET-"].update(settings.get("thinking_budget", 1000))
    window["-FILE-SIZE-LIMIT-"].update(settings.get("file_size_limit", 1.0))

    # Event loop
    while True:
        event, values = window.read()

        # Exit events
        if event == sg.WINDOW_CLOSED or event == "Exit":
            break

        # Handle analyze button
        if event == "-RUN-ANALYSIS-":
            directory = values["-ANALYSIS-DIR-"]
            if not directory:
                sg.popup_error("Please select a directory", background_color='#333333', text_color='#ffffff')
                continue

            window["-ANALYSIS-OUTPUT-"].update("Analyzing directory... Please wait.")
            window["-PROGRESS-ANALYSIS-"].update(visible=True)

            # Run analysis in a separate thread
            threading.Thread(
                target=thread_analyze_directory,
                args=(directory, window),
                daemon=True
            ).start()

        # Handle organize button
        if event == "-RUN-ORGANIZE-":
            directory = values["-ORG-DIR-"]
            dest_dir = values["-DEST-DIR-"]

            if not directory:
                sg.popup_error("Please select a source directory", background_color='#333333', text_color='#ffffff')
                continue

            if not dest_dir:
                sg.popup_error("Please select a destination directory", background_color='#333333', text_color='#ffffff')
                continue

            # Determine organization method
            if values["-ORG-EXT-"]:
                method = "extension"
            elif values["-ORG-DATE-"]:
                method = "date"
            elif values["-ORG-AI-"]:
                method = "content"

            window["-ORG-OUTPUT-"].update(f"Organizing directory using {method} method... Please wait.")
            window["-PROGRESS-ORG-"].update(visible=True, current_count=0)

            # Run organization in a separate thread
            threading.Thread(
                target=thread_organize_directory,
                args=(directory, dest_dir, method, window),
                daemon=True
            ).start()

        # Handle save settings button
        if event == "-SAVE-SETTINGS-":
            api_key = values["-API-KEY-"]
            model = values["-MODEL-"]
            thinking_mode = values["-THINKING-MODE-"]
            thinking_budget = values["-THINKING-BUDGET-"]
            file_size_limit = values["-FILE-SIZE-LIMIT-"]

            window["-SETTINGS-STATUS-"].update("Saving settings...")

            # Save settings in a separate thread
            threading.Thread(
                target=thread_save_settings,
                args=(api_key, model, thinking_mode, thinking_budget, file_size_limit, window),
                daemon=True
            ).start()

        # Handle thread completion
        if event == '-THREAD-DONE-':
            action, result = values[event]

            if action == "analysis":
                window["-ANALYSIS-OUTPUT-"].update(result)
                window["-PROGRESS-ANALYSIS-"].update(visible=False)
                window["-STATUS-ANALYSIS-"].update("Analysis complete.")

            elif action == "organization":
                window["-ORG-OUTPUT-"].update(result)
                window["-PROGRESS-ORG-"].update(visible=False)
                window["-STATUS-ORG-"].update("Organization complete.")

            elif action == "settings":
                window["-SETTINGS-STATUS-"].update(result)

        # Handle status updates
        if event == '-STATUS-UPDATE-':
            key, message = values[event]
            window[key].update(message)

    # Cleanup
    window.close()

if __name__ == "__main__":
    main()
```

### Phase 4: Main Entry Point

Create the main entry point (`src/main.py`):

```python
#!/usr/bin/env python3
import sys
import os
import argparse

def main():
    """Main entry point for the application"""
    parser = argparse.ArgumentParser(description="AI File Organizer")
    parser.add_argument("--cli", action="store_true", help="Run in CLI mode")
    parser.add_argument("--gui", action="store_true", help="Run in GUI mode")

    args = parser.parse_args()

    # Default to GUI mode if no mode specified
    if not args.cli and not args.gui:
        args.gui = True

    # Run in CLI mode
    if args.cli:
        from src.cli.cli_app import main as cli_main
        cli_main()

    # Run in GUI mode
    elif args.gui:
        from src.gui.gui_app import main as gui_main
        gui_main()

if __name__ == "__main__":
    main()
```

## Project Setup

### Requirements File

Create a `requirements.txt` file:

```
anthropic==0.7.0
python-dotenv==1.0.0
PySimpleGUI==4.60.5
```

### README File

Create a `README.md` file:

```markdown
# AI File Organizer

An intelligent file organization tool powered by Claude 3.7 Sonnet.

## Features

- Directory content analysis
- AI-powered organization suggestions
- Automatic file categorization
- Both CLI and GUI interfaces
- Content-based file grouping

## Installation

1. Clone the repository:

```bash
git clone https://github.com/yourusername/ai-file-organizer.git
cd ai-file-organizer
```

2. Create a virtual environment and install dependencies:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. Create a `.env` file with your Anthropic API key:

```
ANTHROPIC_API_KEY=your_api_key_here
```

## Usage

### GUI Mode

```bash
python src/main.py
```

### CLI Mode

```bash
python src/main.py --cli
```

#### CLI Commands

- List files in a directory:

```bash
python src/main.py --cli list --dir /path/to/directory
```

- Analyze files in a directory:

```bash
python src/main.py --cli analyze --dir /path/to/directory
```

- Organize files by extension:

```bash
python src/main.py --cli organize --dir /path/to/directory --dest /path/to/destination --method extension
```

- Organize files by date:

```bash
python src/main.py --cli organize --dir /path/to/directory --dest /path/to/destination --method date
```

- Organize files by content (using Claude):

```bash
python src/main.py --cli organize --dir /path/to/directory --dest /path/to/destination --method content
```

## License

MIT
```

## Implementation Considerations

### Claude Integration Best Practices

- Provide meaningful context to Claude by including relevant file metadata
- Use thinking mode for complex organization tasks
- Create clear prompts that define organization goals
- Limit the number of files sent to Claude in a single request
- Implement batch processing for large directories

### File Operations Safety

- Always use copy operations by default, with an option to move files
- Implement undo functionality
- Log all file operations
- Handle permission errors gracefully
- Provide progress indicators for long-running operations

### GUI Design Principles

- Keep the interface simple and intuitive
- Provide clear feedback for long-running operations
- Include dark mode option for Windows 11
- Use threading for non-blocking operations
- Implement progress indicators for file operations

## Future Enhancements

### Version 1.5: Enhanced Organization

- Content-based duplicate detection
- Custom organization rules
- More detailed file analysis
- Filename standardization

### Version 2.0: Advanced Features

- Custom organization profiles
- Scheduled organization tasks
- Tag-based organization system
- Integration with cloud storage services
- Advanced duplicate file management