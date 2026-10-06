# Rasa Eco-Travel Advisor

A conversational agent, built with Rasa Open Source, that helps travellers plan
lower-carbon trips: adaptive trip intake, carbon-footprint estimates, eco-certified
accommodation search, weighted option ranking, and human-advisor handover for
complex itineraries.

## Project structure

```
rasa-eco-travel/
├── config.yml              # NLU pipeline + dialogue policies
├── domain.yml               # intents, entities, slots, forms, responses, actions
├── data/
│   ├── nlu.yml               # training examples
│   ├── stories.yml           # multi-turn dialogue flows
│   └── rules.yml              # fallback + cross-cutting rules
├── actions/
│   └── actions.py            # Climatiq / Amadeus integrations, ranking, handover
├── tests/
│   ├── test_stories.yml      # `rasa test core` scenarios
│   └── test_actions.py       # pytest unit tests for custom actions
├── frontend/
│   └── index.html            # Rasa Webchat demo UI (colour-coded cards, handover banner)
├── credentials.yml
├── endpoints.yml
├── Dockerfile.actions
├── docker-compose.yml
├── requirements-actions.txt
├── .env.example
└── .gitignore
```

## Local setup

```bash
python -m venv venv && source venv/bin/activate
pip install rasa==3.6.2 rasa-sdk==3.6.2
cp .env.example .env        # then fill in your real API keys
rasa train
```

Run the assistant (three terminals, or use Docker Compose below):

```bash
rasa run actions                 # action server on :5055
rasa shell                       # or: rasa run --enable-api --cors "*"
```

## Docker deployment

```bash
docker compose up --build
```

This starts three containers on an internal bridge network:
- `rasa` — dialogue engine, exposed on port 5005
- `action-server` — actions.py, port 5055 (kept internal in production)
- `frontend` — static demo UI, port 8080

For cloud hosting, the same `docker-compose.yml` deploys with minimal changes to
AWS ECS/Fargate, Azure Container Apps, GCP Cloud Run, or a HuggingFace Space using
the Docker SDK. Set the same environment variables from `.env.example` as secrets
in the target platform rather than committing `.env`.

## Testing

```bash
rasa test nlu --cross-validation      # intent/entity accuracy, confusion matrix
rasa test core --stories tests/test_stories.yml   # story/fallback coverage
pytest tests/test_actions.py -v       # custom action unit tests (mocked APIs)
```

## Environment variables

See `.env.example`. At minimum, `CLIMATIQ_API_KEY` is required for real carbon
estimates and `AMADEUS_CLIENT_ID` / `AMADEUS_CLIENT_SECRET` for hotel search.
Without these, the relevant actions degrade gracefully with an explanatory
fallback message rather than failing the conversation.

## Notes on data privacy

No personal data is persisted beyond the session tracker store (SQLite by
default; swap for an encrypted store in production). Human-advisor handover
only forwards the destination, dates, budget, sustainability preference and
the last ten user messages — not the full raw conversation log — to limit
data exposure in line with data-minimisation principles.
