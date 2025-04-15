## Music Player App Development Plan (Flutter + Go)

This document outlines a comprehensive development plan for a music player application using Flutter for the frontend and Go for the backend. It includes project structure, recommended libraries, user roles, feature breakdowns with frontend and backend components, and a simplified database schema.

---

### 1. Recommended Project Structure

A well-organized project structure enhances maintainability, especially for solo developers.

#### Flutter (Frontend)

```
music_app_flutter/
├── android/                  # Android-specific files
├── ios/                      # iOS-specific files
├── lib/
│   ├── main.dart             # App entry point
│   ├── app/
│   │   ├── config/           # App-wide configurations (themes, constants)
│   │   ├── core/             # Utilities (error handling, constants)
│   │   ├── di/               # Dependency injection setup
│   │   └── navigation/       # Routing configuration
│   ├── data/
│   │   ├── datasources/      # Data sources (API, local storage)
│   │   │   ├── remote/       # API calls
│   │   │   └── local/        # Local storage (e.g., cache)
│   │   ├── models/           # Data models for API responses
│   │   └── repositories/     # Data layer implementations
│   ├── domain/
│   │   ├── entities/         # Business entities
│   │   ├── repositories/     # Repository interfaces
│   │   └── usecases/         # Business logic
│   ├── presentation/
│   │   ├── providers/        # State management
│   │   ├── screens/          # UI screens
│   │   │   ├── home/         # Home screen
│   │   │   ├── search/       # Search screen
│   │   │   ├── library/      # User library
│   │   │   ├── playlist/     # Playlist details
│   │   │   ├── album/        # Album details
│   │   │   ├── artist/       # Artist details
│   │   │   ├── now_playing/  # Playback screen
│   │   │   ├── profile/      # User profile
│   │   │   └── auth/         # Authentication screens
│   │   ├── widgets/          # Reusable UI components
│   │   └── utils/            # Presentation utilities
│   └── generated/            # Generated files (e.g., JSON serialization)
├── assets/
│   ├── fonts/                # Custom fonts
│   └── images/               # Static images
├── test/                     # Tests
├── pubspec.yaml              # Dependencies
└── README.md                 # Project documentation
```

#### Go (Backend)

```
music_app_go/
├── cmd/
│   └── api/
│       └── main.go           # Server entry point
├── internal/
│   ├── auth/                 # Authentication logic
│   ├── config/               # Configuration management
│   ├── database/             # Database setup and queries
│   ├── handlers/             # HTTP handlers
│   ├── middleware/           # Middleware (e.g., auth, logging)
│   ├── models/               # Database models
│   ├── services/             # Business logic
│   └── utils/                # Utility functions
├── pkg/
│   └── api/                  # API specifications (optional)
├── migrations/               # Database migrations
├── .env                      # Environment variables (not committed)
├── go.mod                    # Go modules
├── go.sum                    # Dependency checksums
└── README.md                 # Project documentation
```

---

### 2. Recommended Libraries/Packages

#### Flutter

- **State Management:** `flutter_riverpod` (scalable, recommended) or `provider`
- **Networking:** `dio` (feature-rich HTTP client) or `http`
- **Audio Playback:** `just_audio` (robust, supports queues) or `audioplayers`
- **Routing:** `go_router` (modern routing) or `auto_route`
- **Storage:** `path_provider` (file paths), `hive` (fast NoSQL) or `sqflite` (SQLite)
- **Dependency Injection:** `get_it` or Riverpod
- **JSON Serialization:** `json_serializable` or `freezed`
- **Icons:** `flutter_vector_icons` or `font_awesome_flutter`
- **UI Helpers:** `cached_network_image` (image caching), `flutter_svg` (SVG support)

#### Go

- **Web Framework:** `gin-gonic/gin` (fast, lightweight) or `labstack/echo`
- **ORM/DB:** `gorm.io/gorm` (ORM) or `jmoiron/sqlx` (SQL), with `gorm.io/driver/postgres`
- **Authentication:** `golang-jwt/jwt/v5` (JWT support)
- **Configuration:** `spf13/viper` (flexible config management)
- **Validation:** `go-playground/validator/v10`
- **Password Hashing:** `golang.org/x/crypto/bcrypt`
- **UUIDs:** `github.com/google/uuid`
- **Logging:** `log` (standard) or `rs/zerolog` (structured logging)
- **(Optional) Cloud Storage:** `aws/aws-sdk-go` (e.g., for S3)
- **(Optional) Migrations:** `golang-migrate/migrate`

---

### 3. User Roles

- **User**: Can listen to music, create playlists, and manage their library.
- **Artist**: Inherits User privileges, plus manages their profile and tracks (upload/edit/delete).
- **Admin**: Full control over users and content (tracks, albums, artists).

Roles are assigned during authentication and enforced via backend middleware.

---

### 4. Feature Breakdown: Pages, UI Elements & API Schemas

#### A. Core Playback Features

**Flutter Pages:**

- `NowPlayingScreen`: Full-screen player.
- `MiniPlayer` (Widget): Compact player at screen bottom.

**Flutter UI Elements:**

- `NowPlayingScreen`:
  - Album Art (`CachedNetworkImage`)
  - Song Title, Artist Name (`Text`)
  - Progress Bar (`Slider`) with Time (`Text`)
  - Play/Pause, Skip Next/Previous (`IconButton`)
  - Shuffle, Repeat (`IconButton`)
  - Volume Slider (`Slider`)
  - Queue, More Options (`IconButton`)
- `MiniPlayer`:
  - Thumbnail (`CachedNetworkImage`)
  - Song Title, Artist (`Text`, scrolling)
  - Play/Pause, Skip Next (`IconButton`)

**Backend API Endpoints:**

- **`GET /api/v1/tracks/{track_id}`**

  - **Purpose:** Fetch track details and stream URL.
  - **Request:** Authenticated, track ID in path.
  - **Response (200):**
    ```json
    {
      "id": "uuid-track-123",
      "title": "Song Title",
      "artist_id": "uuid-artist-456",
      "artist_name": "Artist Name",
      "album_id": "uuid-album-789",
      "album_name": "Album Name",
      "album_art_url": "https://cdn.example.com/art.jpg",
      "duration_ms": 240000,
      "stream_url": "https://cdn.example.com/track.mp3",
      "genre": "Pop"
    }
    ```
  - **Errors:** 404 (Not Found), 401 (Unauthorized).

- **`POST /api/v1/player/next`**

  - **Purpose:** Get next track based on context.
  - **Request:** Authenticated, JSON body:
    ```json
    {
      "current_track_id": "uuid-track-123",
      "context_type": "playlist",
      "context_id": "uuid-playlist-abc",
      "is_shuffling": false,
      "repeat_mode": "none"
    }
    ```
  - **Response (200):** Track details; 204 (End of queue).
  - **Errors:** 400 (Bad Request), 401.

- **`POST /api/v1/player/previous`**

  - **Purpose:** Get previous track.
  - **Request:** Similar to `/next`.
  - **Response (200):** Track details; 204 (Start reached).
  - **Errors:** 400, 401.

- **`POST /api/v1/player/history`** (Optional)
  - **Purpose:** Log track play.
  - **Request:** Authenticated:
    ```json
    {
      "track_id": "uuid-track-123",
      "played_at": "2023-10-01T12:00:00Z"
    }
    ```
  - **Response:** 204 (Success); 401.

**Note:** Playback state is managed client-side with `just_audio`; backend provides track data.

#### B. Music Library & Organization

**Flutter Pages:**

- `HomeScreen`: Recommendations, recent plays.
- `SearchScreen`: Music search.
- `LibraryScreen`: User’s saved content.
- `PlaylistScreen`, `AlbumScreen`, `ArtistScreen`: Detailed views.

**Flutter UI Elements:**

- `HomeScreen`: Carousels (`ListView`), Headers (`Text`).
- `SearchScreen`: Search Bar (`TextField`), Results (`ListView`).
- `LibraryScreen`: Tabs (`TabBarView`), Create Playlist (`FloatingActionButton`).
- `PlaylistScreen`/`AlbumScreen`: Header, Track List (`ListView`), Play/Shuffle Buttons.
- `ArtistScreen`: Image, Bio (`Text`), Tracks (`ListView`), Albums (`GridView`).

**Backend API Endpoints:**

- **`GET /api/v1/browse`**

  - **Purpose:** Fetch home screen content.
  - **Request:** Authenticated, `?page=1&limit=20`.
  - **Response (200):**
    ```json
    {
      "featured_playlists": [{ "id": "uuid-pl-abc", "name": "Chill Vibes" }],
      "new_releases": [{ "id": "uuid-album-xyz", "name": "Latest Hits" }],
      "recently_played": [{ "id": "uuid-track-123", "title": "Song" }]
    }
    ```
  - **Errors:** 401.

- **`GET /api/v1/search`**

  - **Purpose:** Search music.
  - **Request:** Authenticated, `?q=term&type=all`.
  - **Response (200):**
    ```json
    {
      "query": "term",
      "results": {
        "tracks": [{ "id": "uuid", "title": "Song" }],
        "artists": [{ "id": "uuid", "name": "Artist" }]
      }
    }
    ```
  - **Errors:** 400, 401.

- **`GET /api/v1/playlists`**

  - **Purpose:** List user playlists.
  - **Response (200):**
    ```json
    [{ "id": "uuid-pl-abc", "name": "My Favs", "track_count": 15 }]
    ```

- **`POST /api/v1/playlists`**

  - **Purpose:** Create playlist.
  - **Request:** Authenticated:
    ```json
    { "name": "Workout Mix" }
    ```
  - **Response (201):**
    ```json
    { "id": "uuid-pl-xyz", "name": "Workout Mix" }
    ```

- Additional endpoints: Manage playlists (`GET/PUT/DELETE /playlists/{id}`), library tracks, albums, artists.

#### C. Basic Account & User Experience

**Flutter Pages:**

- `AuthScreen`: Login/signup.
- `ProfileScreen`: User info.
- `SettingsScreen`: App settings.

**Flutter UI Elements:**

- `AuthScreen`: Fields (`TextField`), Buttons (`ElevatedButton`).
- `ProfileScreen`: Avatar (`CircleAvatar`), Info (`Text`).
- `SettingsScreen`: List (`ListView`), Toggles (`Switch`).

**Backend API Endpoints:**

- **`POST /api/v1/auth/register`**

  - **Request:**
    ```json
    { "username": "user", "email": "user@example.com", "password": "pass123" }
    ```
  - **Response (201):**
    ```json
    { "user": { "id": "uuid", "username": "user" }, "access_token": "..." }
    ```

- **`POST /api/v1/auth/login`**

  - **Request:**
    ```json
    { "email_or_username": "user", "password": "pass123" }
    ```
  - **Response (200):** User data and tokens.

- **`GET /api/v1/users/me`**
  - **Response (200):**
    ```json
    { "id": "uuid", "username": "user", "role": "User" }
    ```

#### D. Artist Features

**Flutter Pages:**

- `MyTracksScreen`: List artist tracks.
- `UploadTrackScreen`: Upload form.
- `EditTrackScreen`: Edit form.

**Flutter UI Elements:**

- `MyTracksScreen`: Track list (`ListView`), Upload button.
- `UploadTrackScreen`: Fields (`TextField`), File picker.

**Backend API Endpoints:**

- **`GET /api/v1/artist/tracks`**

  - **Purpose:** List artist’s tracks.
  - **Response (200):** Track list.

- **`POST /api/v1/artist/tracks`**
  - **Purpose:** Upload track.
  - **Request:** Multipart form (audio, metadata).
  - **Response (201):** Track details.

#### E. Admin Features

**Backend API Endpoints:**

- **`GET /api/v1/admin/users`**: List users.
- **`PUT /api/v1/admin/tracks/{track_id}`**: Edit any track.

---

### 5. Database Schema (Simplified)

- **users**: `id` (UUID), `username`, `email`, `password_hash`, `role` ('User', 'Artist', 'Admin'), `created_at`.
- **artists**: `id` (UUID), `user_id` (FK), `name`, `bio`.
- **tracks**: `id` (UUID), `title`, `artist_id` (FK), `file_path`.
- **playlists**: `id` (UUID), `name`, `owner_id` (FK).

---

Start with authentication and playback, then expand iteratively. Good luck!
