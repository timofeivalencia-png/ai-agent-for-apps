const chatLog = document.getElementById('chat-log');
const plusButton = document.getElementById('plus-button');
const plusMenu = document.getElementById('plus-menu');
const promptInput = document.getElementById('prompt-input');
const sendButton = document.getElementById('send-button');
const modeIndicator = document.getElementById('mode-indicator');
const attachmentsWrap = document.getElementById('attachments');
const goalText = document.getElementById('goal-text');
const stackText = document.getElementById('stack-text');
const planList = document.getElementById('plan-list');
const previewLayout = document.getElementById('preview-layout');

const state = {
  mode: 'Ask question',
  attachments: []
};

const modeButtons = [...document.querySelectorAll('.mode-option')];

const modeData = {
  'Ask question': {
    goal: 'Clarify the core user need, product direction, and the highest-value problem to solve.',
    stack: 'UX framing, journey mapping, clear decision points, and a realistic execution blueprint.',
    steps: [
      'Map the user intent and business value.',
      'Define the must-have screens and core interactions.',
      'Prioritize a minimal but high-impact first build.'
    ],
    previewLabel: 'Conversational product map'
  },
  'Create website': {
    goal: 'Design a premium website experience with strong narrative structure and conversion focus.',
    stack: 'Responsive layout system, visual storytelling, CTA strategy, and scalable section patterns.',
    steps: [
      'Define the hero, value proposition, and trust-building sections.',
      'Build a modular page system with clear hierarchy and flow.',
      'Finalize content blocks, responsive logic, and call-to-action placement.'
    ],
    previewLabel: 'Premium landing system'
  },
  'Build application': {
    goal: 'Turn the idea into an application with clear logic, workflows, and scalable product architecture.',
    stack: 'Feature planning, module design, state management, data flow, and polished UX states.',
    steps: [
      'Map the primary user flows and critical decision points.',
      'Design the core modules and app screens.',
      'Plan the backend logic, validation, and production-ready states.'
    ],
    previewLabel: 'App architecture preview'
  }
};

function setActiveMode(nextMode) {
  state.mode = nextMode;
  modeIndicator.textContent = nextMode;
  modeButtons.forEach((button) => {
    button.classList.toggle('active', button.dataset.mode === nextMode);
  });
}

function renderAttachments() {
  attachmentsWrap.innerHTML = '';

  if (!state.attachments.length) {
    return;
  }

  state.attachments.forEach((file) => {
    const tag = document.createElement('span');
    tag.className = 'attachment-tag';
    tag.textContent = file.name;
    attachmentsWrap.appendChild(tag);
  });
}

function appendMessage(role, htmlContent) {
  const article = document.createElement('article');
  article.className = `message ${role}`;

  if (role === 'assistant') {
    article.innerHTML = `
      <div class="avatar">AI</div>
      <div class="bubble wide">${htmlContent}</div>
    `;
  } else {
    article.innerHTML = `
      <div class="bubble">${htmlContent}</div>
    `;
  }

  chatLog.appendChild(article);
  chatLog.scrollTop = chatLog.scrollHeight;
}

function showTypingIndicator() {
  const article = document.createElement('article');
  article.className = 'message assistant';
  article.innerHTML = `
    <div class="avatar">AI</div>
    <div class="bubble wide">
      <div class="typing-indicator" aria-label="Agent is typing">
        <span></span><span></span><span></span>
      </div>
    </div>
  `;
  chatLog.appendChild(article);
  chatLog.scrollTop = chatLog.scrollHeight;
  return article;
}

function togglePlusMenu() {
  plusMenu.classList.toggle('hidden');
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

function updatePlaneDetails(prompt, mode) {
  const selected = modeData[mode] || modeData['Ask question'];
  goalText.textContent = selected.goal;
  stackText.textContent = selected.stack;
  planList.innerHTML = selected.steps.map((step) => `<li>${step}</li>`).join('');

  const safePrompt = prompt.length > 54 ? `${prompt.slice(0, 54)}…` : prompt;
  previewLayout.innerHTML = `
    <div class="mini-preview top">
      <div class="mini-preview-header"></div>
      <div class="mini-preview-body"></div>
    </div>
    <div class="mini-preview-grid">
      <div class="mini-preview small"></div>
      <div class="mini-preview small"></div>
      <div class="mini-preview small"></div>
    </div>
    <div class="preview-label">${selected.previewLabel}</div>
    <div class="preview-summary">${safePrompt}</div>
  `;
}

function buildAssistantResponse(prompt, mode, attachments) {
  const mediaLine = attachments.length
    ? `I also reviewed ${attachments.length} attachment${attachments.length > 1 ? 's' : ''}: ${attachments.map((file) => file.name).join(', ')}.`
    : 'No attachments were provided, so this is based on your prompt alone.';

  const details = modeData[mode] || modeData['Ask question'];

  return `
    <div>
      I’m shaping this around: <strong>${prompt}</strong>
    </div>
    <div class="response-block">
      <span class="response-title">Understanding</span>
      <ul class="response-list">
        <li>${details.goal}</li>
        <li>${mediaLine}</li>
      </ul>
    </div>
    <div class="response-block">
      <span class="response-title">Recommended structure</span>
      <ul class="response-list">
        <li>Start with the user journey and the clearest product promise.</li>
        <li>Design the main screens with strong visual hierarchy and a calm information flow.</li>
        <li>Use structured sections, reusable components, and resilient interactions.</li>
      </ul>
    </div>
    <div class="response-block">
      <span class="response-title">Execution plan</span>
      <ul class="response-list">
        <li>${details.stack}</li>
        <li>Validate the experience against clarity, trust, and speed.</li>
        <li>Build in phases so the most important functionality lands first.</li>
      </ul>
    </div>
  `;
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
  updatePlaneDetails(prompt, state.mode);
  const typing = showTypingIndicator();

  window.setTimeout(() => {
    typing.remove();
    appendMessage('assistant', buildAssistantResponse(prompt, state.mode, state.attachments));
    promptInput.value = '';
    state.attachments = [];
    renderAttachments();
  }, 650);
});

promptInput.addEventListener('keydown', (event) => {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault();
    sendButton.click();
  }
});

setActiveMode(state.mode);
renderAttachments();
updatePlaneDetails('Define a premium digital experience for a modern AI product.', state.mode);
