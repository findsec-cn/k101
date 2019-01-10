FROM python:3.7-stretch

# Copy requirements.txt and uwsgi.ini file
COPY requirements.txt /tmp/
COPY uwsgi.ini /etc/uwsgi/

# Upgrade pip and install required python packages
RUN pip install -U pip
RUN pip install -r /tmp/requirements.txt

# Which uWSGI .ini file should be used, to make it customizable
ENV UWSGI_INI /app/uwsgi.ini

# By default, run 2 processes
ENV UWSGI_CHEAPER 2

# By default, when on demand, run up to 16 processes
ENV UWSGI_PROCESSES 16

# Add demo app
COPY ./app /app
WORKDIR /app

CMD ["/usr/local/bin/uwsgi", "--ini=/etc/uwsgi/uwsgi.ini", "--die-on-term", "--need-app", "--stats-http"]

