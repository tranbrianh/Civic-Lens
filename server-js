// server.js
const express = require("express");
const cors = require("cors");
const path = require("path");
const politicians = require("./data/politicians");

const app = express();
const PORT = process.env.PORT || 4000;

app.use(cors());
app.use(express.json());

// Serve frontend
app.use(express.static(path.join(__dirname, "public")));

// Utility functions
const normalize = (s) => (s || "").toLowerCase();

const matchesSearch = (candidate, q) => {
  if (!q) return true;
  const query = normalize(q);
  const fields = [
    candidate.name,
    candidate.party,
    candidate.state,
    candidate.office,
    candidate.district
  ];
  return fields.some((f) => normalize(f).includes(query));
};

const matchesIssues = (candidate, issues) => {
  if (!issues || issues.length === 0) return true;
  for (const key of issues) {
    const data = candidate[key];
    if (!data) continue;
    if (!data.stance || data.stance === "neutral") return false;
  }
  return true;
};

const computeRelevance = (candidate, q) => {
  if (!q) return 0;
  const query = normalize(q);
  let score = 0;
  if (normalize(candidate.name).startsWith(query)) score += 3;
  if (normalize(candidate.name).includes(query)) score += 2;
  if (normalize(candidate.state).includes(query)) score += 1;
  if (normalize(candidate.office).includes(query)) score += 1;
  return score;
};

// API: list/search candidates
app.get("/api/candidates", (req, res) => {
  const { q, issues, sort } = req.query;

  const issueArray = issues
    ? issues
        .split(",")
        .map((i) => i.trim())
        .filter(Boolean)
    : [];

  let results = politicians
    .map((c) => ({
      ...c,
      relevanceScore: computeRelevance(c, q)
    }))
    .filter((c) => matchesSearch(c, q) && matchesIssues(c, issueArray));

  if (sort === "name") {
    results.sort((a, b) => a.name.localeCompare(b.name));
  } else {
    // default: relevance
    results.sort(
      (a, b) => b.relevanceScore - a.relevanceScore || a.name.localeCompare(b.name)
    );
  }

  res.json({ results });
});

// API: single candidate detail
app.get("/api/candidates/:id", (req, res) => {
  const id = Number(req.params.id);
  const candidate = politicians.find((c) => c.id === id);
  if (!candidate) {
    return res.status(404).json({ error: "Candidate not found" });
  }
  res.json({ candidate });
});

// Fallback to index.html for any non-API route
app.use((req, res, next) => {
  if (req.path.startsWith("/api")) {
    return next();
  }
  res.sendFile(path.join(__dirname, "public", "index.html"));
});
