# PySimpleGUI Implementation for AI File Organizer

This document provides comprehensive information about implementing a Windows 11 dark mode GUI for the AI File Organizer project using PySimpleGUI.

## PySimpleGUI Overview

PySimpleGUI is a Python package that simplifies the process of creating GUIs. It provides a wrapper around Tkinter (and other GUI frameworks) that makes it easier to create attractive and functional interfaces with minimal code.

### Key Features

- Simple API that abstracts away complex GUI programming
- Support for multiple GUI frameworks (Tkinter, Qt, WxPython, Remi)
- Extensive widget collection
- Theming capabilities
- Event-driven programming model
- Built-in support for dark mode themes

## Installation

```bash
pip install PySimpleGUI
```

## Basic Structure

A PySimpleGUI application typically follows this structure:

```python
import PySimpleGUI as sg

# 1. Set theme
sg.theme('DarkGrey13')  # Use a dark theme for Windows 11

# 2. Define the layout
layout = [
    [sg.Text("Hello World")],
    [sg.Button("OK")]
]

# 3. Create the window
window = sg.Window("My Application", layout)

# 4. Event loop
while True:
    event, values = window.read()

    # 5. Process events
    if event == sg.WINDOW_CLOSED or event == "OK":
        break

# 6. Close the window
window.close()
```

## Windows 11 Dark Mode Implementation

### Setting Up Dark Mode

```python
import PySimpleGUI as sg

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
```

### Windows 11 Style Elements

```python
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
```

## Main Application Layout

### Basic Layout

```python
def create_main_layout():
    """Create the main application layout"""
    layout = [
        # Title bar
        create_win11_titlebar("AI File Organizer"),

        # Directory selection
        [sg.Text("Directory:", font=('Segoe UI', 10))],
        [create_win11_input("-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],

        # Action buttons
        [
            create_win11_button("Analyze", key="-ANALYZE-"),
            create_win11_button("Organize", key="-ORGANIZE-"),
            create_win11_button("Settings", key="-SETTINGS-")
        ],

        # Output area
        [sg.Text("Results:", font=('Segoe UI', 10))],
        [sg.Multiline(size=(70, 20), key="-OUTPUT-", background_color='#333333', text_color='#ffffff')]
    ]

    return layout
```

### Advanced Layout with Tabs

```python
def create_advanced_layout():
    """Create an advanced application layout with tabs"""
    # Tab 1: File Analysis
    analysis_tab = [
        [sg.Text("Directory:", font=('Segoe UI', 10))],
        [create_win11_input("-ANALYSIS-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],
        [create_win11_button("Analyze Files", key="-RUN-ANALYSIS-")],
        [sg.Multiline(size=(70, 15), key="-ANALYSIS-OUTPUT-", background_color='#333333', text_color='#ffffff')]
    ]

    # Tab 2: File Organization
    organization_tab = [
        [sg.Text("Directory:", font=('Segoe UI', 10))],
        [create_win11_input("-ORG-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],
        [sg.Text("Destination:", font=('Segoe UI', 10))],
        [create_win11_input("-DEST-DIR-"), sg.FolderBrowse(button_color=('#ffffff', '#0067C0'))],
        [sg.Text("Organization Method:", font=('Segoe UI', 10))],
        [
            sg.Radio("By Extension", "ORG_METHOD", key="-ORG-EXT-", background_color='#202020', text_color='#ffffff', default=True),
            sg.Radio("By Date", "ORG_METHOD", key="-ORG-DATE-", background_color='#202020', text_color='#ffffff'),
            sg.Radio("By Content (AI)", "ORG_METHOD", key="-ORG-AI-", background_color='#202020', text_color='#ffffff')
        ],
        [create_win11_button("Organize Files", key="-RUN-ORGANIZE-")],
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
        [create_win11_button("Save Settings", key="-SAVE-SETTINGS-")]
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
```

## Event Handling

### Basic Event Loop

```python
def run_application():
    """Run the main application loop"""
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

    # Event loop
    while True:
        event, values = window.read()

        # Exit events
        if event == sg.WINDOW_CLOSED or event == "Exit":
            break

        # Handle analyze button
        if event == "-ANALYZE-":
            directory = values["-DIR-"]
            if not directory:
                sg.popup_error("Please select a directory", background_color='#333333', text_color='#ffffff')
                continue

            window["-OUTPUT-"].update("Analyzing directory... Please wait.")
            # Perform analysis (implementation in next section)
            result = analyze_directory(directory)
            window["-OUTPUT-"].update(result)

        # Handle organize button
        if event == "-ORGANIZE-":
            directory = values["-DIR-"]
            if not directory:
                sg.popup_error("Please select a directory", background_color='#333333', text_color='#ffffff')
                continue

            window["-OUTPUT-"].update("Organizing directory... Please wait.")
            # Perform organization (implementation in next section)
            result = organize_directory(directory)
            window["-OUTPUT-"].update(result)

    # Cleanup
    window.close()
```

### Advanced Event Handling with Threading

```python
import threading
import queue

def run_advanced_application():
    """Run the application with advanced event handling and threading"""
    # Setup
    setup_dark_mode()
    layout = create_advanced_layout()
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

    # Create a queue for thread communication
    task_queue = queue.Queue()

    # Event loop
    while True:
        event, values = window.read(timeout=100)  # Add timeout for checking queue

        # Exit events
        if event == sg.WINDOW_CLOSED or event == "Exit":
            break

        # Check for completed tasks
        try:
            task_result = task_queue.get_nowait()
            if task_result:
                action, result = task_result
                if action == "analysis":
                    window["-ANALYSIS-OUTPUT-"].update(result)
                elif action == "organization":
                    window["-ORG-OUTPUT-"].update(result)
                elif action == "settings":
                    window["-ANALYSIS-OUTPUT-"].update("Settings saved successfully.")
        except queue.Empty:
            pass

        # Handle analysis button
        if event == "-RUN-ANALYSIS-":
            directory = values["-ANALYSIS-DIR-"]
            if not directory:
                sg.popup_error("Please select a directory", background_color='#333333', text_color='#ffffff')
                continue

            window["-ANALYSIS-OUTPUT-"].update("Analyzing directory... Please wait.")

            # Run analysis in a separate thread
            threading.Thread(
                target=thread_analyze_directory,
                args=(directory, task_queue),
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

            # Run organization in a separate thread
            threading.Thread(
                target=thread_organize_directory,
                args=(directory, dest_dir, method, task_queue),
                daemon=True
            ).start()

        # Handle settings button
        if event == "-SAVE-SETTINGS-":
            api_key = values["-API-KEY-"]
            model = values["-MODEL-"]
            thinking_mode = values["-THINKING-MODE-"]
            thinking_budget = values["-THINKING-BUDGET-"]
            file_size_limit = values["-FILE-SIZE-LIMIT-"]

            # Save settings (implementation in next section)
            threading.Thread(
                target=thread_save_settings,
                args=(api_key, model, thinking_mode, thinking_budget, file_size_limit, task_queue),
                daemon=True
            ).start()

    # Cleanup
    window.close()
```

## Integration with File Operations and Claude API

### Thread Functions

```python
def thread_analyze_directory(directory, result_queue):
    """Analyze directory in a separate thread"""
    try:
        # Import necessary modules
        import os
        import anthropic
        from datetime import datetime

        # Load API key from settings
        api_key = load_settings().get("api_key", "")
        if not api_key:
            result_queue.put(("analysis", "Error: API key not found in settings."))
            return

        # Initialize Claude client
        client = anthropic.Anthropic(api_key=api_key)

        # Collect file information
        files_info = []
        for root, dirs, files in os.walk(directory):
            for file in files:
                file_path = os.path.join(root, file)
                try:
                    stats = os.stat(file_path)
                    files_info.append({
                        'path': file_path,
                        'name': file,
                        'size': stats.st_size,
                        'modified': datetime.fromtimestamp(stats.st_mtime).strftime('%Y-%m-%d %H:%M:%S'),
                        'extension': os.path.splitext(file)[1][1:].lower() if os.path.splitext(file)[1] else ''
                    })
                except Exception as e:
                    continue

        # Limit to first 50 files to avoid overwhelming Claude
        files_info = files_info[:50]

        # Use Claude to analyze directory
        response = client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=2048,
            thinking={"type": "enabled", "budget_tokens": 5000},
            messages=[
                {"role": "user", "content": f"Analyze these files and suggest an organization strategy. Provide a summary of file types, potential categories, and recommendations for organization:\n\n{files_info}"}
            ]
        )

        # Return analysis result
        result_queue.put(("analysis", response.content[0].text))

    except Exception as e:
        result_queue.put(("analysis", f"Error: {str(e)}"))

def thread_organize_directory(directory, dest_dir, method, result_queue):
    """Organize directory in a separate thread"""
    try:
        # Import necessary modules
        import os
        import shutil
        import anthropic
        from datetime import datetime

        # Create destination directory if it doesn't exist
        os.makedirs(dest_dir, exist_ok=True)

        # Initialize variables for tracking progress
        total_files = 0
        processed_files = 0

        # Count total files
        for root, dirs, files in os.walk(directory):
            total_files += len(files)

        # Organize by extension
        if method == "extension":
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

            # Process each file
            results = []
            for root, dirs, files in os.walk(directory):
                for filename in files:
                    file_path = os.path.join(root, filename)

                    # Get file extension
                    _, extension = os.path.splitext(filename)
                    extension = extension[1:].lower()  # Remove the dot and convert to lowercase

                    # Determine category
                    category = 'other'
                    for cat, exts in categories.items():
                        if extension in exts:
                            category = cat
                            break

                    # Create destination path
                    dest_path = os.path.join(dest_dir, category, filename)

                    # Copy file
                    try:
                        shutil.copy2(file_path, dest_path)
                        results.append(f"Copied {filename} to {category}")
                    except Exception as e:
                        results.append(f"Error copying {filename}: {str(e)}")

                    # Update progress
                    processed_files += 1

            # Return results
            result_queue.put(("organization", "\n".join(results)))

        # Organize by date
        elif method == "date":
            # Process each file
            results = []
            for root, dirs, files in os.walk(directory):
                for filename in files:
                    file_path = os.path.join(root, filename)

                    # Get file creation time
                    creation_time = os.path.getctime(file_path)
                    date = datetime.fromtimestamp(creation_time)

                    # Create year and month folders
                    year_folder = os.path.join(dest_dir, str(date.year))
                    month_folder = os.path.join(year_folder, f"{date.month:02d}")

                    os.makedirs(year_folder, exist_ok=True)
                    os.makedirs(month_folder, exist_ok=True)

                    # Create destination path
                    dest_path = os.path.join(month_folder, filename)

                    # Copy file
                    try:
                        shutil.copy2(file_path, dest_path)
                        results.append(f"Copied {filename} to {date.year}/{date.month:02d}")
                    except Exception as e:
                        results.append(f"Error copying {filename}: {str(e)}")

                    # Update progress
                    processed_files += 1

            # Return results
            result_queue.put(("organization", "\n".join(results)))

        # Organize by content (AI)
        elif method == "content":
            # Load API key from settings
            api_key = load_settings().get("api_key", "")
            if not api_key:
                result_queue.put(("organization", "Error: API key not found in settings."))
                return

            # Initialize Claude client
            client = anthropic.Anthropic(api_key=api_key)

            # Process each file
            results = []
            for root, dirs, files in os.walk(directory):
                for filename in files:
                    file_path = os.path.join(root, filename)

                    # Read file content
                    try:
                        with open(file_path, 'r', errors='ignore') as f:
                            content = f.read(10000)  # Read first 10KB
                    except Exception as e:
                        results.append(f"Error reading {filename}: {str(e)}")
                        processed_files += 1
                        continue

                    # Use Claude to analyze content
                    try:
                        response = client.messages.create(
                            model="claude-3-7-sonnet-20250219",
                            max_tokens=100,
                            thinking={"type": "enabled", "budget_tokens": 500},
                            messages=[
                                {"role": "user", "content": f"Analyze this file content and suggest a single category name for organizing it. Respond with just the category name, nothing else: {content}"}
                            ]
                        )

                        category = response.content[0].text.strip().lower()

                        # Clean up category name
                        import re
                        category = re.sub(r'[^\w\s]', '', category)
                        category = re.sub(r'\s+', '_', category)

                        # Create category folder
                        os.makedirs(os.path.join(dest_dir, category), exist_ok=True)

                        # Create destination path
                        dest_path = os.path.join(dest_dir, category, filename)

                        # Copy file
                        shutil.copy2(file_path, dest_path)
                        results.append(f"Copied {filename} to {category}")

                    except Exception as e:
                        results.append(f"Error processing {filename}: {str(e)}")

                    # Update progress
                    processed_files += 1

            # Return results
            result_queue.put(("organization", "\n".join(results)))

    except Exception as e:
        result_queue.put(("organization", f"Error: {str(e)}"))

def thread_save_settings(api_key, model, thinking_mode, thinking_budget, file_size_limit, result_queue):
    """Save settings in a separate thread"""
    try:
        # Import necessary modules
        import json
        import os

        # Create settings object
        settings = {
            "api_key": api_key,
            "model": model,
            "thinking_mode": thinking_mode,
            "thinking_budget": int(thinking_budget),
            "file_size_limit": float(file_size_limit)
        }

        # Save settings to file
        settings_dir = os.path.join(os.path.expanduser("~"), ".ai_file_organizer")
        os.makedirs(settings_dir, exist_ok=True)

        settings_file = os.path.join(settings_dir, "settings.json")
        with open(settings_file, 'w') as f:
            json.dump(settings, f, indent=4)

        # Return success
        result_queue.put(("settings", "Settings saved successfully."))

    except Exception as e:
        result_queue.put(("settings", f"Error saving settings: {str(e)}"))
```

### Settings Management

```python
def load_settings():
    """Load settings from file"""
    import json
    import os

    # Default settings
    default_settings = {
        "api_key": "",
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
```

## Progress Indicators and User Feedback

### Progress Bar

```python
def create_progress_bar(window, key, max_value):
    """Create and update a progress bar"""
    window[key].update(visible=True, current_count=0, max=max_value)

    def update_progress(current_value):
        window[key].update(current_count=current_value)

    return update_progress
```

### Status Updates

```python
def update_status(window, key, message):
    """Update status message"""
    window[key].update(message)
```

## Complete Application Example

```python
import PySimpleGUI as sg
import os
import threading
import queue
import anthropic
import shutil
from datetime import datetime
import json
import re

# Settings management
def load_settings():
    """Load settings from file"""
    # Default settings
    default_settings = {
        "api_key": "",
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

# GUI setup
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

# Thread functions
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
        client = anthropic.Anthropic(api_key=api_key)

        # Update status
        window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ANALYSIS-", "Scanning directory..."))

        # Collect file information
        files_info = []
        for root, dirs, files in os.walk(directory):
            for file in files:
                file_path = os.path.join(root, file)
                try:
                    stats = os.stat(file_path)
                    files_info.append({
                        'path': file_path,
                        'name': file,
                        'size': stats.st_size,
                        'modified': datetime.fromtimestamp(stats.st_mtime).strftime('%Y-%m-%d %H:%M:%S'),
                        'extension': os.path.splitext(file)[1][1:].lower() if os.path.splitext(file)[1] else ''
                    })
                except Exception as e:
                    continue

        # Update status
        window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ANALYSIS-", f"Found {len(files_info)} files. Analyzing with Claude..."))

        # Limit to first 50 files to avoid overwhelming Claude
        files_info = files_info[:50]

        # Use Claude to analyze directory
        thinking_params = {"type": "enabled", "budget_tokens": thinking_budget} if thinking_mode else None

        response = client.messages.create(
            model=model,
            max_tokens=2048,
            thinking=thinking_params,
            messages=[
                {"role": "user", "content": f"Analyze these files and suggest an organization strategy. Provide a summary of file types, potential categories, and recommendations for organization:\n\n{files_info}"}
            ]
        )

        # Return analysis result
        window.write_event_value('-THREAD-DONE-', ("analysis", response.content[0].text))

    except Exception as e:
        window.write_event_value('-THREAD-DONE-', ("analysis", f"Error: {str(e)}"))

def thread_organize_directory(directory, dest_dir, method, window):
    """Organize directory in a separate thread"""
    try:
        # Create destination directory if it doesn't exist
        os.makedirs(dest_dir, exist_ok=True)

        # Initialize variables for tracking progress
        total_files = 0
        processed_files = 0

        # Count total files
        window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ORG-", "Counting files..."))
        for root, dirs, files in os.walk(directory):
            total_files += len(files)

        # Update progress bar max value
        window.write_event_value('-PROGRESS-MAX-', ("-PROGRESS-ORG-", total_files))

        # Organize by extension
        if method == "extension":
            window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ORG-", f"Organizing {total_files} files by extension..."))

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

            # Process each file
            results = []
            for root, dirs, files in os.walk(directory):
                for filename in files:
                    file_path = os.path.join(root, filename)

                    # Get file extension
                    _, extension = os.path.splitext(filename)
                    extension = extension[1:].lower()  # Remove the dot and convert to lowercase

                    # Determine category
                    category = 'other'
                    for cat, exts in categories.items():
                        if extension in exts:
                            category = cat
                            break

                    # Create destination path
                    dest_path = os.path.join(dest_dir, category, filename)

                    # Copy file
                    try:
                        shutil.copy2(file_path, dest_path)
                        results.append(f"Copied {filename} to {category}")
                    except Exception as e:
                        results.append(f"Error copying {filename}: {str(e)}")

                    # Update progress
                    processed_files += 1
                    window.write_event_value('-PROGRESS-UPDATE-', ("-PROGRESS-ORG-", processed_files))

            # Return results
            window.write_event_value('-THREAD-DONE-', ("organization", "\n".join(results)))

        # Organize by date
        elif method == "date":
            window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ORG-", f"Organizing {total_files} files by date..."))

            # Process each file
            results = []
            for root, dirs, files in os.walk(directory):
                for filename in files:
                    file_path = os.path.join(root, filename)

                    # Get file creation time
                    creation_time = os.path.getctime(file_path)
                    date = datetime.fromtimestamp(creation_time)

                    # Create year and month folders
                    year_folder = os.path.join(dest_dir, str(date.year))
                    month_folder = os.path.join(year_folder, f"{date.month:02d}")

                    os.makedirs(year_folder, exist_ok=True)
                    os.makedirs(month_folder, exist_ok=True)

                    # Create destination path
                    dest_path = os.path.join(month_folder, filename)

                    # Copy file
                    try:
                        shutil.copy2(file_path, dest_path)
                        results.append(f"Copied {filename} to {date.year}/{date.month:02d}")
                    except Exception as e:
                        results.append(f"Error copying {filename}: {str(e)}")

                    # Update progress
                    processed_files += 1
                    window.write_event_value('-PROGRESS-UPDATE-', ("-PROGRESS-ORG-", processed_files))

            # Return results
            window.write_event_value('-THREAD-DONE-', ("organization", "\n".join(results)))

        # Organize by content (AI)
        elif method == "content":
            # Load settings
            settings = load_settings()
            api_key = settings.get("api_key", "")
            model = settings.get("model", "claude-3-7-sonnet-20250219")
            thinking_mode = settings.get("thinking_mode", True)
            thinking_budget = settings.get("thinking_budget", 1000)
            file_size_limit = settings.get("file_size_limit", 1.0) * 1024 * 1024  # Convert MB to bytes

            if not api_key:
                window.write_event_value('-THREAD-DONE-', ("organization", "Error: API key not found in settings. Please go to Settings tab and enter your Anthropic API key."))
                return

            # Initialize Claude client
            client = anthropic.Anthropic(api_key=api_key)

            window.write_event_value('-STATUS-UPDATE-', ("-STATUS-ORG-", f"Organizing {total_files} files by content using Claude..."))

            # Process each file
            results = []
            for root, dirs, files in os.walk(directory):
                for filename in files:
                    file_path = os.path.join(root, filename)

                    # Skip files that are too large
                    if os.path.getsize(file_path) > file_size_limit:
                        results.append(f"Skipped {filename} (too large)")
                        processed_files += 1
                        window.write_event_value('-PROGRESS-UPDATE-', ("-PROGRESS-ORG-", processed_files))
                        continue

                    # Read file content
                    try:
                        with open(file_path, 'r', errors='ignore') as f:
                            content = f.read(10000)  # Read first 10KB
                    except Exception as e:
                        results.append(f"Error reading {filename}: {str(e)}")
                        processed_files += 1
                        window.write_event_value('-PROGRESS-UPDATE-', ("-PROGRESS-ORG-", processed_files))
                        continue

                    # Use Claude to analyze content
                    try:
                        thinking_params = {"type": "enabled", "budget_tokens": 500} if thinking_mode else None

                        response = client.messages.create(
                            model=model,
                            max_tokens=100,
                            thinking=thinking_params,
                            messages=[
                                {"role": "user", "content": f"Analyze this file content and suggest a single category name for organizing it. Respond with just the category name, nothing else: {content}"}
                            ]
                        )

                        category = response.content[0].text.strip().lower()

                        # Clean up category name
                        category = re.sub(r'[^\w\s]', '', category)
                        category = re.sub(r'\s+', '_', category)

                        # Create category folder
                        os.makedirs(os.path.join(dest_dir, category), exist_ok=True)

                        # Create destination path
                        dest_path = os.path.join(dest_dir, category, filename)

                        # Copy file
                        shutil.copy2(file_path, dest_path)
                        results.append(f"Copied {filename} to {category}")

                    except Exception as e:
                        results.append(f"Error processing {filename}: {str(e)}")

                    # Update progress
                    processed_files += 1
                    window.write_event_value('-PROGRESS-UPDATE-', ("-PROGRESS-ORG-", processed_files))

            # Return results
            window.write_event_value('-THREAD-DONE-', ("organization", "\n".join(results)))

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

# Main application
def main():
    """Run the main application"""
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

        # Handle progress updates
        if event == '-PROGRESS-UPDATE-':
            key, value = values[event]
            window[key].update(current_count=value)

        # Handle progress max value
        if event == '-PROGRESS-MAX-':
            key, value = values[event]
            window[key].update(max=value)

    # Cleanup
    window.close()

if __name__ == "__main__":
    main()
```

## Resources

- [PySimpleGUI Documentation](https://pysimplegui.readthedocs.io/en/latest/)
- [PySimpleGUI Cookbook](https://www.pysimplegui.org/en/latest/cookbook/)
- [PySimpleGUI Demo Programs](https://github.com/PySimpleGUI/PySimpleGUI/tree/master/DemoPrograms)
- [Windows 11 Design Guidelines](https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/design-principles)