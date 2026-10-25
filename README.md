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
    goal: 'Clarify the user need, product intent, and the highest-value problem to solve first.',
    stack: 'UX framing, journey mapping, clear decision points, and a realistic blueprint for execution.',
    steps: [
      'Map the user intent and the business value.',
      'Define the must-have screens and key decisions.',
      'Prioritize a lean but high-impact first release.'
    ],
    previewLabel: 'Conversational product map'
  },
  'Create website': {
    goal: 'Design a premium website experience with a persuasive story, clarity, and strong conversion flow.',
    stack: 'Responsive layout system, content sequencing, high-clarity sections, and scalable design patterns.',
    steps: [
      'Define the hero, value proposition, and trust-building messages.',
      'Design the flow of sections and calls to action.',
      'Prepare the modular page system for implementation and iteration.'
    ],
    previewLabel: 'Premium landing system'
  },
  'Build application': {
    goal: 'Turn the idea into a sophisticated app with clear logic, flows, and scalable architecture.',
    stack: 'Feature planning, module design, state flows, data structure, and polished product experience.',
    steps: [
      'Map the main user flows and product requirements.',
      'Design the key screens and interaction patterns.',
      'Plan validation, data logic, and production-ready architecture.'
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

  if (!state.attachments.length) return;

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

function updateBlueprint(prompt, mode) {
  const selected = modeData[mode] || modeData['Ask question'];
  goalText.textContent = selected.goal;
  stackText.textContent = selected.stack;
  planList.innerHTML = selected.steps.map((step) => `<li>${step}</li>`).join('');

  const shortPrompt = prompt.length > 54 ? `${prompt.slice(0, 54)}…` : prompt;
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
    <div class="preview-summary">${shortPrompt}</div>
  `;
}

function buildAssistantResponse(prompt, mode, attachments) {
  const mediaLine = attachments.length
    ? `I also reviewed ${attachments.length} attachment${attachments.length > 1 ? 's' : ''}: ${attachments.map((file) => file.name).join(', ')}.`
    : 'No attachments were included, so this plan is based on your prompt alone.';

  const selected = modeData[mode] || modeData['Ask question'];

  return `
    <div>
      I’m shaping this around: <strong>${prompt}</strong>
    </div>
    <div class="response-block">
      <span class="response-title">Understanding</span>
      <ul class="response-list">
        <li>${selected.goal}</li>
        <li>${mediaLine}</li>
      </ul>
    </div>
    <div class="response-block">
      <span class="response-title">Recommended structure</span>
      <ul class="response-list">
        <li>Start with the product promise and the most important user journey.</li>
        <li>Design the key UI blocks with strong hierarchy and clear information flow.</li>
        <li>Use structured sections and reusable logic to keep the build scalable.</li>
      </ul>
    </div>
    <div class="response-block">
      <span class="response-title">Execution plan</span>
      <ul class="response-list">
        <li>${selected.stack}</li>
        <li>Validate the concept against clarity, usability, and speed.</li>
        <li>Build in iterative phases so the core value lands first.</li>
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
  updateBlueprint(prompt, state.mode);
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
updateBlueprint('Define a premium digital experience for a modern AI product.', state.mode);
