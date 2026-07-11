# GitHub Profile Analyzer

An AI-powered GitHub Profile Analyzer. Enter any public GitHub username and get a full
dashboard: profile overview, repository stats, language breakdown, contribution trends,
a computed "Developer Score", and AI-generated career/skill insights — plus PDF/JSON export,
profile comparison, dark/light theme, and more.

Full-stack MERN-style app: **React + TypeScript** frontend, **Node.js + Express** backend.

---

## ✨ Features

- Search any GitHub username → profile picture, bio, company, location, followers, repos, join date
- Repository analysis: most starred/forked repo, languages, size, open issues, watchers, topics
- Language usage % (byte-accurate, via GitHub's `/languages` API) with a pie chart
- Contribution analysis: repo creation timeline, recent activity (last 90 days of public events)
- **Repository Health Score** per repo (docs, license, topics, recent pushes, open issues)
- **Developer Score** (0–100) + estimated experience level, computed from followers, stars, output, language diversity, recent activity
- AI Insights: skill summary, strongest technologies, learning recommendations, project ideas, resume-ready summary, career recommendations
  - Works with **OpenAI** or **Gemini** if you add an API key — falls back to a solid rule-based generator if you don't
- Interactive charts (Recharts): language pie chart, stars bar chart, repo timeline, activity area chart
- Repository table: search, filter by language, sort by stars/forks/updated
- Export: download JSON, download a PDF report, copy a shareable link
- **Compare two profiles** side-by-side
- Bookmarks + recent search history (stored in the browser)
- Trending repositories of the week on the home page
- Dark/light theme toggle, fully responsive
- Optional "Login with GitHub" OAuth scaffold (not required to use the app)

---

## 🧱 Tech Stack

**Frontend:** React 18, TypeScript, Vite, Tailwind CSS, Recharts, Axios, React Router, jsPDF + html2canvas
**Backend:** Node.js, Express, Axios, node-cache, express-rate-limit
**APIs:** GitHub REST API, OpenAI or Gemini API (optional)

---

## 📁 Project Structure

```
github-profile-analyzer/
├── client/                  # React + TypeScript frontend
│   └── src/
│       ├── components/      # ProfileCard, StatsGrid, RepoTable, AIInsightsPanel, etc.
│       ├── charts/           # Recharts components
│       ├── pages/            # HomePage, DashboardPage, ComparePage
│       ├── hooks/            # useGithubAnalysis
│       ├── context/          # ThemeContext (dark/light)
│       ├── services/         # api.ts (Axios client)
│       └── utils/            # storage.ts, format.ts, export.ts
└── server/                  # Express backend
    ├── controllers/         # githubController, aiController
    ├── routes/               # githubRoutes, aiRoutes, authRoutes
    ├── services/             # githubService, analysisService, aiService
    └── middleware/           # errorHandler
```

---

## 🚀 Getting Started

### 1. Backend

```bash
cd server
npm install
cp .env.example .env
```

Open `server/.env` and, at minimum, consider adding a **GitHub personal access token**
(no scopes needed — it's only used for public data) to raise your rate limit from 60 to
5,000 requests/hour: https://github.com/settings/tokens

```
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
```

To enable real AI-generated insights, also add an OpenAI or Gemini key and set
`AI_PROVIDER` accordingly. **This is optional** — without a key, the app automatically
uses a rule-based insights generator so every feature still works.

```
AI_PROVIDER=openai
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxx
```

Start the server:

```bash
npm run dev      # with nodemon, auto-restarts on change
# or
npm start
```

The API runs on `http://localhost:5000`.

### 2. Frontend

In a new terminal:

```bash
cd client
npm install
npm run dev
```

The app runs on `http://localhost:5173` and proxies `/api` requests to the backend
(see `vite.config.ts`).

### 3. Use it

Open `http://localhost:5173`, type a GitHub username (e.g. `torvalds`, `gaearon`, or
your own — `mahi2kx`), and hit **Analyze**.

---

## 🔑 Environment Variables (server/.env)

| Variable | Required? | Purpose |
|---|---|---|
| `PORT` | No (defaults to 5000) | API port |
| `CLIENT_URL` | No | CORS origin, used for OAuth redirect too |
| `GITHUB_TOKEN` | Recommended | Raises GitHub API rate limit to 5,000/hr |
| `AI_PROVIDER` | No | `openai` or `gemini`; omit to use rule-based insights |
| `OPENAI_API_KEY` / `OPENAI_MODEL` | Only if using OpenAI | AI insight generation |
| `GEMINI_API_KEY` / `GEMINI_MODEL` | Only if using Gemini | AI insight generation |
| `GITHUB_OAUTH_CLIENT_ID/SECRET/CALLBACK_URL` | Only for OAuth login | Optional "Login with GitHub" |
| `MONGODB_URI` | Not used yet | Reserved — history/bookmarks currently use localStorage |

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/github/profile/:username` | Raw GitHub user profile |
| GET | `/api/github/repos/:username` | All repos for a user (summarized) |
| GET | `/api/github/analysis/:username` | Full computed analysis (stats, languages, score, etc.) |
| GET | `/api/github/compare?user1=&user2=` | Analysis for two users at once |
| GET | `/api/github/trending?language=` | Trending repos created this week |
| POST | `/api/ai/insights` | `{ analysis }` or `{ username }` → AI (or rule-based) insights |
| GET | `/api/auth/github` | Optional OAuth login redirect |
| GET | `/api/health` | Health check |

---

## 📝 Notes & Design Decisions

- **Language %** is calculated byte-accurately via GitHub's `/repos/{owner}/{repo}/languages`
  endpoint for a user's top 25 repos (by stars) to keep request counts reasonable on accounts
  with hundreds of repos; remaining repos are weighted by their primary `language` field.
- **Contribution heatmap** uses GitHub's public Events API, which only exposes roughly the
  last 90 days of activity — this is a GitHub platform limitation, not a bug.
- **History & bookmarks** are stored in `localStorage` so the app works with zero database
  setup. `MONGODB_URI` is reserved in `.env.example` if you want to move this server-side.
- **PDF export** renders the on-screen dashboard via `html2canvas` + `jsPDF`, so it always
  matches what you see, including your current theme.
- Responses are cached server-side for 5 minutes (`node-cache`) to avoid hammering the
  GitHub API when revisiting the same profile.

---

## 🛣️ Possible Next Steps

- Persist search history/bookmarks to MongoDB per authenticated user
- Move the GitHub OAuth flow to issue a signed session/JWT cookie instead of a redirect param
- Add README quality scoring (currently the backend fetches READMEs but doesn't score them yet — an easy extension in `analysisService.js`)
- Code-split the client bundle (Vite warns the current bundle is a bit large due to jsPDF/html2canvas)
