# ComicCraft - AI Comic Story Creator

ComicCraft is a FastAPI + Jinja2 web application that turns a user's story idea into a five-panel comic. The original project specification defines a pipeline of structured outline generation, detailed narration/dialogue, per-panel image generation, layout assembly, and PDF export. This implementation keeps that architecture while making the AI providers configurable and adding a no-key demo mode for local testing.

## Architecture

- Frontend: server-rendered HTML, Jinja2, responsive CSS, small vanilla JS layer.
- Backend: FastAPI, route handlers in `app/routes.py`.
- Story generation: Google GenAI SDK with structured Pydantic output.
- Image generation: Hugging Face Inference Providers by default when `HF_TOKEN` is set; optional local Diffusers backend; mock image backend for zero-key smoke tests.
- PDF: `fpdf2`.
- Persistence: JSON artifacts under `data/comics/` (no database required).

## Project layout

```text
ComicCraft/
├── app/
│   ├── main.py
│   ├── routes.py
│   ├── config.py
│   ├── models.py
│   └── services/
│       ├── comic_service.py
│       ├── gemini_flash.py
│       ├── gemini_pro.py
│       ├── image_generator.py
│       ├── layout_builder.py
│       ├── exporters.py
│       └── utils.py
├── templates/
├── static/
│   ├── css/
│   ├── js/
│   ├── panels/
│   └── exports/
├── data/comics/
├── tests/
├── .env.example
├── requirements.txt
└── requirements-local-diffusion.txt
```

## Important model note

The source document names Gemini 1.5 Flash/Pro. Those model IDs are no longer the correct choice for a new 2026 deployment, so this code makes model IDs environment variables and defaults to current supported Gemini models. Change them in `.env` as your account/model availability requires.

## VS Code setup

### 1. Open the project

Open the `ComicCraft` folder in VS Code.

### 2. Create a virtual environment

Windows PowerShell:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Windows CMD:

```bat
py -3 -m venv .venv
.venv\Scripts\activate.bat
```

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Configure environment

```bash
copy .env.example .env
```

On macOS/Linux:

```bash
cp .env.example .env
```

For a zero-key smoke test, set:

```env
IMAGE_BACKEND=mock
```

You still need `GEMINI_API_KEY` for real AI story generation. For real hosted image generation, set `HF_TOKEN` and leave `IMAGE_BACKEND=auto` or set `IMAGE_BACKEND=hf`.

### 5. Run the app

```bash
uvicorn app.main:app --reload
```

Open:

- http://127.0.0.1:8000 - web UI
- http://127.0.0.1:8000/docs - Swagger API docs
- http://127.0.0.1:8000/health - health check

## Local Diffusers backend

The source documentation calls for Hugging Face Diffusers and Stable Diffusion. To run the image model locally instead of a hosted inference provider:

```bash
pip install -r requirements-local-diffusion.txt
```

Then set:

```env
IMAGE_BACKEND=local
LOCAL_DIFFUSION_MODEL=runwayml/stable-diffusion-v1-5
```

A CUDA-capable GPU is strongly recommended for practical local generation.

## API examples

### JSON comic generation

`POST /generate-comic/json`

```json
{
  "story_prompt": "A brave fox exploring an enchanted forest.",
  "character_name": "Milo",
  "setting": "Enchanted forest",
  "tone": "dramatic",
  "art_style": "comic book"
}
```

The API returns `comic_id`, the full panel layout, and a PDF download URL.

### Image-only smoke test

`POST /test-image`

```json
{
  "prompt": "A brave fox standing under glowing trees, comic book art"
}
```

## Testing

Run all tests:

```bash
pytest -q
```

The tests use mocked generation paths and do not spend API credits.

## Troubleshooting

- `GEMINI_API_KEY is not configured`: copy `.env.example` to `.env` and add your Google AI Studio key.
- `HF_TOKEN is required`: only when explicitly selecting `IMAGE_BACKEND=hf`.
- Local Diffusers is slow: switch back to `IMAGE_BACKEND=hf` or `IMAGE_BACKEND=mock`.
- If a selected Gemini model is unavailable in your account, change `GEMINI_OUTLINE_MODEL` and `GEMINI_STORY_MODEL` in `.env`.
