# AceUp

## Table of Contents
- [Overview](#overview)
- [Deep Dive into the Architecture & Domain Models](#deep-dive-into-the-architecture--domain-models)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Usage / Running Locally](#usage--running-locally)
- [Project Structure](#project-structure)

## Overview
AceUp is an educational platform and knowledge-base application built in Django. It provides students with access to study materials (notes, PYQs, roadmaps) and incorporates a Gemini-powered AI tutor that strictly answers questions based on the uploaded syllabus PDFs using `pdfplumber` for text extraction. It features an event management system and testimonial showcase.

## Deep Dive into the Architecture & Domain Models
The backend is structured around three primary Django apps: `user` (core content delivery), `admin` (content ingestion), and `learnGpt` (AI integration).

**Domain Models (SQLite3 / Django ORM)**
- `notes`: Stores student notes (PDFs) with attributes for `title`, `year`, `subject`, and `url`. Serves as both downloadable material and context for the AI tutor.
- `docs`: General study documents (`title`, `url`).
- `pyqs`: Previous Year Questions, categorized by `title`, `year`, `subject`, `exam`, and `url`.
- `events` & `event`: A one-to-many relationship. `events` stores core event data (`title`, `date`, `time`, `bd`, markdown `description`, `thumbnail`), while `event` stores a gallery of `photos`, `photoCaption`, and `url` tied to the parent event.
- `roadmaps`: PDF guides for various tech stacks (e.g., `ai-data-scientist.pdf`, `backend.pdf`), categorized by `title` and `url`.
- `testimonial`: Stores student feedback with `name`, `image`, `testi`, `git`, and `linkdin` URL fields.

**Unique Features**
- **Context-Aware AI Tutor (`learnGpt`)**: Unlike standard chatbots, the `/gen/` endpoint uses `pdfplumber` to extract text from all PDF notes matching a specific student's `subject` and `year`. This extracted text is prepended to the user's question and sent to `gemini-1.5-flash` using `google-generativeai`. The prompt explicitly instructs the LLM to answer "comprehensively with the primary context of notes text", keeping answers syllabus-bound.
- **Custom Admin Ingestion Forms**: Instead of relying solely on Django's default admin panel, custom views in `admin/views.py` handle bulk PDF uploads and database object creation for notes, roadmaps, PYQs, and complex formsets for events.
- **Direct File Serving**: Uses Django's `FileResponse` in `user/views.py` (`reader`, `docreader`, `pdfreader`) to stream PDFs inline directly to the browser for seamless in-app reading.

## Prerequisites
- **Language**: Python 3.8+
- **Database**: SQLite3 (default, pre-configured)
- **APIs**: Google Gemini API key (for `learnGpt` tutor)
- **Libraries**: `Django==5.1`, `google-generativeai==0.7.2`, `pdfplumber==0.11.3`, `Pillow==10.4.0`, `python-dotenv==1.0.1`, `markdown2==2.4.0`, `gunicorn==23.0.0`

## Installation & Setup

```bash
# 1. Clone the repository
git clone git@github.com:Pras2005/AceUp.git
cd AceUp

# 2. Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows use: venv\Scripts\activate

# 3. Install exact dependencies
pip install -r requirements.txt

# 4. Create a .env file and add your Google Gemini API Key
echo "GOOGLE_API_KEY=your_gemini_api_key_here" > .env

# 5. Run database migrations
python manage.py makemigrations
python manage.py migrate
```

## Usage / Running Locally

```bash
# Start the Django development server
python manage.py runserver
```

The application will be accessible at `http://127.0.0.1:8000/`.

**Core Endpoints:**
- `/`: Landing page displaying general documents.
- `/displayNotes/`: Note viewing and filtering by subject and year.
- `/gen/`: Interactive AI tutor utilizing Gemini.
- `/roadmapsview/`: Searchable PDF roadmaps for tech domains.
- `/events/`: Event listings and details.
- `/addnotes/`, `/adddocs/`, `/upload_pdfs/`: Custom administration endpoints for content ingestion.

## Project Structure

```text
AceUp/
├── AceUp/                  # Django project configuration
│   ├── settings.py         # App settings, DB config (SQLite), allowed hosts
│   └── urls.py             # Root URL routing (merges user, admin, learnGpt)
├── admin/                  # Custom data ingestion app
│   ├── views.py            # Views for bulk uploading PDFs (notes, PYQs, roadmaps)
│   └── forms.py            # Formsets for complex event creation
├── learnGpt/               # AI Tutor app
│   └── views.py            # Extracts PDF text with pdfplumber, queries Gemini API
├── user/                   # Core frontend and content delivery app
│   ├── models.py           # Domain models: notes, docs, pyqs, events, roadmaps
│   └── views.py            # Endpoints for serving inline PDFs, rendering templates
├── templates/              # HTML templates (index, notes, gpt, roadmaps, events)
├── manage.py               # Django execution script
└── requirements.txt        # Python dependencies
```
