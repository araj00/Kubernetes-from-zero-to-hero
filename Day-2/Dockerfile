# Specifies the base image to use. Here, it is a lightweight Python 3.9 image.
FROM python:3.9-slim  

# Sets /app as the working directory, in the container, for subsequent instructions.
WORKDIR /app  

# Copies the app.py file from the host to the container.
ADD app.py app.py  

# Installs the Flask library required for the application.
RUN pip install flask  

EXPOSE 5000

# Defines the default command to start the application in shell mode and then use docker top {container_name}
# CMD python app.py


# Uncomment the below command and use docker top {image_name} to see the difference between above command and below one
CMD ["python", "app.py"]