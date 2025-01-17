# Recommend For Plex

This is a comprehensive recommendation system that integrates with Plex, TMDB, and TVDB to provide movie and TV show recommendations. It uses PySide6 for the GUI, several PyTorch-based encoders and neural networks for generating embeddings, a knowledge graph for content relationships, and an SQLite database for storing media items, embeddings, preferences, and feedback.

This is designed to be very good at learning, but still light enough to run on a laptop using only CPU.

The instructions are below. You can either run main.py directory or run the compiled EXE in the releases section.

![{E5F47FA3-837B-49CC-8363-4999D31C9F64}](https://github.com/user-attachments/assets/7c5710dd-be45-4bb6-9770-562095d091f5)


![{36003507-4CD2-47BC-A406-A4D89EB7E54E}](https://github.com/user-attachments/assets/c332b13d-593f-4e82-a878-f1d788921c63)




## Please consider this a evolving WIP. I'll do my best to ensure it is stable before any publishing but report any issues you encounter.
## Note: There is a known bug where the application will freeze if it doing embedding computation, you have a large library, and it tries to compute the similarity matrix. The workaround is to scan in the large library and leave the app up without interacting with it, for 2 hours. This will give it time to churn through all the computations it needs to. I'm working on a new version with a LOT of performance and stability fixes, but it could be a few days.


## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Installation and Setup](#installation-and-setup)
  - [Prerequisites](#prerequisites)
  - [Python Dependencies](#python-dependencies)
  - [Running the Application](#running-the-application)
- [Usage](#usage)
  - [Main Interface](#main-interface)
  - [Configuration Tab](#configuration-tab)
  - [Movies and TV Tabs](#movies-and-tv-tabs)
  - [Rating Items](#rating-items)
  - [Background Training](#background-training)
  - [Watchlist Management](#watchlist-management)
- [How It Works](#how-it-works)
  - [Database Schema](#database-schema)
  - [Embedding Computations](#embedding-computations)
  - [Knowledge Graph](#knowledge-graph)
  - [Similarity Computations](#similarity-computations)
  - [Recommendation Flow](#recommendation-flow)
  - [Exploration vs Exploitation](#exploration-vs-exploitation)
- [Advanced](#advanced)
  - [Resetting the Model](#resetting-the-model)
  - [Logging](#logging)
  - [Background Trainer Details](#background-trainer-details)

---

## Overview

This application is designed to recommend movies and TV shows to users. It leverages data from:
- **Plex**: For retrieving library info and metadata (where available).
- **TMDB** (The Movie Database): For enhanced metadata on movies and TV series.
- **TVDB** (TheTVDB): For extended TV series data, especially episodes.

The application uses a PySide6 interface, enabling you to configure keys, Plex server details, select libraries, scan them, and start receiving recommendations. You can rate items from 1 to 10, and the system will adapt its recommendations over time.

---

## Features

- **GUI-based**: Manage everything from one window.
- **Database-Backed**: Uses SQLite (`recommend_for_plex.db`).
- **Embedding Generation**: Uses transformers, graph networks, and custom neural encoders to generate a rich embedding for each media item.
- **Knowledge Graph**: Captures relationships between media items, genres, keywords, and entities extracted from textual metadata.
- **Feedback & Training**: Allows user ratings to dynamically improve the recommendation engine.
- **Caching**: Stores computed embeddings in the database to avoid recomputation.
- **Background Training**: Continuously refines the model and updates similarity metrics in the background.

---

## Installation and Setup

### Prerequisites

1. **Python 3.8+** (recommended).
2. **Plex Server** 
3. **API Keys**:
   - [TMDB API Key](https://www.themoviedb.org/documentation/api)
   - [TVDB API Key](https://thetvdb.com/api-information)
   - [Plex Token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)

### Python Dependencies

Install dependencies in a virtual environment (recommended). For example:
    
    pip install -r requirements.txt

If you do not have a `requirements.txt`, here is a non-exhaustive list of the major libraries needed:
- PySide6
- requests
- aiohttp
- sqlite3 (built-in in Python, but note usage details)
- PlexAPI
- transformers
- torch
- torch-geometric
- numpy
- nltk
- logging (built-in)
- packaging (sometimes required for PyTorch compatibility)

Note: Depending on your environment, some additional PyTorch Geometric dependencies (e.g., `torch-scatter`, `torch-sparse`) may be required separately. Refer to the [PyTorch Geometric installation guide](https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html) for your specific OS and CUDA version.

### Running the Application

#### Recommended

Use the pre-compiled executable available in the **Releases** section of this repository. This option simplifies the setup process by bundling all necessary dependencies.

1. Download the latest release executable from the [Releases](#) section.
2. Run the executable to start the application.

#### Advanced

For users who prefer to run the application from the source code, follow the instructions below:

1. **Clone** or **download** this repository.
2. **Navigate** into the project folder.
3. **Launch** the Python application:
    
       python main.py
    
   or:
    
       python -m main
    
4. The PySide6 UI should appear shortly.

---

## Usage

### Main Interface

When the app first opens, you’ll see three main tabs:
1. **Movies** – Current recommended movie and the option to rate it.
2. **TV Shows** – Current recommended TV show and the option to rate it.
3. **Configuration** – Settings for Plex, API keys, library scanning, etc.

### Configuration Tab

1. **Save Configuration**:
   - Enter *Plex URL* (e.g., `http://127.0.0.1:32400`) and *Plex Token*.
   - Enter *TVDB Key* and *TMDB Key*.
   - Click "Save Configuration" to write a `config.json` in the current folder.
2. **Load Libraries**:
   - Attempts to fetch your Plex libraries (movies, shows).
   - They are shown in the "Library Selection" list.
3. **Library Selection**:
   - Choose which libraries to scan (Ctrl+Click or Shift+Click for multiple).
4. **Model Management**:
   - **Reset Model**: Clears out all ratings, embeddings, and preferences.
   - **Scan Libraries**: Starts scanning the selected Plex libraries.

### Movies and TV Tabs

- Display the recommended poster (if available).
- Show the title, year, runtime, genres, and a short summary.
- You can provide a rating from 1 to 10 using the rating buttons.
- "Skip" button to move to the next recommendation.
- "Block Item" button to never show this item again.
- "Add to Plex Watchlist" button to add the item to your Plex watchlist.

### Rating Items

- As soon as you click a rating, that feedback is saved.
- New recommendations are automatically generated (item changes).
- Over time, the system learns your preferences.

### Background Training

A separate **Background Trainer** thread is initialized at startup. It:
- Periodically updates missing or old embeddings.
- Rebuilds the similarity matrix.
- Uses user feedback to train the model weights incrementally.
- Logs its progress in the bottom text box of the **Configuration** tab.

### Watchlist Management

- **Adding to Watchlist**: Each movie/show recommendation has an "Add to Plex Watchlist" button. When clicked, it will:
  - Search for the exact match in your Plex library
  - If found, add it to your watchlist
  - If multiple matches are found, allow you to select the correct one
  - Show a confirmation when added successfully

- **Clearing Watchlist**: In the Configuration tab:
  - Click "Clear Plex Watchlist" to remove all items
  - Confirms before proceeding
  - Shows real-time progress in the Configuration tab's text area
  - Provides a summary when complete

---

## How It Works

### Database Schema

The SQLite database `recommend_for_plex.db` has these key tables:

- **media_items**: Stores movies/shows metadata (title, year, genres, summary, runtime, etc.).
- **genre_preferences**: Tracks how users rate different genres (for a basic collaborative approach).
- **user_feedback**: Each row has a `media_id`, a `rating`, and a timestamp.
- **embedding_cache**: Binary embeddings for each media item (float32 array).
- **similarity_matrix**: Pairwise similarity scores between items.

### Embedding Computations

When a new item is encountered, several neural encoders generate a **384-dim** embedding:

1. **Text Features** (`_encode_text_features`): Uses a MiniLM-L12-H384 transformer to encode text fields (title, summary, keywords, etc.).
2. **Numerical Features** (`_encode_numerical_features`): Popularity, average votes, etc.
3. **Categorical Features** (`_encode_categorical_features`): Genre one-hot vectors, status, language, etc.
4. **Temporal Features** (`_encode_temporal_features`): Release year, month encoding, etc.
5. **Metadata Features** (`_encode_metadata_features`): Production companies, credits, keywords, etc.
6. **Graph Features** (`_compute_graph_features`): Three graph-based networks (GCN, GAT, GraphSAGE) that compute subgraph embeddings from the knowledge graph. These embeddings are combined into a 384-dim vector.

### Knowledge Graph

A knowledge graph is built to represent relationships such as:

- `media_{id} -- has_genre --> genre_{name}`
- `media_{id} -- has_keyword --> keyword_{name}`
- `media_{id} -- mentions_entity --> entity_{extracted_noun}`

We retrieve a subgraph around the item to build graph embeddings, capturing relational context.

### Similarity Computations

When a user provides feedback (rating >= 1), the system:
- Updates the embedding for that item.
- Kicks off a similarity computation job to measure similarity with other candidates.
- Stores these similarities in the `similarity_matrix`.

### Recommendation Flow

1. **Candidate Selection**:
   - System picks items not yet rated and filters them by type (movie or show).
   - Might randomly do some exploration (select lesser-known or unscored items) based on the total feedback count.
2. **Score Calculation** (`_calculate_item_score`):
   - Combines popularity, vote average, vote count, and genre preference for a quick ranking.
3. **Embeddings / Similarity**:
   - Optionally merges in similarity-based or graph-based approaches for final scoring (hybrid approach).
4. **Presents** the highest-scoring item on the UI.

### Exploration vs Exploitation

- When few ratings exist (< 50), higher exploration rate is used (random picks from new genres).
- As more feedback is collected, system tilts more toward exploitation of known preferences.

---

## Advanced

### Resetting the Model

- Go to **Configuration** → **Model Management** → **Reset Model**.
- This:
  1. Empties `user_feedback`.
  2. Clears `genre_preferences`.
  3. Deletes rows in `embedding_cache`.
  4. Rebuilds the knowledge graph.
- You start fresh as if no content is rated.

### Logging

- The main logger writes to a rotating file in the system temp folder (e.g., `recommender_for_plex.log`).
- You’ll see debug and info messages in the console and logs about scanning, embeddings, training, etc.

### Background Trainer Details

- Runs in `BackgroundTrainer` (a QThread).
- Periodically:
  - **Updates missing embeddings** (any new items).
  - **Refreshes old embeddings** (> 30 days old).
  - **Recomputes the similarity matrix** if needed (>1 hour).
  - **Trains** on recent feedback to further refine internal networks.

