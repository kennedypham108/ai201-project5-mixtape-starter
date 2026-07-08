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

## Issue #3 — The same song keeps showing up twice in search

### How I reproduced it

I started the Flask app and opened the search endpoint with the query `Anthem`:

`GET /songs/search?q=Anthem`

The endpoint is handled by `routes/songs.py`, which reads the `q` query parameter and calls `search_songs(query)` in `services/search_service.py`. The search response returned duplicate copies of the same matching song instead of listing each song once.

## Issue #5 — The last song in a playlist never shows up

### How I reproduced it

I opened the playlist songs endpoint:

## How I found the root cause

I started from the playlist route in `routes/playlists.py`, which calls `get_playlist_songs()` in `services/playlist_service.py`. I traced the function until I found the return statement. The query correctly retrieved every song in the playlist, but the final return statement sliced the list with `songs[:-1]`, removing the last song before returning the results.

## The root cause

The playlist query correctly returned every song, but the code used Python list slicing (`songs[:-1]`) when building the response. Since `[:-1]` excludes the final element of the list, the newest song was always omitted from the API response.

## My fix and side-effect check

I removed the list slice and returned the entire list using `songs` instead of `songs[:-1]`. After the change, every song in the playlist was returned in the correct order, including the newest song. I also verified that the ordering of songs remained unchanged.

`GET /playlists/<playlist_id>/songs`

The route calls `get_playlist_songs(playlist_id)` in `services/playlist_service.py`. The playlist response returned one fewer song than expected. After checking the service function, I confirmed that the returned list excludes the final song in the ordered playlist.

## Issue #4 — Rating notifications are not created

### How I reproduced it

I rated a song shared by another user using:

## How I found the root cause

I traced the request from `routes/songs.py` into `rate_song()` inside `notification_service.py`. The function successfully created or updated a rating and committed it to the database. Unlike the playlist notification function, however, it never called `create_notification()`, which explained why no notification was generated.

## The root cause

The rating feature saved the rating correctly but never created a notification for the song owner. The notification logic existed elsewhere in the application but was never called after a rating was submitted.

## My fix and side-effect check

I added a call to `create_notification()` after successfully saving the rating. I also kept the existing behavior that prevents users from notifying themselves when rating their own songs. After testing, ratings continued to save correctly, and notifications were generated for the original song owner.

`POST /songs/<song_id>/rate`

with a JSON body containing a rater user ID and a score. The route in `routes/songs.py` calls `rate_song(user_id, song_id, int(score))`. The rating was saved successfully, but when I checked the original sharer's notifications with:

`GET /users/<owner_id>/notifications`

there was no notification for the rating.