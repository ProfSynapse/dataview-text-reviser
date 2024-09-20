# Text Reviser for Obsidian

## Introduction

Welcome to the **Text Reviser** tool for Obsidian! This DataviewJS script allows you to effortlessly revise and enhance your text within Obsidian notes using AI-driven language models. Whether you're aiming to improve clarity, grammar, or overall quality, Text Reviser integrates seamlessly into your workflow, providing instant revisions tailored to your instructions.

## Prerequisites

Before setting up Text Reviser, ensure you have the following:

- **Obsidian**: Installed on your device. [Download Obsidian](https://obsidian.md/)
- **Dataview Plugin**: Essential for running DataviewJS scripts.
- **API Key (Supported AI Provider Account)**:
  - **OpenRouter**
  - **Anthropic**
  - **Gemini**
  - **LM Studio** (Local Server)

## Installation

### a. Install the Dataview Plugin

1. **Open Obsidian** and navigate to **Settings** (gear icon).
2. In the left sidebar, click on **Community Plugins**.
3. Toggle **Safe Mode** to **OFF** to allow third-party plugins.
4. Click on **Browse**.
5. Search for **Dataview**.
6. Click **Install**, then **Enable** once the installation completes.
7. **Enable JavaScript Queries** and **Enable inline Javascript queries**:
   - Go to **Settings** > **Dataview**.
   - Toggle **Enable Javascript Queries** and **Enable inline Javascript queries** to **ON**.

### b. Add the Text Reviser Script

1. **Locate the Markdown File**: Find the `.md` file containing the Text Reviser code block for the provider you prefer.
2. **Copy the Entire Code Block**: Ensure you include the backticks and `dataviewjs` tag. It should look like this:

   ```
   ```dataviewjs
   // Text Reviser script goes here
   ```
   ```

4. **Paste into Your Note**: Open the Obsidian note where you want to use Text Reviser and paste the entire code block.

## Configuration

### Insert Your API Key

At the top of the pasted DataviewJS code block, you'll find a placeholder for your API key. Replace the placeholder with your actual API key from your chosen provider.

**Example:**

```javascript
const OPENROUTER_API_KEY = "YOUR_OPENROUTER_API_KEY_HERE";
// Replace "YOUR_OPENROUTER_API_KEY_HERE" with your actual OpenRouter API key
```

**Note:** If you're using **LM Studio**, refer to the [LM Studio Configuration](#lm-studio-configuration) section below.

## Customization

### Updating Prompts

The effectiveness of Text Reviser largely depends on the prompts provided. Customize them to fit your specific revision needs.

1. **Locate the Prompt Section**: Within the script, find the section where prompts are defined for different AI providers.
2. **Modify Prompts**: Adjust the instructions to better suit your desired revision style.

**Example: Customizing Revision Instructions**

```javascript
const prompt = `Professor Synapse, I need your help revising the following text. Please ensure it is clear, concise, and free of grammatical errors.

Text:
${content}`;
```

### Adding New Features

Feel free to extend the script to include additional functionalities such as integrating new AI providers or adding more revision options.

## Usage

### Adding the Text Reviser to a Note

1. **Open a Note**: Navigate to the note where you want to use Text Reviser.
2. **Ensure DataviewJS Block is Present**: The script should be within a DataviewJS code block as described in the [Installation](#installation) section.

### Revise Text

1. **Select Text**: Highlight the text you wish to revise within the note. If no text is selected, the entire note's content will be used.
2. **Click Revise Button**: A **Revise Selected Text** button (circular with a pencil icon) appears near the bottom of the note.
3. **Enter Instructions**:
   - A modal prompts you to input revision instructions (e.g., "Improve clarity", "Make it more concise").
4. **Submit Instructions**: Click **Submit** to send the request to the AI.
5. **Review Revised Text**:
   - A new modal displays the revised text.
   - Options available:
     - **Copy Revised Text**: Copies the text to your clipboard.
     - **Replace Text**: Inserts the revised text into your note.
     - **Revise Again**: Allows further revisions based on new instructions.
     - **Cancel**: Closes the modal without making changes.

## Example Prompt for LLM

If you wish to customize or extend the script using an LLM (e.g., ChatGPT), here's how you can proceed:

**Prompt Example:**

> "I have a DataviewJS script for a Text Reviser in Obsidian. The script includes API integrations with OpenRouter, Anthropic, Gemini, and LM Studio. At the top of each script, there's a placeholder to insert the API key. I want to update the revision prompts to be more specific to academic writing and add a feature that logs each revision made. Please modify the script accordingly, ensuring that the API key section remains clearly defined and provide comments where changes were made."

**Steps:**

1. **Feed the Script and Prompt**: Provide the existing script along with the above prompt to the LLM.
2. **Review Changes**: The LLM will update the prompts and add the requested logging feature.
3. **Implement Updates**: Copy the modified script back into your Obsidian note within the DataviewJS code block.

## Support and Contributions

We welcome your contributions and feedback!

- **Discussion Forum**: Join the conversation on our GitHub Discussions.
- **Issues and Feature Requests**: Submit them via the GitHub Issues page.
- **Contribute**: Fork the repository, make your changes, and submit a pull request. We're excited to collaborate and improve together!
