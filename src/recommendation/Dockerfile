FROM python:3.12-slim-bookworm AS builder

RUN apt-get update && apt-get install -y ca-certificates

WORKDIR /usr/src/app

COPY requirements.txt ./

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

COPY . .

RUN opentelemetry-bootstrap -a install 


FROM gcr.io/distroless/python3-debian12

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app /usr/src/app

COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

ENV RECOMMENDATION_PORT 1010

USER nonroot:nonroot

ENTRYPOINT ["python", "recommendation_server.py"]
