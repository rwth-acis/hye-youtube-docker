FROM rwthacis/las2peer-contact-service:develop
USER root

RUN apk add git apache-ant
RUN mkdir /build
RUN mv /src/service/i5.las2peer.services.contactService-*.jar /src/lib
RUN chown -R las2peer:las2peer /build
WORKDIR /build
USER las2peer

# Build file-service
RUN git clone https://github.com/rwth-acis/las2peer-fileservice
WORKDIR las2peer-fileservice
RUN git checkout v3.0.0
RUN chmod +x ./gradlew && ./gradlew jar
RUN cp file_service/build/libs/i5.las2peer.services.fileService-*.jar /src/lib/

# Build user-information-service
WORKDIR /build
RUN git clone https://github.com/rwth-acis/las2peer-user-information-service
WORKDIR las2peer-user-information-service/
RUN git checkout develop
RUN ant jar
RUN cp export/jars/* /src/lib/

COPY --chown=las2peer:las2peer ./i5.las2peer.services.contactService.ContactService.properties /src/etc/
COPY --chown=las2peer:las2peer ./docker-entrypoint.sh /src/
WORKDIR /src
ENTRYPOINT ["./docker-entrypoint.sh"]
