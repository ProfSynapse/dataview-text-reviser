```dataviewjs
const OPENROUTER_API_KEY = "YOUR_API_KEY_HERE";

const app = dv.app; // Access the Obsidian app instance

// Reference to the revise button
let reviseButton = null;

// Initialize the revise button functionality
function initReviseButton() {
    document.addEventListener('selectionchange', handleSelectionChange);
    window.addEventListener('scroll', handleScroll);
}

// Handle selection changes
function handleSelectionChange() {
    const selection = window.getSelection();
    const selectedText = selection.toString().trim();

    if (selectedText.length > 0 && !reviseButton) {
        // Show the revise button near the selection
        showReviseButton(selection);
    } else if (selectedText.length === 0 && reviseButton) {
        // Hide the revise button if no text is selected
        hideReviseButton();
    }
}

// Handle window scroll to reposition the button if visible
function handleScroll() {
    if (reviseButton) {
        const selection = window.getSelection();
        if (selection.rangeCount > 0) {
            repositionButton(selection);
        }
    }
}

// Function to create and display the revise button
function showReviseButton(selection) {
    // Create the button element
    reviseButton = document.createElement('button');
    reviseButton.id = 'reviseButton';
    reviseButton.classList.add('revise-button');

    // Add pen icon (SVG)
    reviseButton.innerHTML = `
        <svg viewBox="0 0 24 24" fill="currentColor" width="20" height="20">
            <path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c0.39-0.39 0.39-1.03 0-1.42l-2.34-2.34c-0.39-0.39-1.03-0.39-1.42 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/>
        </svg>
    `;

    // Style the button
    reviseButton.style.position = 'absolute';
    reviseButton.style.zIndex = '1000';
    reviseButton.style.width = '32px';
    reviseButton.style.height = '32px';
    reviseButton.style.borderRadius = '50%';
    reviseButton.style.display = 'flex';
    reviseButton.style.justifyContent = 'center';
    reviseButton.style.alignItems = 'center';
    reviseButton.style.cursor = 'pointer';
    reviseButton.style.backgroundColor = 'var(--interactive-accent)';
    reviseButton.style.color = 'var(--text-on-accent)';
    reviseButton.style.boxShadow = 'var(--overlay-shadow)';
    reviseButton.style.border = 'none';
    reviseButton.style.padding = '4px';

    // Append the button to the body
    document.body.appendChild(reviseButton);

    // Position the button near the selected text
    repositionButton(selection);

    // Attach click event to the button
    reviseButton.onclick = async () => {
        const activeLeaf = app.workspace.activeLeaf;
        const activeView = activeLeaf?.view;
        const editor = activeView?.editor;

        if (!editor) {
            new Notice('📝 Revise: No active editor found.');
            return;
        }

        let selectedText = editor.getSelection();
        if (!selectedText) {
            new Notice('📝 Revise: No text selected.');
            return;
        }

        const instructions = await openInstructionModal();
        if (instructions) {
            await reviseText(selectedText, instructions, editor);
        }
    };
}

// Function to reposition the button near the selected text
function repositionButton(selection) {
    if (!reviseButton) return;

    const range = selection.getRangeAt(0);
    const rect = range.getBoundingClientRect();

    // Calculate position: above and centered on the selection
    const top = rect.top + window.scrollY - 40; // Adjust as needed
    const left = rect.left + window.scrollX + (rect.width / 2) - (reviseButton.offsetWidth / 2);

    reviseButton.style.top = `${top}px`;
    reviseButton.style.left = `${left}px`;
}

// Function to hide and remove the revise button
function hideReviseButton() {
    if (reviseButton) {
        reviseButton.remove();
        reviseButton = null;
    }
}

// Function to open a custom modal to prompt for revision instructions
async function openInstructionModal() {
    return new Promise((resolve) => {
        // Create the modal overlay
        const modalOverlay = document.createElement('div');
        modalOverlay.className = 'custom-modal-overlay';

        // Create the modal container
        const modalContainer = document.createElement('div');
        modalContainer.className = 'custom-modal-container';

        // Modal header
        const modalHeader = document.createElement('h2');
        modalHeader.textContent = 'Revise Text';
        modalContainer.appendChild(modalHeader);

        // Instruction label
        const instructionLabel = document.createElement('label');
        instructionLabel.textContent = 'Enter revision instructions:';
        modalContainer.appendChild(instructionLabel);

        // Instruction textarea
        const instructionTextarea = document.createElement('textarea');
        instructionTextarea.placeholder = 'Type your instructions here...';
        instructionTextarea.rows = 4;
        modalContainer.appendChild(instructionTextarea);

        // Action buttons
        const buttonContainer = document.createElement('div');
        buttonContainer.className = 'modal-button-container';

        const submitButton = document.createElement('button');
        submitButton.textContent = 'Submit';
        submitButton.className = 'mod-cta';
        submitButton.onclick = () => {
            const instructions = instructionTextarea.value.trim();
            if (instructions) {
                document.body.removeChild(modalOverlay);
                resolve(instructions);
            } else {
                new Notice('Please enter some instructions.');
            }
        };

        const cancelButton = document.createElement('button');
        cancelButton.textContent = 'Cancel';
        cancelButton.onclick = () => {
            document.body.removeChild(modalOverlay);
            resolve(null);
        };

        buttonContainer.appendChild(submitButton);
        buttonContainer.appendChild(cancelButton);
        modalContainer.appendChild(buttonContainer);

        // Append modal to overlay
        modalOverlay.appendChild(modalContainer);
        document.body.appendChild(modalOverlay);
    });
}

// Function to revise text using OpenRouter.ai API
async function reviseText(selectedText, instructions, editor) {
    // Create the loading overlay
    const loadingOverlay = document.createElement('div');
    loadingOverlay.className = 'custom-modal-overlay';
    const loadingContainer = document.createElement('div');
    loadingContainer.className = 'custom-modal-container';
    loadingContainer.textContent = 'Revising...';
    loadingOverlay.appendChild(loadingContainer);
    document.body.appendChild(loadingOverlay);

    try {
        const prompt = `
Revise the following text based on these instructions: ${instructions}

Text:
${selectedText}
        `;

        const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
            method: "POST",
            headers: {
                "Authorization": `Bearer ${OPENROUTER_API_KEY}`,
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                "model": "anthropic/claude-3.5-sonnet",
                "messages": [
                    {
                        "role": "system",
                        "content": "You are an expert editor and proofreader. Revise the user's text according to their instructions, ensuring clarity, correctness, and adherence to the requested style and tone."
                    },
                    {
                        "role": "user",
                        "content": prompt
                    }
                ],
                "temperature": 0.7
            })
        });

        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(`Error ${response.status}: ${errorData.error?.message || 'Unknown error'}`);
        }

        const data = await response.json();
        const revisedText = data.choices?.[0]?.message?.content?.trim() || 'No revised text returned.';

        document.body.removeChild(loadingOverlay);

        // Open modal to display revised text
        openRevisedTextModal(revisedText, editor);

    } catch (error) {
        console.error("Error calling OpenRouter.ai API:", error);
        new Notice(`Error: ${error.message}`);
        document.body.removeChild(loadingOverlay);
    }
}

// Function to display the revised text in a custom modal
function openRevisedTextModal(revisedText, editor) {
    // Create the modal overlay
    const modalOverlay = document.createElement('div');
    modalOverlay.className = 'custom-modal-overlay';

    // Create the modal container
    const modalContainer = document.createElement('div');
    modalContainer.className = 'custom-modal-container';

    // Modal header
    const modalHeader = document.createElement('h2');
    modalHeader.textContent = 'Revised Text';
    modalContainer.appendChild(modalHeader);

    // Revised text label
    const revisedTextLabel = document.createElement('label');
    revisedTextLabel.textContent = 'You can edit the revised text before applying it:';
    modalContainer.appendChild(revisedTextLabel);

    // Revised text textarea
    const revisedTextarea = document.createElement('textarea');
    revisedTextarea.value = revisedText;
    revisedTextarea.rows = 10;
    modalContainer.appendChild(revisedTextarea);

    // Action buttons
    const buttonContainer = document.createElement('div');
    buttonContainer.className = 'modal-button-container';

    const copyButton = document.createElement('button');
    copyButton.textContent = 'Copy to Clipboard';
    copyButton.onclick = () => {
        navigator.clipboard.writeText(revisedTextarea.value);
        new Notice('Revised text copied to clipboard.');
    };

    const replaceButton = document.createElement('button');
    replaceButton.textContent = 'Replace Selection';
    replaceButton.className = 'mod-cta';
    replaceButton.onclick = () => {
        editor.replaceSelection(revisedTextarea.value);
        new Notice('Text replaced.');
        document.body.removeChild(modalOverlay);
    };

    const cancelButton = document.createElement('button');
    cancelButton.textContent = 'Cancel';
    cancelButton.onclick = () => {
        document.body.removeChild(modalOverlay);
    };

    buttonContainer.appendChild(copyButton);
    buttonContainer.appendChild(replaceButton);
    buttonContainer.appendChild(cancelButton);
    modalContainer.appendChild(buttonContainer);

    // Append modal to overlay
    modalOverlay.appendChild(modalContainer);
    document.body.appendChild(modalOverlay);
}

// Initialize the revise button when the script is loaded
initReviseButton();

// Add custom CSS for the modals
const style = document.createElement('style');
style.innerHTML = `
.custom-modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: var(--background-primary);
    background: rgba(0, 0, 0, 0.75);
    z-index: 1000;
    display: flex;
    justify-content: center;
    align-items: center;
}

.custom-modal-container {
    background: var(--background-primary);
    color: var(--text-normal);
    padding: 20px;
    border-radius: 8px;
    max-width: 600px;
    width: 100%;
    box-shadow: var(--overlay-shadow);
}

.custom-modal-container h2 {
    margin-top: 0;
}

.custom-modal-container textarea {
    width: 100%;
    box-sizing: border-box;
    margin-bottom: 10px;
}

.modal-button-container {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
}

button.mod-cta {
    background-color: var(--interactive-accent);
    color: var(--text-on-accent);
}

button {
    padding: 8px 12px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
`;
document.head.appendChild(style);
```
