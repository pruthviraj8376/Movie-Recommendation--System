# Movie-Recommendation--System

Backend API Development Report Submitted by: Pruthviraj Bhapkar, Prathmesh Deshpande, Pranay Tondare Date: June 21, 2026
______________
Project Overview
The Smart Movie Recommendation System is a backend application built to provide users with personalized movie suggestions and efficient movie management. Users can register, authenticate securely, and access movie-related information through a set of RESTful APIs.
The application is built using Python and FastAPI, with JWT (JSON Web Token) implemented for secure authentication and authorization. It supports full CRUD operations for managing users and movie data, ensuring scalability, security, and efficient data handling for a seamless recommendation experience.
______________
Technology Stack
Technology	Purpose
FastAPI	Python web framework for RESTful endpoints and auto Swagger docs
PostgreSQL	Relational database for persistent data storage
SQLAlchemy	ORM to interact with PostgreSQL using Python models
Uvicorn	ASGI server to run the FastAPI application locally
Pydantic	Request and response schema validation
python-jose	JWT token generation and validation
passlib / bcrypt	Secure password hashing
scikit-learn	TF-IDF vectorisation and cosine similarity for recommendations
Postman	API testing and verification during development
______________
Project Structure
movie-recommendation/
├── app/
│   ├── main.py                        # FastAPI entry point, startup lifespan
│   ├── config.py                      # Environment settings via pydantic-settings
│   ├── database.py                    # SQLAlchemy engine and session
│   ├── models/
│   │   └── models.py                  # User, Movie, Watchlist, Rating tables
│   ├── schemas/
│   │   └── schemas.py                 # Pydantic request and response schemas
│   ├── routers/
│   │   ├── auth.py                    # /api/auth — register, login, profile
│   │   ├── movies.py                  # /api/movies — full CRUD
│   │   ├── recommendations.py         # /api/recommendations — TF-IDF engine
│   │   ├── watchlist.py               # /api/watchlist — add and remove
│   │   └── ratings.py                 # /api/ratings — rate and review
│   ├── services/
│   │   ├── auth_service.py            # JWT and bcrypt helpers
│   │   └── recommendation_service.py  # TF-IDF matrix and cosine similarity
│   └── utils/
│       └── auth.py                    # Your original auth functions (unchanged)
├── scripts/
│   └── seed_movies.py                 # Load movies.csv into PostgreSQL
├── .env.example                       # Environment variable template
├── requirements.txt                   # All dependencies
├── Dockerfile
├── docker-compose.yml
└── movies.csv                         # Dataset (4,803 movies)
______________
Database Schema
Users — user_id (PK), username, email, hashed_password, is_active, created_at
Movies — movie_id (PK), title, genre, release_year, description, director, cast, keywords, vote_average, popularity
Watchlist — id (PK), user_id (FK), movie_id (FK), added_at — UNIQUE(user_id, movie_id)
Ratings — id (PK), user_id (FK), movie_id (FK), score (1–10), review, created_at, updated_at — UNIQUE(user_id, movie_id)
______________
API Endpoints
Auth
POST   /api/auth/register     Register a new user
POST   /api/auth/login        Login and receive access + refresh tokens
GET    /api/auth/me           Get current user profile
PUT    /api/auth/me           Update email or password
DELETE /api/auth/me           Delete account
Movies
GET    /api/movies            List movies with search, genre, director, rating filters
GET    /api/movies/{id}       Get a single movie
POST   /api/movies            Add a movie (auth required)
PUT    /api/movies/{id}       Update a movie (auth required)
DELETE /api/movies/{id}       Delete a movie (auth required)
Recommendations
GET    /api/recommendations?movie=Avatar&top_n=10    Get similar movies (auth required)
Watchlist
GET    /api/watchlist         Get my watchlist
POST   /api/watchlist         Add a movie to watchlist
DELETE /api/watchlist/{id}    Remove from watchlist
Ratings
GET    /api/ratings           Get my ratings
POST   /api/ratings           Rate a movie (score 1–10)
PUT    /api/ratings/{id}      Update a rating
DELETE /api/ratings/{id}      Delete a rating
______________
How the Recommendation Engine Works
1.	All 4,803 movies are loaded from PostgreSQL at server startup
2.	Five text features are combined — genres, keywords, tagline, cast, and director
3.	TF-IDF vectorisation converts the text into a numerical matrix
4.	Cosine similarity is computed across all movie pairs (4,803 × 4,803 matrix)
5.	The matrix is cached in memory — every recommendation request is a sub-millisecond lookup
6.	User input is fuzzy-matched using difflib to handle typos and partial titles
7.	Top-N results are returned ranked by similarity score
______________
Setup and Installation
1. Clone the repository and navigate into it
cd movie-recommendation
2. Create and activate a virtual environment
python -m venv .venv
.venv\Scripts\activate
3. Install dependencies
pip install -r requirements.txt
4. Configure environment variables
copy .env.example .env
Edit .env and set:
DATABASE_URL=postgresql://postgres:yourpassword@localhost:5432/movie_db
JWT_SECRET_KEY=your-secret-key-here
JWT_REFRESH_SECRET_KEY=your-refresh-secret-key-here
5. Create the database in PostgreSQL
CREATE DATABASE movie_db;
6. Seed the movies dataset
python scripts/seed_movies.py --csv movies.csv
7. Start the server
uvicorn app.main:app --reload
______________
Testing
Open http://localhost:8000/docs for the interactive Swagger UI.
Quick workflow:
1.	Call POST /api/auth/register to create an account
2.	Call POST /api/auth/login to get your access token
3.	Click Authorize in Swagger and enter Bearer <your_token>
4.	Call GET /api/recommendations?movie=The+Dark+Knight&top_n=5
______________
Authentication Flow
All protected routes use the get_current_user dependency which:
•	Reads the Authorization: Bearer <token> header
•	Calls decode_token() from app/utils/auth.py to validate the JWT
•	Looks up the user in PostgreSQL and injects them into the route handler
Unauthorized requests receive a 401 Unauthorized response. Inactive accounts receive 403 Forbidden.
______________
Environment Variables
Variable	Description
DATABASE_URL	PostgreSQL connection string
JWT_SECRET_KEY	Secret key for signing access tokens
JWT_REFRESH_SECRET_KEY	Secret key for signing refresh tokens
