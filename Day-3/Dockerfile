FROM ubuntu:latest
RUN apt update && apt install -y iputils-ping
# Entrypoint is used to make sure that commands in it will run when container start and if you send any arguments through docker run ${image_name} command then it gets appended
# To completely override it, we can use --entrypoint in docker run --entrypoint ${command_name} ${image_name} ${arguments}, this will completely override the entrypoint as well as further CMD args if any present
ENTRYPOINT ["ping", "-c", "4"]
CMD ["google.com"]