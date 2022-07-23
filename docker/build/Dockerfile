FROM maven:3.6.3-openjdk-8 AS build

WORKDIR /app

COPY . /app

CMD ["mvn", "clean", "package"]