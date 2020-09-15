# Docker Slim NodeJS image for Android (uses OpenJDK)

## Description

Slim docker image with OpenJDK and NodeJS, useful for building JavaScript based Android apps (the size is 236MB).

The image is ideal to run automated builds in the docker container, like Cordova or React-Native projects. Just use this image and run commands to build the project.

## Build image

`docker build -t openjdk-nodejs-slim .`

## Start container

`docker run --rm -v $PWD:/data openjdk-nodejs-slim`

## Example of use for CI

```
FROM szlicar/openjdk-nodejs-slim

ENV SDK_VERSION 28.0.3
ENV SDK_CHECKSUM f10f9d5bca53cc27e2d210be2cbc7c0f1ee906ad9b868748d74d62e10f2c8275

ENV ANDROID_HOME /opt/android-sdk
ENV LD_LIBRARY_PATH ${ANDROID_HOME}/tools/lib64/qt:${ANDROID_HOME}/tools/lib/libQt5:$LD_LIBRARY_PATH/

ENV GRADLE_VERSION 6.0.1
ENV GRADLE_HOME /opt/gradle-${GRADLE_VERSION}

ENV PATH ${PATH}:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/tools:${GRADLE_HOME}/bin

RUN apt update && apt install -y curl git unzip && apt clean 

# Download SDK tools
RUN curl -SLO "https://dl.google.com/android/repository/commandlinetools-linux-6200805_latest.zip" \
    && echo "${SDK_CHECKSUM} commandlinetools-linux-6200805_latest.zip" | sha256sum -c \
    && mkdir -p "${ANDROID_HOME}" \
    && unzip "commandlinetools-linux-6200805_latest.zip" -d "${ANDROID_HOME}" \
    && rm -Rf "commandlinetools-linux-6200805_latest.zip"

# Accept licenses (API 28)
RUN mkdir -p ${ANDROID_HOME}/licenses \
    && echo -e "\n24333f8a63b6825ea9c5514f83c2829b004d1fee\nd56f5187479451eabf01fb78af6dfcb131a6481e\n" > ${ANDROID_HOME}/licenses/android-sdk-license \
    && echo -e "\n84831b9409646a918e30573bab4c9c91346\n" > ${ANDROID_HOME}/licenses/android-sdk-preview-license \
    && yes | sdkmanager --sdk_root=${ANDROID_HOME} --licenses

# Install SDK packages
RUN sdkmanager --sdk_root=${ANDROID_HOME} 'platform-tools'
RUN sdkmanager --sdk_root=${ANDROID_HOME} 'platforms;android-28'
RUN sdkmanager --sdk_root=${ANDROID_HOME} 'build-tools;28.0.3'
RUN sdkmanager --sdk_root=${ANDROID_HOME} 'extras;android;m2repository'

# Install gradle
RUN curl -SL https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-all.zip -o /opt/gradle-${GRADLE_VERSION}-all.zip \
    && mkdir -p "${GRADLE_HOME}" \
    && unzip "/opt/gradle-${GRADLE_VERSION}-all.zip" -d "/opt"

WORKDIR /data

ENV CORDOVA_ANDROID_GRADLE_DISTRIBUTION_URL file:///opt/gradle-6.0.1-all.zip

# Run generate script
ENTRYPOINT /data/generate.sh
```