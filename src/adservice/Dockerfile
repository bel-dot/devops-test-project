FROM eclipse-temurin:25-jdk AS builder 
WORKDIR /app 
COPY build.gradle settings.gradle gradlew ./ 
COPY gradle ./gradle 
RUN chmod +x gradlew
RUN ./gradlew --no-daemon downloadRepos || true 

COPY . . 

RUN ./gradlew --no-daemon installDist 


FROM eclipse-temurin:25-jre-alpine AS runtime 

WORKDIR /app
COPY --from=builder /app .

ENTRYPOINT ["/app/build/install/hipstershop/bin/AdService"]