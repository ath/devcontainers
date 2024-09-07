# Flutter Development Environment for Android Applications
# This Dockerfile creates an image for Flutter development without local installation of Flutter or Android Studio.
# It is optimized for use in a VS Code Dev Container and for Continuous Delivery.

# Stage 1: Builder
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04 AS builder

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
ENV ANDROID_HOME=/opt/android-sdk-linux
ENV ANDROID_SDK_ROOT=$ANDROID_HOME
ENV FLUTTER_HOME=/usr/bin/flutter
ENV PATH="$FLUTTER_HOME/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$JAVA_HOME/bin:$PATH"

# Install necessary packages
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y \
    ca-certificates \
    curl \
    git \
    unzip \
    xz-utils \
    zip \
    libglu1-mesa \
    openjdk-21-jdk-headless \
    wget \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    libgtk-3-dev \
    usbutils \
    chromium-browser && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Flutter and Android SDK
RUN mkdir -p $FLUTTER_HOME && \
    git clone -b stable https://github.com/flutter/flutter.git $FLUTTER_HOME && \
    mkdir -p ${ANDROID_HOME}/cmdline-tools && \
    wget https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip -O cmdline-tools.zip && \
    unzip cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools && \
    mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest && \
    rm cmdline-tools.zip && \
    yes | sdkmanager --licenses && \
    sdkmanager "platform-tools" "platforms;android-31" "build-tools;31.0.0" && \
    flutter precache && \
    flutter config --no-analytics && \
    flutter doctor && \
    rm -rf /tmp/* /var/tmp/*

# Stage 2: Final Image
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# Set environment variables
ENV JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
ENV ANDROID_HOME=/opt/android-sdk-linux
ENV ANDROID_SDK_ROOT=$ANDROID_HOME
ENV FLUTTER_HOME=/usr/bin/flutter
ENV PATH="$FLUTTER_HOME/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$JAVA_HOME/bin:$PATH"
ENV CHROME_EXECUTABLE=/usr/bin/chromium-browser

# Copy Flutter, Android SDK, and Java from the builder image
COPY --from=builder /usr/bin/flutter /usr/bin/flutter
COPY --from=builder /opt/android-sdk-linux /opt/android-sdk-linux
COPY --from=builder /usr/lib/jvm/java-21-openjdk-amd64 /usr/lib/jvm/java-21-openjdk-amd64

# Install minimal runtime dependencies
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y \
    ca-certificates \
    libglu1-mesa \
    openjdk-21-jre-headless \
    git \
    usbutils \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    chromium-browser && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /app

# Set permissions for the existing user
RUN chown -R vscode:vscode /app && \
    chown -R vscode:vscode $FLUTTER_HOME && \
    chown -R vscode:vscode $ANDROID_HOME && \
    chown -R vscode:vscode $JAVA_HOME

# Switch to the non-root user
USER vscode

# Add Flutter bash completion
RUN echo 'source <(flutter bash-completion)' >> ~/.bashrc

# Perform version checks and store in log files
RUN flutter doctor > /app/flutter_doctor.log 2>&1 && \
    java -version > /app/java_version.log 2>&1 && \
    sdkmanager --version > /app/sdkmanager_version.log 2>&1

# Add labels for better metadata
LABEL maintainer="Your Name <your.email@example.com>"
LABEL description="Flutter development environment for Android applications"
LABEL version="1.0"

# Note: Regular updates
# Run 'flutter upgrade' regularly in your CI/CD pipeline
# and test the image for stability with newer versions.

# Default command when starting the container
CMD ["flutter", "doctor"]

# Usage notes:
# - This image is optimized for use in a VS Code Dev Container.
# - It supports Android application development with Flutter without local installation.
# - USB debugging is supported for use with physical Android devices.
# - For Continuous Delivery, this image can be used in GitHub Actions.
# - Regular updates and tests should be performed to ensure compatibility.
