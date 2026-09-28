# 🎬 MovieForMe

A movie recommendation web app. Type the title of a movie you like —or describe in your own words what you feel like watching— and MovieForMe suggests similar movies.

## ✨ Features

- **Recommend by title**: finds the 10 movies most similar to a given one, using KNN with cosine similarity over genres, cast, director and keywords.
- **Recommend by description**: turns free text into a semantic embedding (`all-MiniLM-L6-v2`) and finds the closest movies with a pre-trained KNN model.
- **Title autocomplete** as you type.
- **Posters** fetched in real time from the [OMDb API](https://www.omdbapi.com/).

## 🧱 Tech Stack

| Layer | Stack |
|-------|-------|
| Frontend | React 19, TypeScript, Vite, Axios |
| Backend | Python, FastAPI, Uvicorn, Pydantic |
| ML | Sentence-Transformers, scikit-learn, pandas, NumPy, SciPy |
| Deployment | Docker (backend) |

## 📁 Project Structure

```
MovieForMe/
├── SetupDeployment.ps1          # Installs dependencies (venv + npm)
├── StartApp.ps1                 # Starts backend and frontend
├── Source/
│   ├── Client/ReactWebApp/      # React + Vite frontend
│   │   └── src/
│   │       ├── APIs/config.ts   # API and OMDb URLs
│   │       ├── Components/      # SearchBar, MovieCard, ToggleSwitch, ...
│   │       └── MovieForMeHome.tsx
│   └── Server/                  # FastAPI backend
│       ├── main.py              # Endpoints
│       ├── models.py            # Pydantic models
│       ├── util.py              # Recommendation logic
│       ├── data/                # Movie dataset (.pk1)
│       ├── modelsML/            # KNN, embeddings and Sentence-Transformer model
│       ├── requirements.txt
│       └── dockerfile
└── test/                        # Model inspection scripts
```

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+ and npm
- Windows PowerShell (for the included scripts)

### Option 1: PowerShell scripts

From the project root:

```powershell
# 1. Create the virtual environment and install dependencies (first time only)
.\SetupDeployment.ps1

# 2. Start backend and frontend in separate windows
.\StartApp.ps1
```

### Option 2: Manual

**Backend**

```bash
cd Source/Server
python -m venv .venv
.venv\Scripts\activate        # Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

**Frontend**

```bash
cd Source/Client/ReactWebApp
npm install
npm run dev
```

- Frontend: http://localhost:5173
- API: http://127.0.0.1:8000 (interactive docs at `/docs`)

### Option 3: Docker (backend)

```bash
cd Source/Server
docker build -t movieforme-api .
docker run -p 8000:8000 movieforme-api
```

## 🔌 API

Base URL: `http://127.0.0.1:8000/api/movies`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/recommendations/title` | Recommendations from a movie title |
| `POST` | `/recommendations/description` | Recommendations from a description |
| `GET`  | `/search/title?q=<text>` | Title suggestions for autocomplete |

**Example request**

```json
POST /api/movies/recommendations/title
{
  "Title": "The Dark Knight",
  "Description": null,
  "DebugMode": false
}
```

**Example response**

```json
{
  "SourceMovie": {
    "Title": "The Dark Knight",
    "Genres": ["Action", "Crime", "Drama"],
    "Rating": 8.2,
    "HomePageURL": "...",
    "PosterURL": null
  },
  "MovieRecommendations": [
    { "Title": "...", "Genres": ["..."], "Rating": 7.5, "HomePageURL": "...", "PosterURL": null }
  ]
}
```

## ⚙️ Configuration

- **API URL and OMDb key**: `Source/Client/ReactWebApp/src/APIs/config.ts`
- **Allowed CORS origins**: `origins` list in `Source/Server/main.py`

## 🧠 How It Works

1. **By title**: the movie is located in the dataset and its distance to every other movie is computed as the sum of cosine distances between their binary vectors for genres, cast, director and keywords. The 10 nearest neighbors are returned.
2. **By description**: the text is normalized, encoded with the `all-MiniLM-L6-v2` model into a 384-dimensional vector, and a KNN model over precomputed movie embeddings returns the most similar movies.
