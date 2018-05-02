---
layout: post
title: Implementing GitLab CI for Android development
category: bpbahn
author: Melvin Witte
tags:
- gitlab-ci
- latex
---

# GitLab CI on Android
If you ever tried implementing CI for Android development, I am sure you ran into some problems you cracked your head over for a few hours. We were at that point with [GitLab CI](https://about.gitlab.com/geatures/gitlab-ci-cd) and want to share our insights with you.

<!-- more -->

## The whole .gitlab-ci.yml

```yml
image: openjdk:8-jdk

variables:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  ANDROID_COMPILE_SDK: "26"
  ANDROID_BUILD_TOOLS: "27.0.3"

before_script:
  - apt-get -q --yes update
  - apt-get -q --yes install lib32stdc++6 lib32z1
  - wget -q http://dl.google.com/android/repository/sdk-tools-linux-3859397.zip -O linux-sdk-tools.zip
  - mkdir linux-sdk-tools
  - unzip -q linux-sdk-tools.zip -d linux-sdk-tools
  - wget -q https://services.gradle.org/distributions/gradle-4.6-bin.zip -O gradle.zip
  - mkdir gradle
  - unzip -q gradle.zip -d gradle
  - chmod +x gradle/gradle-4.6/bin/gradle
  - yes | linux-sdk-tools/tools/bin/sdkmanager --licenses
  - mkdir -p /root/.android
  - touch /root/.android/repositories.cfg
  - linux-sdk-tools/tools/bin/sdkmanager --update > update.log
  - linux-sdk-tools/tools/bin/sdkmanager "platforms;android-${ANDROID_COMPILE_SDK}" "build-tools;${ANDROID_BUILD_TOOLS}" "extras;google;m2repository" "extras;android;m2repository" > installPlatform.log
  - export ANDROID_HOME=$PWD/linux-sdk-tools
  - export PATH=$PATH:$PWD/linux-sdk-tools/platform-tools/

stages:
  - assemble
  - test
  - qa

buildApp:
  stage: assemble
  script:
    - cd android
    - ../gradle/gradle-4.6/bin/gradle assemble
  artifacts:
    paths:
      - android/app/build/outputs/

unitTests:
  stage: test
  dependencies:
    - buildApp
  script:
    - cd android
    - ../gradle/gradle-4.6/bin/gradle test
  artifacts:
    name: "report-unit-tests_${CI_PROJECT_NAME}_${CI_BUILD_REF_NAME}"
    when: on_failure
    expire_in: 4 days
    paths:
      - android/app/build/reports/tests/

instrumentalTests:
  stage: test
  dependencies:
    - buildApp
  tags:
    - do
  script:
    - cd android
    - chmod +x android-wait-for-emulator
    - ../linux-sdk-tools/tools/bin/sdkmanager "system-images;android-${ANDROID_COMPILE_SDK};google_apis;x86"
    - echo no | ../linux-sdk-tools/tools/bin/avdmanager create avd -n testAVD -k "system-images;android-${ANDROID_COMPILE_SDK};google_apis;x86"
    - ../linux-sdk-tools/tools/emulator -avd testAVD -no-audio -no-window &
    - ./android-wait-for-emulator
    - adb devices
    - adb shell settings put global window_animation_scale 0 &
    - adb shell settings put global transition_animation_scale 0 &
    - adb shell settings put global animator_duration_scale 0 &
    - adb shell input keyevent 82 &
    - ../gradle/gradle-4.6/bin/gradle connectedAndroidTest
    - adb -s testAVD emu kill
  artifacts:
    name: "report-instrumental-tests_${CI_PROJECT_NAME}_{CI_BUILD_REF_NAME}"
    when: on_failure
    expire_in: 4 days
    paths:
      - android/app/build/reports/androidTests/

lint:
  stage: qa
  dependencies:
    - buildApp
  script:
    - cd android
    - ../gradle/gradle-4.6/bin/gradle lint
  artifacts:
    name: "reports_${CI_PROJECT_NAME}_${CI_BUILD_REF_NAME}.html"
    expire_in: 4 days
    paths:
      - android/app/build/reports/
```

Lets break it down step by step. First of all the:

```yml
image: openjdk:8-jdk

variables:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  ANDROID_COMPILE_SDK: "26"
  ANDROID_BUILD_TOOLS: "27.0.3"
```

First of all we declare the image. We use ```openjdk:8-jdk```, since it has Java and the JDK we need installed. It has Ubuntu 16.04 as OS.

After that, we set a few variables. ```GRADLE_OPTS: "-Dorg.gradle.daemon=false"``` tells Gradle to not use the Gradle daemon in builds. The daemon is mainly used to cache some information between builds. Since GitLab CI uses fresh docker images at the start of every build to keep the build process isolated, there won't be a chance for Gradle to cache stuff. Then we define the target SDK for this build and which build tools to use. Defining them here as variables saves us work if we want to update them later on.

Let's take a look at the before_script section now. This is run before every job of this build:

```yml
before_script:
  - apt-get -q --yes update
  - apt-get -q --yes install lib32stdc++6 lib32z1
  - wget -q http://dl.google.com/android/repository/sdk-tools-linux-3859397.zip -O linux-sdk-tools.zip
  - mkdir linux-sdk-tools
  - unzip -q linux-sdk-tools.zip -d linux-sdk-tools
  - wget -q https://services.gradle.org/distributions/gradle-4.6-bin.zip -O gradle.zip
  - mkdir gradle
  - unzip -q gradle.zip -d gradle
  - chmod +x gradle/gradle-4.6/bin/gradle
  - yes | linux-sdk-tools/tools/bin/sdkmanager --licenses
  - mkdir -p /root/.android
  - touch /root/.android/repositories.cfg
  - linux-sdk-tools/tools/bin/sdkmanager --update > update.log
  - linux-sdk-tools/tools/bin/sdkmanager "platforms;android-${ANDROID_COMPILE_SDK}" "build-tools;${ANDROID_BUILD_TOOLS}" "extras;google;m2repository" "extras;android;m2repository" > installPlatform.log
  - export ANDROID_HOME=$PWD/linux-sdk-tools
  - export PATH=$PATH:$PWD/linux-sdk-tools/platform-tools/
```
Lets break it down into smaller parts.

```yml
  - apt-get -q --yes update
  - apt-get -q --yes install lib32stdc++6 lib32z1
```

First of all, we update the system and install two packages, which are needed for the SDK tools to work.

```yml
- wget -q http://dl.google.com/android/repository/sdk-tools-linux-3859397.zip -O linux-sdk-tools.zip
- mkdir linux-sdk-tools
- unzip -q linux-sdk-tools.zip -d linux-sdk-tools
- wget -q https://services.gradle.org/distributions/gradle-4.6-bin.zip -O gradle.zip
- mkdir gradle
- unzip -q gradle.zip -d gradle
- chmod +x gradle/gradle-4.6/bin/gradle
- yes | linux-sdk-tools/tools/bin/sdkmanager --licenses
```

Then we download and unzip the SDK tools and Gradle 4.6 for later usage. The apt version of gradle is currently bugged and won't run the build correctly. We also accept the licenses for the SDKs. Without accepting the licenses, the sdkmanager won't download or update the SDKs.

```yml

- mkdir -p /root/.android
- touch /root/.android/repositories.cfg
- linux-sdk-tools/tools/bin/sdkmanager --update > update.log
- linux-sdk-tools/tools/bin/sdkmanager "platforms;android-${ANDROID_COMPILE_SDK}" "build-tools;${ANDROID_BUILD_TOOLS}" "extras;google;m2repository" "extras;android;m2repository" > installPlatform.log
- export ANDROID_HOME=$PWD/linux-sdk-tools
- export PATH=$PATH:$PWD/linux-sdk-tools/platform-tools/
```

When updating, the sdkmanager will look for ~/.android/repositories.cfg. Since the docker image runs as a root, this file is located at /root/.android/repositories.cfg. We then update existing SDKs and install the platform and build tools for android.

```yml
stages:
  - assemble
  - test
  - qa
```

We got three stages so far, ```assemble```, ```test``` and ```qa```. While assembling, the apk is built. In the test stage we run unit and instrumental tests. The quality assurance stage currently only runs lint, but in the future we plan on using more Static Code Analysis tools like [FindBugs](https://findbugs.sourceforge.net/), [PMD](https://pmd.github.io/) or [Checkstyle](https://github.com/checkstyle/checkstyle).

```yml
buildApp:
  stage: assemble
  script:
    - cd android
    - ../gradle/gradle-4.6/bin/gradle assemble
  artifacts:
    paths:
      - android/app/build/outputs/
```

The only job in the ```assemble``` stage is ```buildApp```. It enters the android folder, because that is our root for the android project. We have some more things in the repository, that's why the android project is not at the root path. Then we basically just run ```gradle assemble``` to build the apk. Since android is the root for our android app, the output artifacts are at ```android/app/build/outputs/```. This will later on be used as dependency for the tests.

```yml
unitTests:
  stage: test
  dependencies:
    - buildApp
  script:
    - cd android
    - ../gradle/gradle-4.6/bin/gradle test
  artifacts:
    name: "report-unit-tests_${CI_PROJECT_NAME}_${CI_BUILD_REF_NAME}"
    when: on_failure
    expire_in: 4 days
    paths:
      - android/app/build/reports/tests/
```

The ```unitTests``` job runs all unit tests. It depends on the ```buildApp``` job's artifacts (which will be downloaded by this job and put in the same structure they were created previously). Similar to the previous job, it simply runs ```gradle test``` in the android project root. This job only uploads artifacts in case of failure, since that is the only case we need to look at them, and these reports will only be available for four days.

```yml
instrumentalTests:
  stage: test
  dependencies:
    - buildApp
  tags:
    - do
  script:
    - cd android
    - chmod +x android-wait-for-emulator
    - ../linux-sdk-tools/tools/bin/sdkmanager "system-images;android-${ANDROID_COMPILE_SDK};google_apis;x86"
    - echo no | ../linux-sdk-tools/tools/bin/avdmanager create avd -n testAVD -k "system-images;android-${ANDROID_COMPILE_SDK};google_apis;x86"
    - ../linux-sdk-tools/tools/emulator -avd testAVD -no-audio -no-window &
    - ./android-wait-for-emulator
    - adb devices
    - adb shell settings put global window_animation_scale 0 &
    - adb shell settings put global transition_animation_scale 0 &
    - adb shell settings put global animator_duration_scale 0 &
    - adb shell input keyevent 82 &
    - ../gradle/gradle-4.6/bin/gradle connectedAndroidTest
    - adb -s testAVD emu kill
  artifacts:
    name: "report-instrumental-tests_${CI_PROJECT_NAME}_{CI_BUILD_REF_NAME}"
    when: on_failure
    expire_in: 4 days
    paths:
      - android/app/build/reports/androidTests/
```

The ```instrumentalTests``` job is a bit more complicated. First of all, it should only run on shared runners that have the ```do``` tag. Since the instrumental tests run on emulator the docker image needs to support KVM hardware acceleration. The only shared runners supporting KVM are the ```do``` tagged runner. [android-wait-for-emulator](https://github.com/travis-ci/travis-cookbooks/blob/master/community-cookbooks/android-sdk/files/default/android-wait-for-emulator) is a script that waits for an emulator to be ready for usage. We then download the image for an x86 emulator with our target SDK and create and start an avd with that system image and wait until it has started. We then disable animations and transitions since those can interfere with instrumental tests. ```keyevent 82``` unlocks the device, after which we run ```gradle connectedAndroidTest```. This gradle task is only available (check with gradle tasks) if you have an emulator running or a device connected. Afterwards we stop the emulator and, as with the unit tests, upload the reports in case of failure, which expire after four days.

```yml
lint:
  stage: qa
  dependencies:
    - buildApp
  script:
    - cd android
    - ../gradle/gradle-4.6/bin/gradle lint
  artifacts:
    name: "reports_${CI_PROJECT_NAME}_${CI_BUILD_REF_NAME}.html"
    expire_in: 4 days
    paths:
      - android/app/build/reports/
```

This runs lint. Basically it runs ```gradle lint``` and uploads the reports. Since lint also has warnings that do not fail the build, we always want to have the report.

And that's all we have so far. The next steps we want to take are running the build with [caching](https://docs.gitlab.com/ee/ci/caching), as well as implementing more static code analysis tools and making them visible in the repository.
