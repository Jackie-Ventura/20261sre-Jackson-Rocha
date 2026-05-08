FROM python:3.11-slim

# Instala dependências de sistema para psycopg2 e outras libs
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Instala as bibliotecas base da arquitetura
RUN pip install --no-cache-dir \
    duckdb \
    pandas \
    sqlalchemy \
    psycopg2-binary \
    boto3 \
    pydantic \
    python-dotenv

# Mantém o container vivo para execução de jobs
CMD ["tail", "-f", "/dev/null"]
