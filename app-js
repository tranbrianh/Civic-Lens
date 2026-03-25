const searchInput = document.getElementById("searchInput");
const sortSelect = document.getElementById("sortSelect");
const issueCheckboxes = document.querySelectorAll(".issue-filters input[type='checkbox']");
const resultCount = document.getElementById("resultCount");
const loadingIndicator = document.getElementById("loadingIndicator");
const resultsList = document.getElementById("resultsList");
const emptyState = document.getElementById("emptyState");
const detailCard = document.getElementById("detailCard");

let currentResults = [];
let currentDetailId = null;
let debounceTimeout = null;

function getSelectedIssues() {
  const issues = [];
  issueCheckboxes.forEach((cb) => {
    if (cb.checked) issues.push(cb.value);
  });
  return issues;
}

function stanceLabel(stance) {
  if (stance === "support") return "Generally supports";
  if (stance === "oppose") return "Generally opposes";
  if (stance === "mixed") return "Mixed / conditional";
  return "No clear position";
}

function partyClass(party) {
  if (!party) return "";
  const p = party.toLowerCase();
  if (p.startsWith("dem")) return "dem";
  if (p.startsWith("rep")) return "rep";
  return "ind";
}

async function fetchResults() {
  const q = searchInput.value.trim();
  const sort = sortSelect.value;
  const issues = getSelectedIssues();
  const params = new URLSearchParams();

  if (q) params.set("q", q);
  if (sort) params.set("sort", sort);
  if (issues.length > 0) params.set("issues", issues.join(","));

  loadingIndicator.classList.remove("hidden");

  try {
    const res = await fetch(`/api/candidates?${params.toString()}`);
    const data = await res.json();
    currentResults = data.results || [];
    renderResults();
  } catch (err) {
    console.error(err);
    currentResults = [];
    renderResults();
  } finally {
    loadingIndicator.classList.add("hidden");
  }
}

function renderResults() {
  resultsList.innerHTML = "";
  if (currentResults.length === 0) {
    resultCount.textContent = "0 matches";
    emptyState.classList.remove("hidden");
    detailCard.classList.add("hidden");
    currentDetailId = null;
    return;
  }

  emptyState.classList.add("hidden");
  resultCount.textContent = `${currentResults.length} match${currentResults.length > 1 ? "es" : ""}`;

  currentResults.forEach((c, index) => {
    const div = document.createElement("div");
    div.className = "result-item";
    div.dataset.id = c.id;

    const topIssues = ["environment", "fiscal", "migration"].filter((key) => c[key]);

    div.innerHTML = `
      <div class="result-main">
        <div class="result-name">${c.name}</div>
        <div class="result-meta">
          ${c.office || "Office"}
          ${c.district ? " · " + c.district : ""}
          ${c.state ? " · " + c.state : ""}
        </div>
        <div class="result-issues">
          Issues: ${topIssues.length > 0 ? topIssues.join(", ") : "N/A"}
        </div>
      </div>
      <div>
        <span class="badge ${partyClass(c.party)}">${c.party || "Party"}</span>
      </div>
    `;

    div.addEventListener("click", () => {
      currentDetailId = c.id;
      renderDetail(c);
    });

    resultsList.appendChild(div);

    if (index === 0 && !currentDetailId) {
      currentDetailId = c.id;
      renderDetail(c);
    }
  });

  if (currentDetailId) {
    const current = currentResults.find((c) => c.id === currentDetailId);
    if (current) renderDetail(current);
  }
}

function renderDetail(c) {
  if (!c) {
    detailCard.classList.add("hidden");
    return;
  }

  detailCard.classList.remove("hidden");

  const issuesConfig = [
    { key: "environment", label: "Environment" },
    { key: "fiscal", label: "Fiscal" },
    { key: "migration", label: "Migration" },
    { key: "civilRights", label: "Civil rights" },
    { key: "healthcare", label: "Healthcare" },
    { key: "other", label: "Other" }
  ];

  const issuesHtml = issuesConfig
    .map((issue) => {
      const data = c[issue.key];
      if (!data) return "";
      return `
        <div class="issue-card">
          <div class="issue-card-title">${issue.label}</div>
          <div class="issue-card-stance">${stanceLabel(data.stance)}</div>
          <div class="issue-card-summary">${data.summary}</div>
        </div>
      `;
    })
    .join("");

  const sourcesHtml = (c.sources || [])
    .map(
      (s) =>
        `<a href="${s.url}" target="_blank" rel="noreferrer">${s.label}</a>`
    )
    .join("");

  detailCard.innerHTML = `
    <div class="detail-header">
      <div>
        <div class="detail-name">${c.name}</div>
        <div class="detail-office">
          ${c.office || "Office"}
          ${c.district ? " · " + c.district : ""}
          ${c.state ? " · " + c.state : ""}
          ${c.level ? " · " + c.level : ""}
        </div>
      </div>
      <div>
        <span class="badge ${partyClass(c.party)}">${c.party || "Party"}</span>
      </div>
    </div>
    <div class="detail-issues">
      ${issuesHtml}
    </div>
    <div class="sources">
      <strong>Sources:</strong>
      ${sourcesHtml || "To be added."}
    </div>
    <div class="sources" style="margin-top:4px;">
      Also check: 
      <a href="https://ballotpedia.org" target="_blank" rel="noreferrer">Ballotpedia</a>,
      <a href="https://www.vote411.org" target="_blank" rel="noreferrer">Vote411</a>
    </div>
  `;
}

function triggerSearchWithDebounce() {
  clearTimeout(debounceTimeout);
  debounceTimeout = setTimeout(fetchResults, 250);
}

// Event bindings
searchInput.addEventListener("input", triggerSearchWithDebounce);
sortSelect.addEventListener("change", triggerSearchWithDebounce);
issueCheckboxes.forEach((cb) => cb.addEventListener("change", triggerSearchWithDebounce));

// Initial load
fetchResults();
