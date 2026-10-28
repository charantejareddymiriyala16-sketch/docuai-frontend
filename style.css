
const API_BASE = "https://docuai-backend-63p9.onrender.com";
const REQUEST_TIMEOUT_MS = 90000;

const fileInput = document.getElementById("fileInput");
const fileInfo = document.getElementById("fileInfo");
const summarizeBtn = document.getElementById("summarizeBtn");
const statusEl = document.getElementById("status");
const resultCard = document.getElementById("resultCard");
const resultTitle = document.getElementById("resultTitle");
const summaryEl = document.getElementById("summary");
const copySummaryBtn = document.getElementById("copySummary");
const chatCard = document.getElementById("chatCard");
const messagesEl = document.getElementById("messages");
const chatForm = document.getElementById("chatForm");
const questionInput = document.getElementById("questionInput");
const newChatBtn = document.getElementById("newChat");
const historyEl = document.getElementById("history");

let currentFile = null;
let currentDocumentText = "";
let currentChatId = null;
let chats = [];
let busy = false;

function setStatus(message, type = "") {
  statusEl.textContent = message;
  statusEl.style.color =
    type === "error" ? "#fca5a5" :
    type === "success" ? "#7ee7c1" : "#969bad";
}

function setBusy(value) {
  busy = value;

  summarizeBtn.disabled = value || !currentFile;
  summarizeBtn.innerHTML = value
    ? "<span>◌</span> Processing document..."
    : "<span>✦</span> Summarize document <span>→</span>";

  questionInput.disabled = value;
  chatForm.querySelector("button").disabled = value;
}

function newChat() {
  currentChatId = null;
  currentFile = null;
  currentDocumentText = "";

  fileInput.value = "";
  fileInfo.hidden = true;
  fileInfo.textContent = "";

  resultCard.hidden = true;
  chatCard.hidden = true;
  summaryEl.textContent = "";
  resultTitle.textContent = "Your summary";

  messagesEl.replaceChildren();
  questionInput.value = "";

  setStatus("");
  setBusy(false);
  renderHistory();
}

newChatBtn.addEventListener("click", newChat);

fileInput.addEventListener("change", () => {
  const file = fileInput.files[0];

  if (!file) {
    currentFile = null;
    summarizeBtn.disabled = true;
    return;
  }

  const allowed = /\.(pdf|docx|txt)$/i.test(file.name);

  if (!allowed) {
    currentFile = null;
    fileInput.value = "";
    fileInfo.hidden = true;
    setStatus("Please choose a PDF, DOCX, or TXT file.", "error");
    summarizeBtn.disabled = true;
    return;
  }

  if (file.size > 10 * 1024 * 1024) {
    currentFile = null;
    fileInput.value = "";
    fileInfo.hidden = true;
    setStatus("File is too large. Maximum size is 10 MB.", "error");
    summarizeBtn.disabled = true;
    return;
  }

  currentFile = file;
  fileInfo.hidden = false;
  fileInfo.textContent =
    `📄 ${file.name} · ${formatSize(file.size)}`;

  summarizeBtn.disabled = busy;
  setStatus("File selected. Ready to summarize.");
});

function formatSize(bytes) {
  if (bytes < 1024) return `${bytes} B`;

  if (bytes < 1024 * 1024) {
    return `${(bytes / 1024).toFixed(1)} KB`;
  }

  return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
}

function makeChatId() {
  return `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`;
}

function ensureChat(title) {
  if (!currentChatId) {
    currentChatId = makeChatId();

    chats.unshift({
      id: currentChatId,
      title: title || "New conversation",
      messages: [],
      summary: "",
      documentText: "",
      fileName: ""
    });
  }

  return chats.find(chat => chat.id === currentChatId);
}

function renderHistory() {
  historyEl.replaceChildren();

  if (!chats.length) {
    const empty = document.createElement("div");
    empty.className = "empty-history";
    empty.textContent = "Your conversations will appear here.";
    historyEl.appendChild(empty);
    return;
  }

  chats.forEach(chat => {
    const button = document.createElement("button");

    button.className =
      "history-item" +
      (chat.id === currentChatId ? " active" : "");

    button.textContent = "▤  " + chat.title;
    button.title = chat.title;

    button.addEventListener("click", () => openChat(chat.id));

    historyEl.appendChild(button);
  });
}

function openChat(id) {
  const chat = chats.find(item => item.id === id);
  if (!chat) return;

  currentChatId = id;
  currentDocumentText = chat.documentText || "";
  currentFile = null;

  fileInput.value = "";
  fileInfo.hidden = true;
  summarizeBtn.disabled = true;

  resultCard.hidden = !chat.summary;
  summaryEl.textContent = chat.summary || "";

  resultTitle.textContent = chat.fileName
    ? `Summary: ${chat.fileName}`
    : "Your summary";

  chatCard.hidden = !chat.documentText;
  messagesEl.replaceChildren();

  chat.messages.forEach(message => {
    addMessage(message.role, message.text, false);
  });

  setStatus(
    chat.fileName
      ? `Previous conversation: ${chat.fileName}.`
      : "Opened saved conversation."
  );

  renderHistory();
}

function addMessage(role, text, save = true) {
  const message = document.createElement("div");

  message.className =
    "message " + (role === "user" ? "user" : "assistant");

  message.textContent = text;
  messagesEl.appendChild(message);
  messagesEl.scrollTop = messagesEl.scrollHeight;

  if (save && currentChatId) {
    const chat = chats.find(item => item.id === currentChatId);

    if (chat) {
      chat.messages.push({ role, text });
    }
  }
}

async function summarizeDocument() {
  if (!currentFile || busy) return;

  const file = currentFile;
  const controller = new AbortController();

  const timeoutId = setTimeout(() => {
    controller.abort();
  }, REQUEST_TIMEOUT_MS);

  setBusy(true);
  setStatus("Uploading your document. Waiting for Gemini...");

  try {
    const formData = new FormData();
    formData.append("file", file);

    const response = await fetch(`${API_BASE}/summarize`, {
      method: "POST",
      body: formData,
      signal: controller.signal
    });

    const data = await response.json().catch(() => ({}));

    if (!response.ok) {
      throw new Error(
        data.detail ||
        `Request failed (${response.status}). Please try again.`
      );
    }

    if (!data.summary) {
      throw new Error("The server returned an empty summary.");
    }

    const chat = ensureChat(file.name);

    chat.title = file.name;
    chat.fileName = file.name;
    chat.summary = data.summary || "No summary was returned.";

    chat.documentText = data.document_text || "";
    currentDocumentText = chat.documentText;

    resultTitle.textContent = `Summary: ${file.name}`;
    summaryEl.textContent = chat.summary;

    resultCard.hidden = false;
    chatCard.hidden = !chat.documentText;

    renderHistory();

    setStatus(
    chat.documentText
    ? "Summary ready! You can now ask questions about your document."
    : "Summary ready, but document text was not returned.",
    "success"
    );

    resultCard.scrollIntoView({
      behavior: "smooth",
      block: "start"
    });

  } catch (error) {
    if (error.name === "AbortError") {
      setStatus(
        "The request took over 90 seconds. Gemini may be overloaded. Please try again later.",
        "error"
      );
    } else {
      setStatus(
        error.message ||
        "Could not summarize the file. Please try again.",
        "error"
      );
    }
  } finally {
    clearTimeout(timeoutId);
    setBusy(false);
  }
}

summarizeBtn.addEventListener("click", summarizeDocument);

copySummaryBtn.addEventListener("click", async () => {
  const text = summaryEl.textContent.trim();

  if (!text) {
    setStatus("There is no summary to copy yet.", "error");
    return;
  }

  try {
    await navigator.clipboard.writeText(text);

    copySummaryBtn.textContent = "Copied!";

    setTimeout(() => {
      copySummaryBtn.textContent = "Copy summary";
    }, 1500);

  } catch {
    setStatus(
      "Clipboard access failed. Select and copy the summary manually.",
      "error"
    );
  }
});

chatForm.addEventListener("submit", async event => {
  event.preventDefault();

  const question = questionInput.value.trim();

  if (!question || busy) return;

  if (!currentChatId) {
    setStatus(
      "Summarize a document first to start asking questions.",
      "error"
    );
    return;
  }

  const chat = chats.find(item => item.id === currentChatId);

  if (!chat || !chat.documentText) {
    setStatus(
      "Document Q&A is not connected yet. We need to update the backend to provide the extracted document text.",
      "error"
    );
    return;
  }

  const controller = new AbortController();

  const timeoutId = setTimeout(() => {
    controller.abort();
  }, REQUEST_TIMEOUT_MS);

  questionInput.value = "";
  addMessage("user", question);
  setBusy(true);
  setStatus("Finding an answer in your document...");

  try {
    const formData = new FormData();

    formData.append("question", question);
    formData.append("document_text", chat.documentText);

    const response = await fetch(`${API_BASE}/chat`, {
      method: "POST",
      body: formData,
      signal: controller.signal
    });

    const data = await response.json().catch(() => ({}));

    if (!response.ok) {
      throw new Error(
        data.detail || `Request failed (${response.status}).`
      );
    }

    addMessage(
      "assistant",
      data.answer || "No answer was returned."
    );

    setStatus("Answer received.", "success");

  } catch (error) {
    const message = error.name === "AbortError"
      ? "The request timed out. Please try again."
      : error.message || "Could not answer the question.";

    addMessage("assistant", `Sorry, I couldn't answer that: ${message}`);
    setStatus(message, "error");

  } finally {
    clearTimeout(timeoutId);
    setBusy(false);
  }
});

renderHistory();
setStatus("Connected to the DocuAI backend. Choose a document to begin.");
