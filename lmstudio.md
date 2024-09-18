# LM STUDIO
```dataviewjs
// Function to create the revise button and attach it to the note editor
function createReviseButton() {
    const activeLeaf = app.workspace.activeLeaf;
    if (activeLeaf && activeLeaf.view.getViewType() === "markdown") {
        const noteEditor = activeLeaf.view.containerEl;

        let reviseButton = document.createElement('button');
        reviseButton.id = 'reviseButton';
        reviseButton.textContent = 'Revise Selected Text';
        reviseButton.style.position = 'absolute';
        reviseButton.style.top = '40px';
        reviseButton.style.right = '20px';
        reviseButton.style.zIndex = '1000';

        reviseButton.addEventListener('click', () => {
            const editor = activeLeaf.view.editor;
            const selectedText = editor.getSelection();
            if (selectedText) {
                openInstructionModal(selectedText);
            } else {
                alert("Please select some text in the editor first.");
            }
        });

        noteEditor.appendChild(reviseButton);
    }
}

// Function to remove the revise button
function removeReviseButton() {
    const reviseButton = document.getElementById('reviseButton');
    if (reviseButton) {
        reviseButton.remove();
    }
}

// Function to check if the current note contains the DataviewJS block
function noteContainsDataviewJS() {
    const activeLeaf = app.workspace.activeLeaf;
    if (activeLeaf && activeLeaf.view.getViewType() === "markdown") {
        const content = activeLeaf.view.data;
        return content.includes('```dataviewjs');
    }
    return false;
}

// Function to update the revise button visibility
function updateReviseButtonVisibility() {
    if (noteContainsDataviewJS()) {
        createReviseButton();
    } else {
        removeReviseButton();
    }
}

// Function to open a custom modal to prompt for revision instructions
function openInstructionModal(selectedText) {
    const modalBackdrop = document.createElement('div');
    modalBackdrop.style.position = 'fixed';
    modalBackdrop.style.top = '0';
    modalBackdrop.style.left = '0';
    modalBackdrop.style.width = '100%';
    modalBackdrop.style.height = '100%';
    modalBackdrop.style.backgroundColor = 'rgba(0, 0, 0, 0.5)';
    modalBackdrop.style.zIndex = '1000';
    
    const modal = document.createElement('div');
    modal.style.position = 'fixed';
    modal.style.top = '50%';
    modal.style.left = '50%';
    modal.style.transform = 'translate(-50%, -50%)';
    modal.style.backgroundColor = getComputedStyle(document.body).getPropertyValue('--background-primary') || '#fff';
    modal.style.color = getComputedStyle(document.body).getPropertyValue('--text-normal') || '#000';
    modal.style.padding = '20px';
    modal.style.borderRadius = '8px';
    modal.style.zIndex = '1001';
    modal.style.width = '60%';
    modal.style.maxWidth = '600px';
    modal.style.boxShadow = '0 4px 6px rgba(0, 0, 0, 0.1)';
    
    const instructionLabel = document.createElement('label');
    instructionLabel.textContent = 'Enter revision instructions:';
    instructionLabel.style.display = 'block';
    instructionLabel.style.marginBottom = '10px';
    
    const instructionInput = document.createElement('textarea');
    instructionInput.style.width = '100%';
    instructionInput.style.height = '100px';
    instructionInput.style.marginBottom = '15px';
    instructionInput.style.padding = '5px';
    instructionInput.style.backgroundColor = getComputedStyle(document.body).getPropertyValue('--background-secondary') || '#f0f0f0';
    instructionInput.style.color = getComputedStyle(document.body).getPropertyValue('--text-normal') || '#000';
    instructionInput.style.border = '1px solid ' + (getComputedStyle(document.body).getPropertyValue('--background-modifier-border') || '#ddd');
    instructionInput.style.borderRadius = '4px';
    
    const buttonContainer = document.createElement('div');
    buttonContainer.style.textAlign = 'right';
    
    const submitButton = document.createElement('button');
    submitButton.textContent = 'Submit';
    submitButton.style.marginRight = '10px';
    submitButton.style.padding = '5px 10px';
    submitButton.style.backgroundColor = getComputedStyle(document.body).getPropertyValue('--interactive-accent') || '#5c7cfa';
    submitButton.style.color = getComputedStyle(document.body).getPropertyValue('--text-on-accent') || '#fff';
    submitButton.style.border = 'none';
    submitButton.style.borderRadius = '4px';
    submitButton.style.cursor = 'pointer';

    const cancelButton = document.createElement('button');
    cancelButton.textContent = 'Cancel';
    cancelButton.style.padding = '5px 10px';
    cancelButton.style.backgroundColor = getComputedStyle(document.body).getPropertyValue('--background-modifier-error') || '#ff5555';
    cancelButton.style.color = getComputedStyle(document.body).getPropertyValue('--text-on-accent') || '#fff';
    cancelButton.style.border = 'none';
    cancelButton.style.borderRadius = '4px';
    cancelButton.style.cursor = 'pointer';
    
    buttonContainer.appendChild(submitButton);
    buttonContainer.appendChild(cancelButton);
    
    modal.appendChild(instructionLabel);
    modal.appendChild(instructionInput);
    modal.appendChild(buttonContainer);
    modalBackdrop.appendChild(modal);
    document.body.appendChild(modalBackdrop);
    
    submitButton.addEventListener('click', function() {
        const instructions = instructionInput.value.trim();
        if (instructions) {
            document.body.removeChild(modalBackdrop);
            reviseText(selectedText, instructions);
        } else {
            alert('Please enter some instructions.');
        }
    });
    
    cancelButton.addEventListener('click', function() {
        document.body.removeChild(modalBackdrop);
    });
}

// Function to revise text using LM Studio local model
async function reviseText(text, instructions) {
    try {
        const response = await fetch("http://localhost:1234/v1/chat/completions", { // Adjust port as needed
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                "messages": [
                    {"role": "system", "content": "You are a helpful assistant that revises text based on user instructions."},
                    {"role": "user", "content": `Revise the following text based on these instructions: ${instructions}\n\nText: ${text}`}
                ]
            })
        });

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        const revisedText = data.choices[0].message.content.trim();
        displayRevisedText(revisedText);
    } catch (error) {
        console.error("Error calling LM Studio API:", error);
        dv.el("p", `Error: ${error.message}`, {attr: {style: "color: red;"}});
    }
}

// Function to display revised text
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
        alert("Revised text copied to clipboard.");
    });

    replaceButton.addEventListener('click', () => {
        const activeLeaf = app.workspace.activeLeaf;
        if (activeLeaf && activeLeaf.view.getViewType() === "markdown") {
            const editor = activeLeaf.view.editor;
            const selectedText = editor.getSelection();
            if (selectedText) {
                editor.replaceSelection(revisedText);
                alert("Selected text replaced.");
            } else {
                alert("No text selected to replace.");
            }
        } else {
            alert("No active markdown editor found.");
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

// Main execution
updateReviseButtonVisibility();

// Set up event listener for active leaf changes
app.workspace.on('active-leaf-change', updateReviseButtonVisibility);
```
