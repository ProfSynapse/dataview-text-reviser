# LM STUDIO
```dataviewjs
// Define the local LM Studio server URL and the model you want to use
const LM_STUDIO_URL = 'http://localhost:1234/v1/chat/completions'; // Adjust the port if necessary
const MODEL_ID = 'TheBloke/phi-2-GGUF/phi-2.Q4_K_S.gguf'; // Replace with your loaded model's ID

let currentNoteView = null; // Keep track of the current note view

// Function to initialize the revise button
function initReviseButton() {
    updateReviseButtonVisibility(); // Initial check

    // Set up event listener for active leaf changes
    app.workspace.on('active-leaf-change', updateReviseButtonVisibility);
}

// Function to check if the note contains the DataviewJS block
function noteContainsDataviewJS(noteView) {
    const content = noteView.data;
    return content.includes('```dataviewjs');
}

// Function to create the revise button and attach it to the note container
function createReviseButton(noteContainer, noteView) {
    // Check if the button already exists in the note container
    if (noteContainer.querySelector('#reviseButton')) {
        return; // Button already exists, no need to add it again
    }

    // Create the button
    let reviseButton = document.createElement('button');
    reviseButton.id = 'reviseButton';
    reviseButton.classList.add('clickable-icon');
    reviseButton.setAttribute('aria-label', 'Revise Selected Text');
    reviseButton.style.position = 'absolute';
    reviseButton.style.bottom = '20px';
    reviseButton.style.left = '50%';
    reviseButton.style.transform = 'translateX(-50%)';
    reviseButton.style.zIndex = '1000';
    reviseButton.style.width = '50px';
    reviseButton.style.height = '50px';
    reviseButton.style.borderRadius = '50%';
    reviseButton.style.backgroundColor = getComputedStyle(document.body).getPropertyValue('--interactive-accent');
    reviseButton.style.color = getComputedStyle(document.body).getPropertyValue('--text-on-accent');
    reviseButton.style.display = 'flex';
    reviseButton.style.justifyContent = 'center';
    reviseButton.style.alignItems = 'center';
    reviseButton.style.fontSize = '24px';
    reviseButton.style.boxShadow = '0 4px 6px rgba(0, 0, 0, 0.1)';
    reviseButton.style.cursor = 'pointer';

    // Add an icon to the button
    reviseButton.innerHTML = `
        <svg viewBox="0 0 24 24" fill="currentColor" width="28" height="28">
            <path d="M3 17.25V21h3.75L17.81,9.94L14.06,6.19L3,17.25zM20.71,7.04c0.39-0.39,0.39-1.03,0-1.42l-2.34-2.34c-0.39-0.39-1.03-0.39-1.42,0l-1.83,1.83l3.75,3.75L20.71,7.04z"/>
        </svg>
    `;

    // Tooltip
    reviseButton.title = 'Revise Selected Text';

    // Event handler
    const buttonClickHandler = async () => {
        const editor = noteView.editor;
        let selectedText = editor.getSelection();
        if (!selectedText) {
            // If no text is selected, use the entire note content
            selectedText = editor.getValue();
        }

        const instructions = await openInstructionModal(selectedText);
        if (instructions) {
            await reviseText(selectedText, instructions);
        }
    };

    reviseButton.addEventListener('click', buttonClickHandler);

    // Store the handler so we can remove it later
    reviseButton.buttonClickHandler = buttonClickHandler;

    // Append the button to the note container
    noteContainer.appendChild(reviseButton);

    // Store the button in the note view for later removal
    noteView.reviseButton = reviseButton;
}

// Function to remove the revise button from the note container
function removeReviseButton(noteView) {
    if (noteView && noteView.reviseButton) {
        let reviseButton = noteView.reviseButton;
        reviseButton.removeEventListener('click', reviseButton.buttonClickHandler);
        if (reviseButton.parentNode) {
            reviseButton.parentNode.removeChild(reviseButton);
        }
        delete noteView.reviseButton;
    }
}

// Function to update the revise button visibility based on the active note
function updateReviseButtonVisibility() {
    const activeLeaf = app.workspace.activeLeaf;

    // Ensure we're in a markdown view
    if (activeLeaf && activeLeaf.view.getViewType() === "markdown") {
        const noteView = activeLeaf.view;
        const noteContainer = noteView.containerEl;

        // Remove the button from the previous note if the note has changed
        if (
            currentNoteView &&
            currentNoteView.file &&
            noteView.file &&
            currentNoteView.file.path !== noteView.file.path
        ) {
            removeReviseButton(currentNoteView);
        }

        // Check if the current note contains the DataviewJS block
        if (noteContainsDataviewJS(noteView)) {
            // Create the revise button and attach it to the note container
            createReviseButton(noteContainer, noteView);
        } else {
            // Remove the button if it exists in the current note
            removeReviseButton(noteView);
        }

        // Update currentNoteView to the new noteView
        currentNoteView = noteView;
    } else {
        // Not in markdown view, remove button if it exists
        if (currentNoteView) {
            removeReviseButton(currentNoteView);
            currentNoteView = null;
        }
    }
}

// Function to open a custom modal to prompt for revision instructions and display results
function openInstructionModal(selectedText, previousInstructions = '') {
    return new Promise((resolve) => {
        // Create modal backdrop
        const modalBackdrop = document.createElement('div');
        modalBackdrop.style.position = 'fixed';
        modalBackdrop.style.top = '0';
        modalBackdrop.style.left = '0';
        modalBackdrop.style.width = '100%';
        modalBackdrop.style.height = '100%';
        modalBackdrop.style.backgroundColor = 'rgba(0, 0, 0, 0.5)';
        modalBackdrop.style.zIndex = '1000';

        // Create modal container
        const modal = document.createElement('div');
        modal.classList.add('modal', 'mod-obsidian'); // Use Obsidian's CSS classes
        modal.style.position = 'fixed';
        modal.style.top = '50%';
        modal.style.left = '50%';
        modal.style.transform = 'translate(-50%, -50%)';
        modal.style.padding = '20px';
        modal.style.borderRadius = '8px';
        modal.style.zIndex = '1001';
        modal.style.width = '60%';
        modal.style.maxWidth = '600px';
        modal.style.maxHeight = '80%';
        modal.style.overflowY = 'auto';
        modal.style.boxShadow = '0 4px 6px rgba(0, 0, 0, 0.1)';

        // Instruction label and input
        const instructionLabel = document.createElement('label');
        instructionLabel.textContent = 'Enter revision instructions:';
        instructionLabel.style.display = 'block';
        instructionLabel.style.marginBottom = '10px';

        const instructionInput = document.createElement('textarea');
        instructionInput.style.width = '100%';
        instructionInput.style.height = '80px';
        instructionInput.style.marginBottom = '15px';
        instructionInput.style.padding = '5px';
        instructionInput.classList.add('input');
        instructionInput.setAttribute('aria-label', 'Revision Instructions');
        instructionInput.value = previousInstructions;

        // Action buttons
        const buttonContainer = document.createElement('div');
        buttonContainer.style.textAlign = 'right';

        const submitButton = document.createElement('button');
        submitButton.textContent = 'Submit';
        submitButton.style.marginRight = '10px';
        submitButton.classList.add('mod-cta');

        const cancelButton = document.createElement('button');
        cancelButton.textContent = 'Cancel';
        cancelButton.classList.add('mod-warning');

        buttonContainer.appendChild(submitButton);
        buttonContainer.appendChild(cancelButton);

        // Append elements to modal
        modal.appendChild(instructionLabel);
        modal.appendChild(instructionInput);
        modal.appendChild(buttonContainer);

        // Append modal to backdrop
        modalBackdrop.appendChild(modal);
        document.body.appendChild(modalBackdrop);

        instructionInput.focus();

        // Event listeners
        const submitHandler = () => {
            const instructions = instructionInput.value.trim();
            if (instructions) {
                // Show loading indicator
                modalBackdrop.remove();
                resolve({ instructions, selectedText });
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

// Function to revise text using LM Studio local model
async function reviseText(text, instructionsObj) {
    const { instructions, selectedText } = instructionsObj;

    // Create a modal to display the revised text
    const modalBackdrop = document.createElement('div');
    modalBackdrop.style.position = 'fixed';
    modalBackdrop.style.top = '0';
    modalBackdrop.style.left = '0';
    modalBackdrop.style.width = '100%';
    modalBackdrop.style.height = '100%';
    modalBackdrop.style.backgroundColor = 'rgba(0,0,0,0.5)';
    modalBackdrop.style.zIndex = '1000';

    const modal = document.createElement('div');
    modal.classList.add('modal', 'mod-obsidian');
    modal.style.position = 'fixed';
    modal.style.top = '50%';
    modal.style.left = '50%';
    modal.style.transform = 'translate(-50%, -50%)';
    modal.style.padding = '20px';
    modal.style.borderRadius = '8px';
    modal.style.zIndex = '1001';
    modal.style.width = '60%';
    modal.style.maxWidth = '600px';
    modal.style.maxHeight = '80%';
    modal.style.overflowY = 'auto';
    modal.style.boxShadow = '0 4px 6px rgba(0,0,0,0.1)';

    // Loading indicator
    const loadingIndicator = document.createElement('div');
    loadingIndicator.textContent = 'Revising...';
    loadingIndicator.style.marginBottom = '15px';
    modal.appendChild(loadingIndicator);

    // Append modal to backdrop
    modalBackdrop.appendChild(modal);
    document.body.appendChild(modalBackdrop);

    try {
        const response = await fetch(LM_STUDIO_URL, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
                // No Authorization header needed for local LM Studio server
            },
            body: JSON.stringify({
                model: MODEL_ID, // Specify the loaded model
                messages: [
                    { role: "system", content: "You are a helpful assistant that revises text based on user instructions." },
                    { role: "user", content: `Revise the following text based on these instructions: ${instructions}\n\nText:\n${text}` }
                ],
                max_tokens: 500,
                temperature: 0.7
            })
        });

        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(`Error ${response.status}: ${errorData.error.message || 'Unknown error'}`);
        }

        const data = await response.json();
        const revisedText = data.choices[0].message.content.trim();

        // Remove loading indicator
        loadingIndicator.remove();

        // Display revised text in textarea
        const revisedLabel = document.createElement('label');
        revisedLabel.textContent = 'Revised Text (you can edit it before copying or replacing):';
        revisedLabel.style.display = 'block';
        revisedLabel.style.marginBottom = '10px';

        const revisedTextArea = document.createElement('textarea');
        revisedTextArea.style.width = '100%';
        revisedTextArea.style.height = '200px';
        revisedTextArea.style.marginBottom = '15px';
        revisedTextArea.style.padding = '5px';
        revisedTextArea.classList.add('input');
        revisedTextArea.value = revisedText;

        const buttonContainer = document.createElement('div');
        buttonContainer.style.textAlign = 'right';

        const copyButton = document.createElement('button');
        copyButton.textContent = 'Copy Revised Text';
        copyButton.style.marginRight = '10px';
        copyButton.classList.add('mod-cta');

        const replaceButton = document.createElement('button');
        replaceButton.textContent = 'Replace Text';
        replaceButton.style.marginRight = '10px';
        replaceButton.classList.add('mod-cta');

        const reviseAgainButton = document.createElement('button');
        reviseAgainButton.textContent = 'Revise Again';
        reviseAgainButton.style.marginRight = '10px';
        reviseAgainButton.classList.add('mod-cta');

        const cancelButton = document.createElement('button');
        cancelButton.textContent = 'Cancel';
        cancelButton.classList.add('mod-warning');

        buttonContainer.appendChild(copyButton);
        buttonContainer.appendChild(replaceButton);
        buttonContainer.appendChild(reviseAgainButton);
        buttonContainer.appendChild(cancelButton);

        modal.appendChild(revisedLabel);
        modal.appendChild(revisedTextArea);
        modal.appendChild(buttonContainer);

        // Event handlers
        const copyHandler = () => {
            navigator.clipboard.writeText(revisedTextArea.value);
            new Notice('Revised text copied to clipboard.');
        };

        const replaceHandler = () => {
            const activeLeaf = app.workspace.activeLeaf;
            if (activeLeaf && activeLeaf.view.getViewType() === 'markdown') {
                const editor = activeLeaf.view.editor;
                const selectedText = editor.getSelection();
                if (selectedText) {
                    editor.replaceSelection(revisedTextArea.value);
                } else {
                    // Replace entire note content if no text is selected
                    editor.setValue(revisedTextArea.value);
                }
                new Notice('Text replaced.');
                cleanup();
            } else {
                new Notice('No active markdown editor found.');
            }
        };

        const reviseAgainHandler = async () => {
            cleanup();
            const instructions = await openInstructionModal(revisedTextArea.value, instructions);
            if (instructions) {
                await reviseText(revisedTextArea.value, instructions);
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
            modalBackdrop.remove();
        }

        // Close modal when clicking outside of it
        modalBackdrop.addEventListener('click', (e) => {
            if (e.target === modalBackdrop) {
                cleanup();
            }
        });
    } catch (error) {
        console.error("Error calling LM Studio API:", error);
        new Notice(`Error: ${error.message}`);
        modalBackdrop.remove(); // Remove modal in case of error
    }
}

// Function to display revised text (optional if using modals)
function displayRevisedText(revisedText) {
    const revisedContainer = dv.el('div', '');

    const revisedParagraph = dv.el('p', revisedText);
    revisedParagraph.style.marginBottom = '10px';
    const copyButton = dv.el('button', 'Copy Revised Text');
    const replaceButton = dv.el('button', 'Replace Selected Text');
    const reviseAgainButton = dv.el('button', 'Revise Again');

    copyButton.style.marginRight = '10px';
    replaceButton.style.marginRight = '10px';

    copyButton.addEventListener('click', () => {
        navigator.clipboard.writeText(revisedText);
        new Notice("Revised text copied to clipboard.");
    });

    replaceButton.addEventListener('click', () => {
        const activeLeaf = app.workspace.activeLeaf;
        if (activeLeaf && activeLeaf.view.getViewType() === "markdown") {
            const editor = activeLeaf.view.editor;
            const selectedText = editor.getSelection();
            if (selectedText) {
                editor.replaceSelection(revisedText);
                new Notice("Selected text replaced.");
            } else {
                new Notice("No text selected to replace.");
            }
        } else {
            new Notice("No active markdown editor found.");
        }
    });

    reviseAgainButton.addEventListener('click', () => {
        openInstructionModal(revisedText);
    });

    revisedContainer.appendChild(revisedParagraph);
    revisedContainer.appendChild(copyButton);
    revisedContainer.appendChild(replaceButton);
    revisedContainer.appendChild(reviseAgainButton);

    const existingRevised = document.getElementById('revisedText');
    if (existingRevised) {
        existingRevised.replaceWith(revisedContainer);
    } else {
        dv.container.appendChild(revisedContainer);
    }
    revisedContainer.id = 'revisedText';
}

// Initialize the revise button when the script is loaded
initReviseButton();
```
