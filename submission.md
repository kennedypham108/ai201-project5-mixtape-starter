# Project 5 – Mixtape Bug Hunt Submission

## Codebase Map

### Main Files and Responsibilities

- **app.py**
  - Creates the Flask application, initializes the database, and registers all application routes.

- **models.py**
  - Defines the SQLAlchemy database models used throughout the application, including users, songs, playlists, notifications, and related entities.

- **routes/**
  - Contains all API endpoints.
  - The routes are responsible for receiving HTTP requests, validating inputs, calling the appropriate service functions, and returning responses.

- **routes/songs.py**
  - Handles song sharing, searching, and rating endpoints.

- **routes/playlists.py**
  - Handles playlist creation and playlist song management.

- **routes/users.py**
  - Handles user profile information, listening streaks, and notifications.

- **routes/feed.py**
  - Handles the Friends Listening Now activity feed.

- **services/**
  - Contains all business logic for the application.
  - The route files delegate almost all processing to these service modules.

- **streak_service.py**
  - Calculates and updates user listening streaks.

- **feed_service.py**
  - Generates the Friends Listening Now feed.

- **search_service.py**
  - Performs song searching and filtering.

- **notification_service.py**
  - Creates and retrieves notifications for user activity.

- **playlist_service.py**
  - Retrieves playlist contents and manages playlist-related logic.

- **seed_data.py**
  - Populates the database with sample users, songs, playlists, and other test data.

- **tests/**
  - Contains automated tests for streaks, searching, and playlists.

---

## Data Flow Example

### User rates a song

1. A client sends a request to:

   `POST /songs/<song_id>/rate`

2. The request is handled by `routes/songs.py`.

3. The route validates the request and forwards it to the appropriate function inside `notification_service.py`.

4. The service performs the business logic, updates the database if necessary, creates any required notification, and returns the result.

5. The route formats the response and sends it back to the client.

---

## Organization Patterns

- The application follows a layered architecture.
- Route files contain very little business logic.
- Service files contain nearly all application logic.
- Database operations are performed through SQLAlchemy models.
- Every feature follows the pattern:

  **HTTP Request → Route → Service → Database Models → Response**

This separation keeps routing, business logic, and database access organized and easier to debug.