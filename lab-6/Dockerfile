FROM python:3.12-slim
WORKDIR /app
RUN pip install flask
COPY app.py .
ENV APP_MODE=production
ENV APP_REGION=canada-west
EXPOSE 8080
CMD ["python", "app.py"]