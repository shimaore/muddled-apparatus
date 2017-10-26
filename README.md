Android Publisher
-----------------

Purpose:

- Integrate into gitlab-ci, using the artifacts generated by the Android Builder. For example if the Dockerfile to build the application contains (at least):

```
FROM gitlab.k-net.fr:1234/android-tv/docker.android:master
RUN mkdir /opt/src
WORKDIR /opt/src
COPY . /opt/src
RUN gradle --info assembleRelease
```

this means your gitlab-ci must contain:

```
script:
- docker build -t ${CI_REGISTRY_IMAGE}:${CI_COMMIT_REF_SLUG} --build-arg JKS="${JKS}" --build-arg JKS_PASSWORD="${JKS_PASSWORD}" .
- docker run ${CI_REGISTRY_IMAGE}:${CI_COMMIT_REF_SLUG} /bin/bash -c 'tar czf - -C /opt/src/app/build/output/apk app-release.apk .' | tar xzvf -
artifacts:
  paths:
  - app-release.apk
```

- Publish the APK onto the Google Play Store using the Google Play Developer API.
