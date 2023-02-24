FROM python:3.10-slim-buster


#inside container
WORKDIR /backend-flask

#inside Container

COPY requirements.txt requirements.txt

#installl python libraries
RUN pip3 install -r requirements.txt

#Outside to Inside Container
COPY . .


#Set Environment Var
ENV FLASK_ENV=development

EXPOSE ${PORT}

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

#COPY script.sh script.sh
#RUN chmod +x script.sh
#ENTRYPOINT ./script.sh