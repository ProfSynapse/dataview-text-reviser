```dataviewjs
// **Anthropic Revise Button**
// **IMPORTANT:** Replace 'YOUR_ANTHROPIC_API_KEY' with your actual Anthropic API key.
// **Never expose your API keys publicly.** If you've done so, revoke them immediately and generate new ones.

const ANTHROPIC_API_KEY = 'YOUR_ANTHROPIC_API_KEY'; // Replace with your Anthropic API key

let currentNoteView = null; // Keep track of the current note view
let reviseButton = null;

// Function to initialize the revise button
function initReviseButton() {
    document.addEventListener('selectionchange', handleSelectionChange);
    window.addEventListener('scroll', handleScroll);
}

// Handle selection changes
function handleSelectionChange() {
    const selection = window.getSelection();
    const selectedText = selection.toString().trim();

    if (selectedText.length > 0 && !reviseButton) {
        showReviseButton(selection);
    } else if (selectedText.length === 0 && reviseButton) {
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

    // Add pencil icon (SVG)
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
    reviseButton.style.opacity = '0.9';
    reviseButton.style.transition = 'opacity 0.3s';

    // Hover effect
    reviseButton.onmouseover = () => {
        reviseButton.style.opacity = '1';
    };
    reviseButton.onmouseout = () => {
        reviseButton.style.opacity = '0.9';
    };

    // Append the button to the body
    document.body.appendChild(reviseButton);

    // Position the button near the selected text
    repositionButton(selection);

    // Attach click event to the button
    reviseButton.onclick = async () => {
        const activeLeaf = dv.app.workspace.activeLeaf;
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
        // Create modal backdrop
        const modalBackdrop = document.createElement('div');
        modalBackdrop.className = 'custom-modal-overlay';

        // Create modal container
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

        const cancelButton = document.createElement('button');
        cancelButton.textContent = 'Cancel';
        cancelButton.className = 'mod-warning';

        buttonContainer.appendChild(submitButton);
        buttonContainer.appendChild(cancelButton);
        modalContainer.appendChild(buttonContainer);

        // Append modal to backdrop
        modalBackdrop.appendChild(modalContainer);
        document.body.appendChild(modalBackdrop);

        instructionTextarea.focus();

        // Event listeners
        const submitHandler = () => {
            const instructions = instructionTextarea.value.trim();
            if (instructions) {
                modalBackdrop.remove();
                resolve(instructions);
            } else {
                new Notice('Please enter some instructions.');
            }
        };

        const cancelHandler = () => {
            modalBackdrop.remove();
            resolve(null);
        };

        submitButton.addEventListener('click', submitHandler);
        cancelButton.addEventListener('click', cancelHandler);

        // Cleanup function to prevent memory leaks
        function cleanup() {
            submitButton.removeEventListener('click', submitHandler);
            cancelButton.removeEventListener('click', cancelHandler);
            modalBackdrop.remove();
        }

        // Close modal when clicking outside of it
        modalBackdrop.addEventListener('click', (e) => {
            if (e.target === modalBackdrop) {
                cleanup();
                resolve(null);
            }
        });
    });
}

// Function to revise text using Anthropic API
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
        const response = await fetch("https://api.anthropic.com/v1/messages", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "x-api-key": ANTHROPIC_API_KEY
            },
            body: JSON.stringify({
                "model": "claude-3",
                "messages": [
                    {
                        "role": "system",
                        "content": "You are an expert writer and editor. Revise the user's text according to their instructions, ensuring clarity, correctness, and adherence to the requested style and tone."
                    },
                    {
                        "role": "user",
                        "content": `Revise the following text based on these instructions: ${instructions}\n\nText:\n${selectedText}`
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
        const revisedText = data.completions?.[0]?.completion?.trim() || 'No revised text returned.';

        // Remove loading indicator
        loadingOverlay.remove();

        // Open modal to display revised text
        openRevisedTextModal(revisedText, editor);
    } catch (error) {
        console.error("Error calling Anthropic API:", error);
        new Notice(`Error: ${error.message}`);
        loadingOverlay.remove(); // Remove modal in case of error
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
    revisedTextLabel.textContent = 'Revised Text (you can edit it before copying or replacing):';
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
    copyButton.className = 'mod-cta';

    const replaceButton = document.createElement('button');
    replaceButton.textContent = 'Replace Selection';
    replaceButton.className = 'mod-cta';

    const reviseAgainButton = document.createElement('button');
    reviseAgainButton.textContent = 'Revise Again';
    reviseAgainButton.className = 'mod-cta';

    const cancelButton = document.createElement('button');
    cancelButton.textContent = 'Cancel';
    cancelButton.className = 'mod-warning';

    buttonContainer.appendChild(copyButton);
    buttonContainer.appendChild(replaceButton);
    buttonContainer.appendChild(reviseAgainButton);
    buttonContainer.appendChild(cancelButton);
    modalContainer.appendChild(buttonContainer);

    // Append modal to overlay
    modalOverlay.appendChild(modalContainer);
    document.body.appendChild(modalOverlay);

    // Event handlers
    const copyHandler = () => {
        navigator.clipboard.writeText(revisedTextarea.value);
        new Notice('Revised text copied to clipboard.');
    };

    const replaceHandler = () => {
        const selectedRange = editor.getSelectionRange();
        if (selectedRange) {
            editor.replaceSelection(revisedTextarea.value);
        } else {
            // Replace entire note content if no text is selected
            editor.setValue(revisedTextarea.value);
        }
        new Notice('Text replaced.');
        cleanup();
    };

    const reviseAgainHandler = async () => {
        cleanup();
        const instructions = await openInstructionModal();
        if (instructions) {
            await reviseText(revisedTextarea.value, instructions, editor);
        }
    };

    const cancelHandler = () => {
        cleanup();
    };

    copyButton.addEventListener('click', copyHandler);
    replaceButton.addEventListener('click', replaceHandler);
    reviseAgainButton.addEventListener('click', reviseAgainHandler);
    cancelButton.addEventListener('click', cancelHandler);

    // Cleanup function to prevent memory leaks
    function cleanup() {
        copyButton.removeEventListener('click', copyHandler);
        replaceButton.removeEventListener('click', replaceHandler);
        reviseAgainButton.removeEventListener('click', reviseAgainHandler);
        cancelButton.removeEventListener('click', cancelHandler);
        modalOverlay.remove();
    }

    // Close modal when clicking outside of it
    modalOverlay.addEventListener('click', (e) => {
        if (e.target === modalOverlay) {
            cleanup();
        }
    });
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

button.mod-warning {
    background-color: var(--warning);
    color: var(--text-on-warning);
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
