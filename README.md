# sw-movie-app

A simple Node.js + Express API that fetches data from the Star Wars API (SWAPI) and allows users to save favorite movies or characters to a MongoDB database.

---

## 📦 Features

- Fetches movie and character data from [SWAPI](https://swapi.dev/)
- Allows users to save favorites via POST requests
- Lists saved favorites from MongoDB
- Built for containerized environments (Docker, Kubernetes)

---

## 🧠 Tech Stack

- **Node.js**
- **Express**
- **MongoDB**
- **Mongoose**
- **Axios**

---

## 🚀 Endpoints

### GET `/movies`
Fetches all Star Wars movies from SWAPI.

### GET `/people`
Fetches all Star Wars characters from SWAPI.

### GET `/favorites`
Returns all saved favorites from MongoDB.

### POST `/favorites`
Adds a favorite (must be type `"movie"` or `"character"`).

**Example Request:**
```json
{
  "name": "Luke Skywalker",
  "type": "character",
  "url": "https://swapi.dev/api/people/1/"
}
```

---

## 🐳 Running with Docker

### 1. Build the image

```bash
docker build -t <your-dockerhub-username>/sw-movie-app:latest .
```

### 2. Run the container (requires MongoDB accessible at `mongodb://mongodb:27017/swfavorites`)

```bash
docker run -p 3000:3000 <your-dockerhub-username>/sw-movie-app
```

---

## 🔁 Kubernetes / Argo CD

This app is designed to be deployed with Kubernetes via Argo CD.  
See [sw-movie-app-k8s](https://github.com/<your-username>/sw-movie-app-k8s) for manifests and setup instructions.

---

## 🧪 Development

### Install dependencies

```bash
npm install
```

### Start the app

```bash
node app.js
```

Ensure MongoDB is running and accessible at `mongodb://localhost:27017/swfavorites` or override the connection string in the code.

---

## 📄 License

MIT
