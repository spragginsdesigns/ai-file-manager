# Anthropic API Documentation for Claude 3.7 Sonnet

This document provides comprehensive information about the Anthropic API, specifically focusing on Claude 3.7 Sonnet and its capabilities for the AI File Organizer project.

## Claude 3.7 Sonnet Overview

Claude 3.7 Sonnet is Anthropic's first hybrid reasoning model, combining fast responses with extended, step-by-step thinking capabilities.

### Key Features

- **Hybrid Reasoning**: Combines fast responses with extended, step-by-step thinking
- **Extended Thinking Mode**: Toggle between standard mode and extended thinking mode
- **Adjustable Reasoning Budget**: Control thinking tokens (up to 128K tokens)
- **Large Context Window**: Supports up to 200K tokens for input
- **Multimodal Capabilities**: Processes both text and images
- **Code Generation**: Excels at software engineering tasks
- **Computer Use Capabilities**: Can interact with computer interfaces
- **Multiple Language Support**: Handles various languages including English, French, Arabic, Chinese, Hindi, Spanish, and more
- **API Integration**: Available through Anthropic's API, Amazon Bedrock, and Google Cloud's Vertex AI
- **Pricing**: $3 per million input tokens and $15 per million output tokens (includes thinking tokens)

## Standard Mode vs. Extended Thinking Mode

Claude 3.7 Sonnet introduces two key modes:

### Standard Mode
- Similar to previous Claude models
- Provides quick responses for straightforward queries
- Better for simple, conversational interactions
- Uses fewer tokens, resulting in lower costs
- Default mode in the API

### Extended Thinking Mode
- Allows Claude to engage in more in-depth, step-by-step reasoning
- Improves performance on complex tasks like math, physics, coding, and multi-step problems
- Shows the model's reasoning process
- Uses more tokens, increasing costs but potentially improving quality
- Must be explicitly enabled in API calls

## Python Implementation

### Installation

```bash
pip install anthropic
```

### Basic Usage

```python
import anthropic

# Initialize the client
client = anthropic.Anthropic()

# Make an API call
response = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello, Claude! What's new in version 3.7?"}
    ]
)

# Print the response
print(response.content[0].text)
```

### Using Extended Thinking Mode

```python
response = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=2048,
    thinking={
        "type": "enabled",
        "budget_tokens": 1024
    },
    messages=[
        {"role": "user", "content": "Explain quantum entanglement in detail."}
    ]
)

# Print the thinking process and final response
for content in response.content:
    if content.type == "thinking":
        print("Thinking:", content.thinking)
    elif content.type == "text":
        print("Final response:", content.text)
```

### Setting a Thinking Budget

```python
response = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1000,
    messages=[{"role": "user", "content": "Analyze this code..."}],
    thinking={
        "type": "enabled",
        "budget_tokens": 5000
    }
)
```

### Streaming Responses

```python
with client.messages.stream(
    model="claude-3-7-sonnet-20250219",
    max_tokens=2048,
    messages=[
        {"role": "user", "content": "Write a short story about time travel."}
    ]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Streaming with Thinking Mode

```python
for event in client.messages.stream(
    model="claude-3-7-sonnet-20250219",
    max_tokens=2048,
    thinking={"type": "enabled"},
    messages=[{"role": "user", "content": "Solve this complex problem..."}]
):
    if event.type == "thinking_delta":
        print("Reasoning:", event.delta.text)
    elif event.type == "text_delta":
        print("Answer:", event.delta.text)
```

### Multimodal Capabilities

```python
import base64

# Load an image file
with open("image.jpg", "rb") as image_file:
    image_data = base64.b64encode(image_file.read()).decode('utf-8')

response = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/jpeg",
                        "data": image_data
                    }
                },
                {
                    "type": "text",
                    "text": "Describe this image in detail."
                }
            ]
        }
    ]
)

print(response.content[0].text)
```

## Integration with AI File Organizer

For the AI File Organizer project, Claude 3.7 Sonnet can be used to:

1. **Analyze file content**: Use Claude to understand the content of files and suggest appropriate categories
2. **Generate organization strategies**: Ask Claude to recommend optimal organization structures based on file types and content
3. **Extract metadata**: Use Claude to identify key information in files that can be used for organization
4. **Understand user queries**: Process natural language requests from users about how they want their files organized

### Example: Content-Based File Organization

```python
def organize_by_content(source_dir, dest_dir, claude_client):
    for filename in os.listdir(source_dir):
        file_path = os.path.join(source_dir, filename)
        if os.path.isfile(file_path):
            # Read file content
            try:
                with open(file_path, 'r') as f:
                    content = f.read()
            except:
                continue

            # Use Claude to analyze content and suggest category
            response = claude_client.messages.create(
                model="claude-3-7-sonnet-20250219",
                max_tokens=100,
                thinking={"type": "enabled", "budget_tokens": 500},
                messages=[
                    {"role": "user", "content": f"Analyze this file content and suggest a single category name for organizing it: {content[:2000]}"}
                ]
            )

            category = response.content[0].text.strip()

            # Move file to category folder
            dest_folder = os.path.join(dest_dir, category)
            os.makedirs(dest_folder, exist_ok=True)
            shutil.move(file_path, os.path.join(dest_folder, filename))
```

## Windows 11 GUI Integration

For the AI File Organizer's Windows 11 dark mode GUI, PySimpleGUI is recommended:

### Installation

```bash
pip install PySimpleGUI
```

### Dark Mode Implementation

```python
import PySimpleGUI as sg

# Set dark theme
sg.theme('DarkGrey13')

# Further customize colors for Windows 11 style
sg.set_options(window_color='#202020')
sg.set_options(text_color='#ffffff')
sg.set_options(button_color=('#ffffff', '#323232'))

# Create window with custom titlebar for Windows 11 style
layout = [
    [sg.Text("AI File Organizer", font=("Segoe UI", 16))],
    [sg.Text("Directory:"), sg.Input(key="-DIR-"), sg.FolderBrowse()],
    [sg.Button("Analyze"), sg.Button("Organize")],
    [sg.Multiline(size=(70, 20), key="-OUTPUT-")]
]

window = sg.Window("AI File Organizer", layout, use_custom_titlebar=True)
```

### Integrating Claude API with GUI

```python
import PySimpleGUI as sg
import anthropic
import os
import threading

def process_directory(directory, client, window):
    # This function runs in a separate thread to keep GUI responsive
    window.write_event_value('-PROCESSING-', 'Started')

    # Use Claude to analyze directory
    files_info = []
    for root, dirs, files in os.walk(directory):
        for file in files:
            files_info.append(os.path.join(root, file))

    # Send first 50 files to Claude for analysis
    response = client.messages.create(
        model="claude-3-7-sonnet-20250219",
        max_tokens=2048,
        thinking={"type": "enabled", "budget_tokens": 5000},
        messages=[
            {"role": "user", "content": f"Analyze these files and suggest an organization strategy: {files_info[:50]}"}
        ]
    )

    window.write_event_value('-RESULT-', response.content[0].text)

# Main GUI loop
def main():
    client = anthropic.Anthropic()

    # GUI layout
    layout = [
        [sg.Text("AI File Organizer", font=("Segoe UI", 16))],
        [sg.Text("Directory:"), sg.Input(key="-DIR-"), sg.FolderBrowse()],
        [sg.Button("Analyze"), sg.Button("Organize")],
        [sg.Multiline(size=(70, 20), key="-OUTPUT-")]
    ]

    window = sg.Window("AI File Organizer", layout, use_custom_titlebar=True)

    while True:
        event, values = window.read()

        if event == sg.WINDOW_CLOSED:
            break

        if event == "Analyze" and values["-DIR-"]:
            window["-OUTPUT-"].update("Analyzing directory... Please wait.")
            threading.Thread(
                target=process_directory,
                args=(values["-DIR-"], client, window),
                daemon=True
            ).start()

        if event == '-RESULT-':
            window["-OUTPUT-"].update(values[event])

    window.close()

if __name__ == "__main__":
    main()
```

## Resources

- [Anthropic API Documentation](https://docs.anthropic.com/claude/reference/getting-started-with-the-api)
- [Claude Models Overview](https://docs.anthropic.com/claude/docs/models-overview)
- [Thinking Mode Documentation](https://docs.anthropic.com/claude/docs/thinking-mode)
- [Anthropic Client Libraries](https://docs.anthropic.com/claude/docs/anthropic-client-libraries)
- [Messages Streaming API](https://docs.anthropic.com/claude/reference/messages-streaming)
- [PySimpleGUI Documentation](https://pysimplegui.readthedocs.io/en/latest/)
- [PySimpleGUI Cookbook](https://www.pysimplegui.org/en/latest/cookbook/)