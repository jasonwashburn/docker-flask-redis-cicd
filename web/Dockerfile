FROM python:3.11.2-slim-bullseye as builder

RUN apt-get update && \
    apt-get upgrade --yes

RUN useradd --create-home pagetracker
USER pagetracker
WORKDIR /home/pagetracker

ENV VIRTUALENV=/home/pagetracker/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --chown=pagetracker:pagetracker pyproject.toml constraints.txt ./
RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir -c constraints.txt ".[dev]"

COPY --chown=pagetracker:pagetracker src/ src/
COPY --chown=pagetracker:pagetracker test/ test/

RUN python -m pip install . -c constraints.txt && \
    python -m pytest test/unit/ && \
    python -m flake8 src/ && \
    python -m isort src/ --check && \
    python -m black src/ --check && \
    python -m pylint src/ --disable=C0114,C0116,R1705 && \
    python -m bandit src/ --quiet && \
    python -m pip wheel --wheel-dir dist/ . -c constraints.txt

FROM python:3.11.3-slim-bullseye

RUN apt-get update && \
    apt-get upgrade --yes

RUN useradd --create-home pagetracker
USER pagetracker
WORKDIR /home/pagetracker

ENV VIRTUALENV=/home/pagetracker/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --from=builder /home/pagetracker/dist/page_tracker-*.whl /home/pagetracker/

RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir page_tracker-*.whl

CMD [ "flask", "--app", "page_tracker.app", "run", \
    "--host", "0.0.0.0", "--port", "5000" ]

