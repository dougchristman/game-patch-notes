Project Plan: Game Patch Notes Database & App
Project Setup & Planning
1.1. Define the Project’s Scope and Requirements
Document the Vision & MVP
Write a one-pager describing the product:
"A database of game patch notes with interactive diff views and notifications."
List MVP features:
Overwatch 2 patch note scraping
Follow system for games/characters
Patch comparisons
List deferred features:
Multi-language support
Premium subscriptions
Technical Decisions
Confirm the tech stack:
Backend: Django + Django REST Framework
Database: PostgreSQL (with Neon for production)
Scraping: BeautifulSoup + Requests
Web Frontend: React.js + Tailwind CSS
Mobile App: React Native with Expo
Notifications: Firebase Cloud Messaging (mobile) and SendGrid (email)
Define data sources for each game (initially Overwatch 2; later Marvel Rivals).
Documentation & Project Management
Set up a Git repository on GitHub/GitLab.
Create a project roadmap (using Trello, GitHub Projects, etc.) with milestones and issues.
Write a README.md and project charter outlining the tech stack, MVP scope, and timeline.

Backend Development with Django
2.1. Initial Project Setup
Create the Django Project and App:
BEGIN CODE SNIPPET django-admin startproject patchnotes cd patchnotes python manage.py startapp api END CODE SNIPPET
Configure Project Settings:
Set up PostgreSQL as your database in settings.py (configure Neon later).
Add installed apps: rest_framework, your api app, and authentication libraries (e.g., django-allauth).
Version Control & Environment Setup:
Create a .gitignore file.
Set up a virtual environment and document dependencies in requirements.txt.
2.2. Database Design and Models
Define Models:
Create models for Game, Character, Ability (if needed), Patch, ChangeLog, and later AbilityChange.
Example for Character:
BEGIN CODE SNIPPET
api/models.py
from django.db import models
class Game(models.Model): name = models.CharField(max_length=100) official_site = models.URLField(blank=True, null=True)
class Character(models.Model): game = models.ForeignKey(Game, on_delete=models.CASCADE, related_name='characters') name = models.CharField(max_length=100) current_stats = models.JSONField() # e.g., abilities, health, etc.
class Patch(models.Model): game = models.ForeignKey(Game, on_delete=models.CASCADE, related_name='patches') version = models.CharField(max_length=20) release_date = models.DateTimeField()
class ChangeLog(models.Model): patch = models.ForeignKey(Patch, on_delete=models.CASCADE, related_name='changelogs') character = models.ForeignKey(Character, on_delete=models.CASCADE, related_name='changes', null=True, blank=True) ability = models.CharField(max_length=100, null=True, blank=True) old_value = models.JSONField(null=True, blank=True) new_value = models.JSONField(null=True, blank=True) END CODE SNIPPET
Run Migrations:
BEGIN CODE SNIPPET python manage.py makemigrations python manage.py migrate END CODE SNIPPET
2.3. Build the REST API with Django REST Framework
Setup DRF:
Install and add rest_framework to your installed apps.
Create Serializers & Views:
Write serializers for your models.
Build API endpoints (list, detail, create, update) for games, patches, and characters.
Example serializer snippet:
BEGIN CODE SNIPPET
api/serializers.py
from rest_framework import serializers from .models import Game, Character, Patch, ChangeLog
class GameSerializer(serializers.ModelSerializer): class Meta: model = Game fields = 'all'
class CharacterSerializer(serializers.ModelSerializer): class Meta: model = Character fields = 'all' END CODE SNIPPET
Define URLs and Endpoints:
Create URL routes in api/urls.py and include them in the main urls.py.
Consider adding JWT endpoints for authentication.
2.4. Implement User Authentication
Choose a Passwordless Auth Flow:
Integrate django-allauth for magic link authentication.
Configure user models and email backends (using SendGrid for production).
API Endpoints for User Actions:
Create endpoints for user registration, login, and managing follow preferences.

Web Scraping System
3.1. Identify Data Sources and Build a Scraper for Overwatch 2
Research and Document the Source:
Identify the URL(s) and page structure for Overwatch 2 patch notes.
Document the HTML selectors and patterns to extract data.
Write a Basic Scraper Using BeautifulSoup and Requests:
Create a script (e.g., ow2_scraper.py) to fetch and parse the page.
Example:
BEGIN CODE SNIPPET import requests from bs4 import BeautifulSoup
def get_html(url): response = requests.get(url) response.raise_for_status() return response.text
def parse_ow2_patch(url): html = get_html(url) soup = BeautifulSoup(html, 'html.parser') patch_version = soup.select_one('.patch-version').get_text(strip=True) changes = [] for section in soup.select('.hero-section'): character = section.select_one('.hero-name').get_text(strip=True) ability_changes = [] ul = section.find_next('ul') if ul: for li in ul.find_all('li'): ability_changes.append(li.get_text(strip=True)) changes.append({'character': character, 'changes': ability_changes}) return { 'version': patch_version, 'changes': changes, }
if name == 'main': patch_data = parse_ow2_patch('https://www.overwatch2.com/patch-notes') print(patch_data) END CODE SNIPPET
Data Normalization Pipeline:
Convert raw scraped data into a format matching your database schema.
Create functions to map character names and ability names to existing DB records.
3.2. Integrate Scraper with Django
Create Management Commands or Celery Tasks:
Wrap your scraping logic in a Django management command (e.g., "python manage.py scrape_ow2") or as a Celery task.
Schedule Regular Scrapes:
Use Celery Beat to schedule scrapes every 30 minutes.
Document error handling and caching strategies to avoid overloading the source website.

Web Frontend Development
4.1. Setup the React Project
Initialize the Project:
BEGIN CODE SNIPPET npx create-react-app patchnotes-frontend cd patchnotes-frontend END CODE SNIPPET
Integrate Tailwind CSS:
Follow Tailwind’s installation instructions for React.
Set Up Project Structure:
Organize folders:
/src/components
/src/pages
/src/api
/src/styles
4.2. Build Core Pages and Components
Homepage:
Display trending patches.
Fetch patch data from the Django API.
Game/Character Detail Pages:
Show character stats, patch history, and an interactive timeline (using D3.js or a React timeline component).
Patch Comparison Tool:
Build a component for side-by-side comparison of patch changes.
Example component outline:
BEGIN CODE SNIPPET // src/components/PatchComparison.js import React from 'react';
function PatchComparison({ patchA, patchB }) { // Render diffs between patchA and patchB return ( <div> <h2>Patch Comparison</h2> {/* Render diff view */} </div> ); }
export default PatchComparison; END CODE SNIPPET
User Authentication & Profile:
Create pages for login (magic link) and for managing followed games/characters.
Implement API calls for authentication and user data.
4.3. Connect to the Backend API
API Service Layer:
Create a file (e.g., src/api/api.js) to centralize API calls using fetch or Axios.
Integrate endpoints for fetching patches, characters, and managing follows.

Mobile App Development (React Native)
5.1. Initialize the React Native Project with Expo
Create a New Expo Project:
BEGIN CODE SNIPPET npx create-expo-app patchnotes-mobile cd patchnotes-mobile END CODE SNIPPET
Project Structure:
Organize by screens (e.g., /screens/Home.js, /screens/NotificationCenter.js, /screens/Profile.js).
5.2. Develop Core Screens
Home/Patch Note Reader Screen:
Reuse API calls from the backend.
Follow/Notification Settings Screen:
Allow users to manage followed games/characters.
Notification Center:
Create a screen that displays notifications received via Firebase Cloud Messaging.
5.3. Integrate Push Notifications
Firebase Cloud Messaging (FCM) Setup:
Configure FCM in your Expo project.
Write helper functions to handle push notifications.
Document the process for both Android and iOS.

Notification System
6.1. Backend Notification Logic
Database Triggers / Signals:
Use Django signals (post-save) on your Patch or ChangeLog models to trigger notification tasks.
Notification Types:
Mobile Push: Triggered via Celery tasks calling FCM endpoints.
Email Notifications: Using SendGrid, triggered from the backend.
Subscription Management:
Build API endpoints for users to subscribe/unsubscribe from specific games/characters.
Store user notification preferences in the database.
6.2. Integration Testing
Simulate Notification Flow:
Manually create patch changes in the backend and verify notifications are sent.
Write tests (if possible) to automate verification of notification delivery.

Testing & Deployment
7.1. Local and Automated Testing
Unit Testing:
Write tests for Django models, serializers, views, and management commands.
For React, write component tests (using Jest/React Testing Library).
Integration Testing:
Test API endpoints with Postman or automated scripts.
Test end-to-end flows (login, data fetch, notifications).
7.2. CI/CD Pipeline
Set Up GitHub Actions:
Create workflows for:
Running backend tests (e.g., "python manage.py test").
Running frontend tests.
Linting and code quality checks.
Deployment Scripts:
Backend: Deploy on Heroku (or an alternative) and configure environment variables.
Database: Connect to Neon for PostgreSQL.
Frontend: Deploy the React app on Vercel/Netlify.
Mobile: Build and deploy the Expo app (test on simulators and devices).
7.3. Monitoring & Logging
Error Tracking:
Integrate Sentry (or similar) for backend and frontend error tracking.
Logging & Performance:
Set up logging in Django.
Monitor API performance and scraper logs.

Documentation & Project Management (Google/Meta Style)
8.1. Developer Documentation
Code Documentation:
Use docstrings in Python and JSDoc/inline comments in JavaScript.
Maintain clear READMEs in both backend and frontend repositories.
API Documentation:
Generate API docs (e.g., using DRF’s built-in schema generation or Swagger/OpenAPI).
Document each endpoint, required parameters, and example responses.
8.2. Project Management Artifacts
Design Documents:
Create technical design documents (architecture diagrams, flowcharts) for major components (backend, scraper, frontend, mobile).
Milestones & Issue Tracking:
Break down the project into milestones (e.g., "MVP Backend", "Initial Scraper", "MVP Frontend").
Use a tool (Jira, GitHub Projects, Trello) to track issues and progress.
Meeting Notes / Stand-ups (Even if Solo):
Keep a development diary or changelog to document decisions and progress.
8.3. Future Scalability & Maintenance
Scraper Maintenance Plan:
Document procedures for updating scrapers when game websites change.
User Feedback & Metrics:
Plan how to gather usage metrics and user feedback (or simulate this for a hobby project).
Iterative Roadmap:
Maintain and update a roadmap as the project grows (e.g., adding Marvel Rivals, premium features).

Final Checklist & Next Steps

Set Up Repositories & Initial Documentation:
Create separate repositories for backend and frontend.
Add project charters and initial README files.
Develop the Core Backend:
Set up Django, models, and API endpoints.
Write tests for core functionalities.
Build and Test the Overwatch 2 Scraper:
Write the scraper and integrate it as a Django management command or Celery task.
Validate data normalization and database insertion.
Create the Web Frontend (MVP):
Build the homepage, detail pages, and basic authentication flows.
Connect with the backend API.
Develop the Mobile App (Basic Version):
Set up screens and integrate with the same API.
Implement push notifications via Firebase.
Implement the Notification System:
Configure and test backend notifications.
Ensure users can subscribe and receive alerts.
Set Up Testing, CI/CD, and Deployment Pipelines:
Automate testing.
Deploy projects to Heroku, Vercel, and Neon.
Document Everything:
Ensure every step, endpoint, and component is well-documented for future reference and scalability.
