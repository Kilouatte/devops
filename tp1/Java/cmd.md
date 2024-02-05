## Dockerfile du Hello World
FROM openjdk:11
COPY . /usr/src/Main.java
WORKDIR /usr/src/Main.java
RUN javac Main.java
CMD ["java", "Main"]
