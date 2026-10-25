const chatLog = document.getElementById('chat-log');
const plusButton = document.getElementById('plus-button');
const plusMenu = document.getElementById('plus-menu');
const promptInput = document.getElementById('prompt-input');
const sendButton = document.getElementById('send-button');
const modeIndicator = document.getElementById('mode-indicator');
const attachmentsWrap = document.getElementById('attachments');

const state = {
  mode: 'Ask question',
  attachments: []
};

const modeButtons = [...document.querySelectorAll('.mode-option')];

function setActiveMode(nextMode) {
  state.mode = nextMode;
  modeIndicator.textContent = nextMode;
  modeButtons.forEach((button) => {
    const active = button.dataset.mode === nextMode;
    button.classList.toggle('active', active);
  });
}

function togglePlusMenu() {
  plusMenu.classList.toggle('hidden');
}

function appendMessage(role, content) {
  const article = document.createElement('article');
  article.className = `message ${role}`;

  if (role === 'assistant') {
    article.innerHTML = `
      <div class="avatar">AI</div>
      <div class="bubble wide">${content}</div>
    `;
  } else {
    article.innerHTML = `
      <div class="bubble">${content}</div>
    `;
  }

  chatLog.appendChild(article);
  chatLog.scrollTop = chatLog.scrollHeight;
}

function renderAttachments() {
  attachmentsWrap.innerHTML = '';

  state.attachments.forEach((file) => {
    const tag = document.createElement('span');
    tag.className = 'attachment-tag';
    tag.textContent = file.name;
    attachmentsWrap.appendChild(tag);
  });
}

function handleFileSelection(event) {
  const files = [...event.target.files];
  if (!files.length) return;

  files.forEach((file) => {
    const duplicate = state.attachments.some((item) => item.name === file.name && item.size === file.size);
    if (!duplicate) {
      state.attachments.push(file);
    }
  });

  renderAttachments();
  event.target.value = '';
  plusMenu.classList.add('hidden');
}

function generateResponse(prompt, mode, attachments) {
  const attachmentSummary = attachments.length
    ? `
      Included context from ${attachments.length} attachment${attachments.length > 1 ? 's' : ''}: ${attachments.map((file) => file.name).join(', ')}.
    `
    : '';

  if (mode === 'Ask question') {
    return `Here is the direction I would recommend: ${prompt}. ${attachmentSummary}I would break this into user goals, interface requirements, frontend structure, and the logic needed for a polished experience. The result would be a focused, high-conversion design with strong UX, clean content hierarchy, and a practical implementation plan.`;
  }

  if (mode === 'Create website') {
    return `Website build plan: create a premium landing experience around “${prompt}” with a narrative hero section, conversion-focused CTAs, feature grid, pricing-free messaging, and responsive layout. ${attachmentSummary}I would structure pages for product story, feature highlights, proof points, and a final CTA while keeping the system flexible for custom content and dynamic sections.`;
  }

  return `Application build plan: design a sophisticated app for “${prompt}” with modular architecture, a strong data model, secure flows, and polished UI states. ${attachmentSummary}I would map out the feature set, define the key screens, wire up the core app logic, and prepare a production-ready implementation plan for both the frontend and backend requirements.`;
}

plusButton.addEventListener('click', togglePlusMenu);
modeButtons.forEach((button) => {
  button.addEventListener('click', () => {
    setActiveMode(button.dataset.mode);
    plusMenu.classList.add('hidden');
  });
});

['image-upload', 'document-upload', 'audio-upload', 'model-upload'].forEach((id) => {
  document.getElementById(id).addEventListener('change', handleFileSelection);
});

sendButton.addEventListener('click', () => {
  const prompt = promptInput.value.trim();
  if (!prompt) {
    promptInput.focus();
    return;
  }

  appendMessage('user', prompt);
  const response = generateResponse(prompt, state.mode, state.attachments);
  appendMessage('assistant', response);

  promptInput.value = '';
  state.attachments = [];
  renderAttachments();
});

promptInput.addEventListener('keydown', (event) => {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault();
    sendButton.click();
  }
});

setActiveMode(state.mode);
renderAttachments();
