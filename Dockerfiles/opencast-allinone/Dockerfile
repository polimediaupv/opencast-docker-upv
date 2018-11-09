FROM openjdk:8-jdk-slim-stretch AS build

ENV OPENCAST_SRC="/usr/src/opencast" \
    OPENCAST_REPO="https://github.com/opencast/opencast.git" \
    OPENCAST_BRANCH="r/6.x" \
    OPENCAST_VERSION="6-SNAPSHOT" \
    REMOTEPROVIDER_SRC="/usr/src/opencast-userdirectory-remoteprovider" \
    REMOTEPROVIDER_REPO="https://github.com/polimediaupv/matterhorn-userdirectory-remoteprovider.git" \
    EDITOR_SRC="/usr/src/opencast-paella-editor" \
    EDITOR_REPO="https://github.com/polimediaupv/opencast-paella-editor.git"


RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      tar gzip git \
      ca-certificates openssl \
      maven make gcc g++ libc-dev

RUN mkdir -p "${OPENCAST_SRC}" \ 
 && git clone --recursive "${OPENCAST_REPO}" "${OPENCAST_SRC}" \
 && cd "${OPENCAST_SRC}" \
 && git checkout "${OPENCAST_BRANCH}" \
 && mvn --quiet --batch-mode install -DskipTests=true -Dcheckstyle.skip=true -DskipJasmineTests=true

RUN mkdir -p "${REMOTEPROVIDER_SRC}" \
 && git clone --recursive "${REMOTEPROVIDER_REPO}" "${REMOTEPROVIDER_SRC}" \
 && cd "${REMOTEPROVIDER_SRC}" \
 && git checkout "${OPENCAST_BRANCH}" \
 && sed -i "s|<version>.*-SNAPSHOT</version>|<version>$OPENCAST_VERSION</version>|" pom.xml \
 && mvn --quiet --batch-mode install -DskipTests=true -Dcheckstyle.skip=true -DskipJasmineTests=true
 
RUN mkdir -p "${EDITOR_SRC}" \
 && git clone --recursive "${EDITOR_REPO}" "${EDITOR_SRC}" \
 && cd "${EDITOR_SRC}" \
 && git checkout "${OPENCAST_BRANCH}" \
 && sed -i "s|<version>.*-SNAPSHOT</version>|<version>$OPENCAST_VERSION</version>|" pom.xml \
 && mvn --quiet --batch-mode install -DskipTests=true -Dcheckstyle.skip=true -DskipJasmineTests=true


FROM quay.io/opencast/allinone:next

RUN apk add --no-cache git py2-pip \
    gcc g++ curl-dev python2-dev \
 && pip install --upgrade pip \
 && mkdir -p "/usr/share/VA-OC-scripts" \
 && git clone --recursive "https://github.com/polimediaupv/VA-OC-scripts.git" "/usr/share/VA-OC-scripts" \
 && pip install -r /usr/share/VA-OC-scripts/requeriments.txt


COPY --chown=opencast:opencast --from=build "/usr/src/opencast-userdirectory-remoteprovider/target/*.jar" "${OPENCAST_HOME}/deploy"
COPY --chown=opencast:opencast --from=build "/usr/src/opencast-paella-editor/target/*.jar" "${OPENCAST_HOME}/deploy"
