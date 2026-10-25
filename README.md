const chatLog = document.getElementById('chat-log');
const plusButton = document.getElementById('plus-button');
const plusMenu = document.getElementById('plus-menu');
const promptInput = document.getElementById('prompt-input');
const sendButton = document.getElementById('send-button');
const modeIndicator = document.getElementById('mode-indicator');
const attachmentsWrap = document.getElementById('attachments');

const goalText = document.getElementById('goal-text');
const stackText = document.getElementById('stack-text');
const deliverableText = document.getElementById('deliverable-text');

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

function buildAssistantResponse(prompt, mode, attachments) {
  const mediaText = attachments.length
    ? `I also reviewed ${attachments.length} attachment${attachments.length > 1 ? 's' : ''}: ${attachments.map((file) => file.name).join(', ')}.`
    : 'No attachments were included, so I am building from your prompt alone.';

  const strategy = {
    'Ask question': {
      goal: 'Clarify the product intent, user needs, and conversion path before designing the interface.',
      stack: 'UX framing, component architecture, interaction mapping, and a realistic implementation plan.',
      deliverables: 'Clear product direction, user flow recommendations, and a high-confidence build checklist.'
    },
    'Create website': {
      goal: 'Design a premium landing experience with strong visual hierarchy and a persuasive conversion flow.',
      stack: 'Responsive layouts, polished sections, storytelling structure, CTA strategy, and scalable content blocks.',
      deliverables: 'Page architecture, section content plan, design system direction, and implementation breakdown.'
    },
    'Build application': {
      goal: 'Turn the idea into a full application with clear logic, workflows, and user journeys.',
      stack: 'Feature planning, data modeling, state management, core screens, and technical architecture.',
      deliverables: 'Project roadmap, modules, workflows, security considerations, and execution phases.'
    }
  };

  const result = strategy[mode] || strategy['Ask question'];

  return `
    <div>
      I’m shaping this around your goal: <strong>${prompt}</strong>
    </div>
    <div class="response-block">
      <span class="response-title">Understanding</span>
      <ul class="response-list">
        <li>${result.goal}</li>
        <li>${mediaText}</li>
      </ul>
    </div>
    <div class="response-block">
      <span class="response-title">Recommended structure</span>
      <ul class="response-list">
        <li>Start with the user journey and top priorities.</li>
        <li>Design the main screens around clarity, trust, and action.</li>
        <li>Define the core data and logic needed for a polished experience.</li>
      </ul>
    </div>
    <div class="response-block">
      <span class="response-title">Execution plan</span>
      <ul class="response-list">
        <li>${result.stack}</li>
        <li>Build iteratively with checkpoints for UX refinement and development validation.</li>
        <li>${result.deliverables}</li>
      </ul>
    </div>
  `;
}

function updateBlueprint(prompt, mode) {
  const map = {
    'Ask question': {
      goal: 'Clarify the product intent, user journey, and strategic direction before building.',
      stack: 'UX framing, flow mapping, and a clear implementation blueprint.',
      deliverable: 'Narrative structure, clarity improvements, and action-driven next steps.'
    },
    'Create website': {
      goal: `Develop a compelling site around “${prompt.slice(0, 60)}${prompt.length > 60 ? '…' : ''}” with a strong visual story and conversion strategy.`,
      stack: 'Responsive UI system, multi-section design, strong CTA hierarchy, and content patterns.',
      deliverable: 'High-converting pages, responsive sections, and a realistic implementation breakdown.'
    },
    'Build application': {
      goal: `Architect a meaningful product around “${prompt.slice(0, 60)}${prompt.length > 60 ? '…' : ''}” with logical flows and sophisticated interactions.`,
      stack: 'Modular feature architecture, typed data models, workflow logic, and scalable screens.',
      deliverable: 'Roadmap, core modules, middleware, and development phases for production-ready execution.'
    }
  };

  const details = map[mode] || map['Ask question'];
  goalText.textContent = details.goal;
  stackText.textContent = details.stack;
  deliverableText.textContent = details.deliverable;
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
  const typing = showTypingIndicator();
  updateBlueprint(prompt, state.mode);

  window.setTimeout(() => {
    typing.remove();
    appendMessage('assistant', buildAssistantResponse(prompt, state.mode, state.attachments));
    promptInput.value = '';
    state.attachments = [];
    renderAttachments();
  }, 700);
});

promptInput.addEventListener('keydown', (event) => {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault();
    sendButton.click();
  }
});

setActiveMode(state.mode);
renderAttachments();
