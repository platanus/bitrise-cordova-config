# CI/CD configuration for Cordova using Bitrise

## Create bitrise project

- Go to www.bitrise.io/dashboard and click on "+ Add new app".
- Select the repository.
- Select "no, auto-add SSH key" to give access to the repo.
- Skip the brach scanning and click on "Configure manually without project scanning"
- Select the `master` branch and click *next*
- On *Project build configuration* select *Cordova* and f
    - Fill the input for the `config.xml` location with `.`
    - Select `android` platform.
    - Select *Xamarin, Visual Studio for Mac, Stable channel, on macOS* stack.
- Register the webhook cliking on *Register the webhook for me!*.

You are done. No you can go to the *workflow* section and start preparing the configuration.

## General config

The `app: envs:` section specifies Environment Variables which are available for every build, every workflow, every step.

- `APP_ID`: The bundle id for the app.
- `PLATFORMS`: To set the platform to build. It can be `android`, `ios`, or `ios,android`

If you want to run a step only when a platform is defined in `PLATFORMS` you can use the `run_if` step property like this.

```
...
steps:
- deploy-to-itunesconnect-deliver:
    run_if: |-
      {{ enveq "is_ios" "true"}}
- google-play-deploy:
    run_if: |-
      {{ enveq "is_android" "true"}}
...
```

## Staging workflow

These are the main features of the staging workflow

- It builds an android application apk that we can distribute via email.
- The application apk is considered a development app, so it should be able to be installed alongside with the production app.
  To achieve this, we append the stage as a suffix to the bundle id. `cl.platan.myapp_staging`
- To identify the development app from the production one, we add some labels to the icon and we append
  the stage to the app name. `My App - Staging`
- When you download and install the new build, you can update the existing app.
  We use the Bitrise build number as the app `versionCode`, this way we have an incremental `versionCode` on each build.

### Staging config

We can define specific configurations for the staging build in the `envs` key under the `staging` workflow.

- `STAGE: staging`
  The name of the stage
- `BUILD_CONFIG: debug` the build type pass to the cordova build command

### Debug code signing

#### Android

To be able to update the app each time we trigger a new build, we have to sign the apk with the same certificate.
Bitrise creates a new environment on each build, so the default debugging keystore is different each time hence we are going to
get an error if we try to update our app.

To overcome this problem we need to create a common keystore for every build.

Use this command to create the keystore. Use `android` as the keystore and key password.

```
keytool -genkey -v -keystore my-debug-key.keystore -alias androiddebugkey -keyalg RSA -keysize 2048 -validity 10000
```

Then upload the `my-debug-key.keystore` to bitrise in the *Code Signing* section under *generic file storage*. Use `DEBUG_KEYSTORE` as
the unique ID input.

We need to override the default keystore config in the [workflow config](#staging-config)

```yaml
- BITRISEIO_ANDROID_KEYSTORE_URL: "$BITRISEIO_DEBUG_KEYSTORE_URL"
- BITRISEIO_ANDROID_KEYSTORE_PRIVATE_KEY_PASSWORD: android
- BITRISEIO_ANDROID_KEYSTORE_ALIAS: androiddebugkey
- BITRISEIO_ANDROID_KEYSTORE_PASSWORD: android
```

### Trigger map

To trigger this workflow, we need to add some trigger map. We do this in the `trigger_map` key. By default, we start with
a map that triggers the staging workflow when we push new changes to the `master` branch.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Credits

Thank you [contributors](https://github.com/platanus/bitrise-cordova-config/graphs/contributors)!

<img src="http://platan.us/gravatar_with_text.png" alt="Platanus" width="250"/>

Bitrise Cordova Configuration is maintained by [platanus](http://platan.us).

## License

Bitrise Cordova Configuration is 2017 platanus, spa. It is free software and may be redistributed under the terms specified in the LICENSE file.
