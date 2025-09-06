FROM nginx:latest
COPY ./nginx_config.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080

CMD ["uvicorn", "main:app","--reload", "--host", "0.0.0.0", "--port", "8000"]
