Publish the APK
---------------

    seem = require 'seem'
    {debug,hand} = (require 'tangible') 'publish'

https://developers.google.com/android-publisher/tracks

1. Upload one or more APKs by calling the Edits.apks: upload method.

    publish = hand (packageName,track,version,apk,access) ->

      headers =
        'Authorization': "#{access.token_type} #{access.access_token}"

      edit =
        expiryTimeSeconds: 3600 + Date.now()//1000

      debug 'edit', edit

      {body} = yield request
        .post "https://www.googleapis.com/androidpublisher/v2/applications/#{packageName}/edits"
        .set headers
        .accept 'json'
        .send edit
        .catch (error) ->
          debug 'error', error.response.text
          Promise.reject error

      edit = body

      debug 'edit', edit

https://developers.google.com/android-publisher/api-ref/edits/apks/upload

      file = fs.createReadStream apk

      ###
      upload = request
        .post "https://www.googleapis.com/upload/androidpublisher/v2/applications/#{packageName}/edits/#{edit.id}/apks"
        .set headers
        .type 'application/vnd.android.package-archive'
        .query uploadType:'media'
      ###
      upload = rp
        .post
          url: "https://www.googleapis.com/upload/androidpublisher/v2/applications/#{packageName}/edits/#{edit.id}/apks"
          headers:
            'Authorization': "#{access.token_type} #{access.access_token}"
            'Content-Type': 'application/vnd.android.package-archive'
          qs:
            uploadType:'media'
          json: true

      try
        {body} = yield file.pipe upload
      catch error
        debug 'upload failed', error.message ? error.toString()
        process.exit 1

      debug 'upload', body

Assign to track

      {body} = yield request
        .put "https://www.googleapis.com/androidpublisher/v2/applications/#{packageName}/edits/#{edit.id}/tracks/#{track}"
        .set headers
        .send
          track: track
          versionCodes: [version]

      track = body

      debug 'track', track

Commit

      {body} = yield request
        .post "https://www.googleapis.com/androidpublisher/v2/applications/#{packageName}/edits/#{edit.id}:commit"
        .set headers

      debug 'commit', body

      return

We're using Service Accounts for authentication (not OAuth Clients).
https://developers.google.com/identity/protocols/OAuth2ServiceAccount

Generate the JWT.

    get_access_token = hand ->
      audience = 'https://www.googleapis.com/oauth2/v4/token'

      content =
        scope: 'https://www.googleapis.com/auth/androidpublisher'
      signed_jwt = jwt.sign content, gpad.private_key,
        algorithm:'RS256'
        expiresIn: '59m'
        audience: audience
        issuer: gpad.client_email

      debug 'signed_jwt', signed_jwt

Get Access-Token using the JWT (renew every hour).

      {body} = yield request
        .post audience
        .type 'form'
        .send
          grant_type: 'urn:ietf:params:oauth:grant-type:jwt-bearer'
          assertion: signed_jwt
        .catch (error) ->
          debug 'error', error.response.text
          Promise.reject error

      body

    uuidv4 = require 'uuid/v4'
    request = require 'superagent'
    rp = require 'request-promise-native'
    jwt = require 'jsonwebtoken'
    gpad = JSON.parse process.env.GOOGLE_PLAY_JSON
    fs = require 'fs'
    package_name = process.env.ANDROID_PACKAGE_NAME
    package_track = process.env.ANDROID_PACKAGE_TRACK
    package_version = parseInt process.env.ANDROID_PACKAGE_VERSION
    package_apk = process.env.ANDROID_PACKAGE_APK

    do seem ->
      access = yield get_access_token()
      yield publish package_name, package_track, package_version, package_apk, access
