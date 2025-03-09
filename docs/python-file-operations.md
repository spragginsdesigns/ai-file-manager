# Python File Operations for AI File Organizer

This document provides comprehensive information about Python file operations that are essential for the AI File Organizer project.

## Basic File Operations

### Listing Files in a Directory

```python
import os

# List all files in a directory
files = os.listdir(directory_path)

# Recursively list all files in a directory and subdirectories
for root, dirs, files in os.walk(directory_path):
    for file in files:
        file_path = os.path.join(root, file)
        print(file_path)
```

### Getting File Information

```python
import os
import datetime

def get_file_info(file_path):
    """Get comprehensive information about a file"""
    stats = os.stat(file_path)

    # Basic file information
    file_info = {
        'path': file_path,
        'name': os.path.basename(file_path),
        'directory': os.path.dirname(file_path),
        'size': stats.st_size,  # Size in bytes
        'created': datetime.datetime.fromtimestamp(stats.st_ctime),
        'modified': datetime.datetime.fromtimestamp(stats.st_mtime),
        'accessed': datetime.datetime.fromtimestamp(stats.st_atime),
    }

    # Get file extension
    filename, extension = os.path.splitext(file_path)
    file_info['extension'] = extension[1:] if extension else ''  # Remove the dot

    return file_info
```

### Creating Directories

```python
import os

# Create a directory
os.mkdir(directory_path)

# Create a directory and all necessary parent directories
os.makedirs(directory_path, exist_ok=True)  # exist_ok=True prevents error if directory exists
```

### Moving and Copying Files

```python
import shutil

# Move a file
shutil.move(source_path, destination_path)

# Copy a file
shutil.copy2(source_path, destination_path)  # copy2 preserves metadata
```

## File Organization Strategies

### Organizing by File Extension

```python
import os
import shutil

def organize_by_extension(source_dir, dest_dir):
    """
    Organize files by their extension

    Args:
        source_dir: Directory containing files to organize
        dest_dir: Base directory where organized files will be placed
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
    for filename in os.listdir(source_dir):
        file_path = os.path.join(source_dir, filename)

        # Skip directories
        if not os.path.isfile(file_path):
            continue

        # Get file extension
        _, extension = os.path.splitext(filename)
        extension = extension[1:].lower()  # Remove the dot and convert to lowercase

        # Determine category
        category = 'other'
        for cat, exts in categories.items():
            if extension in exts:
                category = cat
                break

        # Move file to appropriate category folder
        dest_path = os.path.join(dest_dir, category, filename)
        shutil.copy2(file_path, dest_path)  # Use copy2 to preserve metadata

        print(f"Moved {filename} to {category}")

    return True
```

### Organizing by Creation Date

```python
import os
import shutil
import datetime

def organize_by_date(source_dir, dest_dir):
    """
    Organize files by their creation date

    Args:
        source_dir: Directory containing files to organize
        dest_dir: Base directory where organized files will be placed
    """
    # Create destination directory if it doesn't exist
    os.makedirs(dest_dir, exist_ok=True)

    # Process each file in the source directory
    for filename in os.listdir(source_dir):
        file_path = os.path.join(source_dir, filename)

        # Skip directories
        if not os.path.isfile(file_path):
            continue

        # Get file creation time
        creation_time = os.path.getctime(file_path)
        date = datetime.datetime.fromtimestamp(creation_time)

        # Create year and month folders
        year_folder = os.path.join(dest_dir, str(date.year))
        month_folder = os.path.join(year_folder, f"{date.month:02d}")

        os.makedirs(year_folder, exist_ok=True)
        os.makedirs(month_folder, exist_ok=True)

        # Move file to appropriate date folder
        dest_path = os.path.join(month_folder, filename)
        shutil.copy2(file_path, dest_path)  # Use copy2 to preserve metadata

        print(f"Moved {filename} to {date.year}/{date.month:02d}")

    return True
```

## Content-Based Organization with Claude

### Reading File Content

```python
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
```

### Analyzing Content with Claude

```python
def analyze_file_content(file_path, claude_client):
    """
    Use Claude to analyze file content and suggest a category

    Args:
        file_path: Path to the file
        claude_client: Initialized Anthropic client

    Returns:
        Suggested category as string
    """
    # Read file content
    content = read_file_content(file_path)

    if not content or content.startswith("Error") or content.startswith("File too large"):
        return "unknown"

    # Truncate content if it's too long
    if len(content) > 10000:
        content = content[:10000] + "..."

    # Use Claude to analyze content
    try:
        response = claude_client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=100,
            thinking={"type": "enabled", "budget_tokens": 500},
            messages=[
                {"role": "user", "content": f"Analyze this file content and suggest a single category name for organizing it. Respond with just the category name, nothing else: {content}"}
            ]
        )

        category = response.content[0].text.strip().lower()

        # Clean up category name (remove punctuation, etc.)
        import re
        category = re.sub(r'[^\w\s]', '', category)
        category = re.sub(r'\s+', '_', category)

        return category

    except Exception as e:
        print(f"Error analyzing content: {str(e)}")
        return "unknown"
```

### Organizing Files by Content

```python
def organize_by_content(source_dir, dest_dir, claude_client):
    """
    Organize files by their content using Claude

    Args:
        source_dir: Directory containing files to organize
        dest_dir: Base directory where organized files will be placed
        claude_client: Initialized Anthropic client
    """
    # Create destination directory if it doesn't exist
    os.makedirs(dest_dir, exist_ok=True)

    # Process each file in the source directory
    for filename in os.listdir(source_dir):
        file_path = os.path.join(source_dir, filename)

        # Skip directories
        if not os.path.isfile(file_path):
            continue

        # Get category from Claude
        category = analyze_file_content(file_path, claude_client)

        # Create category folder
        category_folder = os.path.join(dest_dir, category)
        os.makedirs(category_folder, exist_ok=True)

        # Move file to category folder
        dest_path = os.path.join(category_folder, filename)
        shutil.copy2(file_path, dest_path)

        print(f"Moved {filename} to {category}")

    return True
```

## Advanced File Operations

### Handling Duplicate Files

```python
import hashlib

def get_file_hash(file_path):
    """Calculate MD5 hash of a file"""
    hash_md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest()

def find_duplicates(directory):
    """Find duplicate files in a directory"""
    hash_dict = {}

    for root, dirs, files in os.walk(directory):
        for filename in files:
            file_path = os.path.join(root, filename)
            file_hash = get_file_hash(file_path)

            if file_hash in hash_dict:
                hash_dict[file_hash].append(file_path)
            else:
                hash_dict[file_hash] = [file_path]

    # Return only the duplicate files
    return {hash_val: paths for hash_val, paths in hash_dict.items() if len(paths) > 1}
```

### Renaming Files with Consistent Patterns

```python
def standardize_filenames(directory):
    """Rename files to follow a consistent pattern"""
    for filename in os.listdir(directory):
        file_path = os.path.join(directory, filename)

        if not os.path.isfile(file_path):
            continue

        # Get file information
        file_info = get_file_info(file_path)

        # Create standardized name: YYYY-MM-DD_original-name.ext
        date_str = file_info['created'].strftime('%Y-%m-%d')
        name_part = os.path.splitext(filename)[0].lower().replace(' ', '-')
        extension = file_info['extension']

        new_name = f"{date_str}_{name_part}.{extension}"
        new_path = os.path.join(directory, new_name)

        # Rename file
        os.rename(file_path, new_path)
        print(f"Renamed {filename} to {new_name}")
```

## Error Handling and Safety

### Safe File Operations

```python
def safe_move(source, destination):
    """Safely move a file with error handling"""
    try:
        # Check if destination exists
        if os.path.exists(destination):
            # Create a backup
            backup = f"{destination}.bak"
            shutil.copy2(destination, backup)
            print(f"Created backup: {backup}")

        # Move the file
        shutil.move(source, destination)
        return True

    except Exception as e:
        print(f"Error moving {source} to {destination}: {str(e)}")
        return False

def safe_delete(file_path):
    """Safely delete a file with error handling"""
    try:
        # Create a backup in a temporary directory
        import tempfile
        temp_dir = tempfile.gettempdir()
        backup = os.path.join(temp_dir, os.path.basename(file_path))
        shutil.copy2(file_path, backup)

        # Delete the file
        os.remove(file_path)
        print(f"Deleted {file_path} (backup at {backup})")
        return True

    except Exception as e:
        print(f"Error deleting {file_path}: {str(e)}")
        return False
```

### Logging Operations

```python
import logging

def setup_logging(log_file='file_organizer.log'):
    """Set up logging for file operations"""
    logging.basicConfig(
        filename=log_file,
        level=logging.INFO,
        format='%(asctime)s - %(levelname)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )

    # Also log to console
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    formatter = logging.Formatter('%(levelname)s: %(message)s')
    console.setFormatter(formatter)
    logging.getLogger('').addHandler(console)

def log_operation(operation, source, destination=None, success=True):
    """Log a file operation"""
    if destination:
        message = f"{operation}: {source} -> {destination}"
    else:
        message = f"{operation}: {source}"

    if success:
        logging.info(message)
    else:
        logging.error(message)
```

## Integration with Claude API

### Batch Processing Files

```python
def batch_process_with_claude(files, claude_client, batch_size=10):
    """
    Process files in batches to optimize Claude API usage

    Args:
        files: List of file paths
        claude_client: Initialized Anthropic client
        batch_size: Number of files to process in each batch

    Returns:
        Dictionary mapping file paths to categories
    """
    results = {}

    # Process files in batches
    for i in range(0, len(files), batch_size):
        batch = files[i:i+batch_size]

        # Prepare batch content
        batch_content = []
        for file_path in batch:
            content = read_file_content(file_path)
            if content and not content.startswith("Error") and not content.startswith("File too large"):
                # Truncate content if needed
                if len(content) > 1000:
                    content = content[:1000] + "..."

                batch_content.append(f"File: {os.path.basename(file_path)}\nContent: {content}\n---")

        # Skip empty batches
        if not batch_content:
            continue

        # Use Claude to analyze batch
        try:
            response = claude_client.messages.create(
                model="claude-3-7-sonnet-20250219",
                max_tokens=1000,
                thinking={"type": "enabled", "budget_tokens": 2000},
                messages=[
                    {"role": "user", "content": f"For each file below, suggest a category for organizing it. Respond with a JSON object mapping filenames to categories. Files:\n\n{''.join(batch_content)}"}
                ]
            )

            # Parse response (assuming Claude returns valid JSON)
            import json
            try:
                categories = json.loads(response.content[0].text)

                # Map categories back to file paths
                for file_path in batch:
                    filename = os.path.basename(file_path)
                    if filename in categories:
                        results[file_path] = categories[filename]
                    else:
                        results[file_path] = "unknown"

            except json.JSONDecodeError:
                # Fallback if Claude doesn't return valid JSON
                for file_path in batch:
                    results[file_path] = "unknown"

        except Exception as e:
            print(f"Error processing batch: {str(e)}")
            # Set unknown category for all files in the batch
            for file_path in batch:
                results[file_path] = "unknown"

    return results
```

## Resources

- [Python os module documentation](https://docs.python.org/3/library/os.html)
- [Python shutil module documentation](https://docs.python.org/3/library/shutil.html)
- [Python pathlib module documentation](https://docs.python.org/3/library/pathlib.html)