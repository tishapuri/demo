# ios_demo

A reference iOS project using [GitLab Mobile DevOps](https://about.gitlab.com/handbook/engineering/incubation/mobile-devops/) and [Fastlane](http://fastlane.tools/).

## Getting Started with GitLab Mobile DevOps for iOS

There are few steps required in order to automate a build, sign, and release pipeline for an iOS application. The instructions below will walk you through each step. GitLab Mobile DevOps uses [Fastlane](http://fastlane.tools/). See the [installation instructions](https://docs.fastlane.tools/getting-started/ios/setup/) for details on how to install it via Homebrew or RubyGems. 

Please note: this configuration assumes the use of [GitLab macOS Shared Runners](https://docs.gitlab.com/ee/ci/runners/saas/macos_saas_runner.html). If you haven't joined the beta yet, follow the [instructions here](https://gitlab.com/gitlab-com/runner-saas-macos-access-requests/-/issues/new) to get your project added.

### Initialize Fastlane

Ensure the application has a `Gemfile` in the root directory with the following contents:

```ruby
source "https://rubygems.org"

gem 'fastlane'
```

Then run `bundle install` to install Fastlane and all of it's dependencies.

Once installed, setting up Fastlane is often as easy as running the following command.

```
bundle exec fastlane init
```

This command will create a `fastlane` folder in the project with an `Appfile` and a stubbed out `Fastfile`. During this process it will ask you for login credentials to App Store Connect in order to generate an app identifier and App Store app if they don't already exist. If all goes well, you should end up with an `Appfile` that looks similar to [this](https://gitlab.com/gitlab-org/incubation-engineering/mobile-devops/demo-projects/ios-demo/-/blob/main/fastlane/Appfile).

If needed, see these instructions on how to [create an App Identifier](https://about.gitlab.com/handbook/engineering/incubation/mobile-devops/guides.html#how-to-create-an-app-identifier-in-the-apple-developer-portal) or [create an App entry in App Store Connect](https://about.gitlab.com/handbook/engineering/incubation/mobile-devops/guides.html#how-to-create-an-app-entry-in-app-store-connect).

### Configure Signing Certificates

The next step is to setup Fastlane Match to manage code signing files for the project. Run the following command to generate a `Matchfile` with the configuration.

```
bundle exec fastlane match init
```

This command will prompt you to choose which storage backend you want to use (select gitlab_secure_files) and to input your project path (for example: gitlab-org/gitlab). 

![](https://about.gitlab.com/images/blogimages/2022-09-19-mobile-devops-with-gitlab-part-3-code-signing-for-ios-with-gitlab-and-fastlane/match-init.png)

If everything work, you will end up with a `Matchfile` that looks similar to [this](https://gitlab.com/gitlab-org/incubation-engineering/mobile-devops/demo-projects/ios-demo/-/blob/main/fastlane/Matchfile).

#### Generate a Personal Access Token

Next, you'll need a GitLab Personal (or Project) Access Token to use Fastlane Match from your local machine. To create a Personal Access Token, visit the Access Tokens section in your GitLab profile (for example: [https://gitlab.com/-/profile/personal_access_tokens](https://gitlab.com/-/profile/personal_access_tokens)). Create a new token with the “api” scope.

#### Generate and upload

Using the token you just created, run the command below to generate development certificates and profiles in the Apple Developer portal and upload those files to GitLab. You'll be able to view the files in your project's CI/CD settings as soon as the command completes.

```
PRIVATE_TOKEN=YOUR-TOKEN bundle exec fastlane match 
```

You can also generate other certificate types by specifying the type in the command, for example:

```
PRIVATE_TOKEN=YOUR-TOKEN bundle exec fastlane match appstore
```

#### Upload-only

If you have already created signing certificates and provisioning profiles for your project, you can use Fastlane Match Import to load your existing files into GitLab. 

```
PRIVATE_TOKEN=YOUR-TOKEN bundle exec fastlane match import
```
You'll be prompted to input the path to your files. Once those options are provided, your files will be uploaded and visible in your project's CI/CD settings. (Note: If you are prompted for the git_url during the import, it is safe to leave it blank and hit enter.)

![](https://about.gitlab.com/images/blogimages/2022-09-19-mobile-devops-with-gitlab-part-3-code-signing-for-ios-with-gitlab-and-fastlane/match-import.png)

If needed, see these instructions on how to [generate a development provisioning profile from an existing certificate](https://about.gitlab.com/handbook/engineering/incubation/mobile-devops/guides.html#how-to-generate-a-development-provisioning-profile-from-an-existing-certificate) or [generate a distribution provisioning profile from an existing certificate](https://about.gitlab.com/handbook/engineering/incubation/mobile-devops/guides.html#how-to-generate-a-distribution-provisioning-profile-from-an-existing-certificate).

### Configure Test Flight Releases

This project uses the Apple App Store integration to configure interactions with App Store. Please follow the [instructions in the docs](https://docs.gitlab.com/ee/user/project/integrations/apple_app_store.html) to generate an API key with Apple and enable the integration. 


### Create a CI Pipeline

With the configuration in place, all that is needed now is to add the `.gitlab-ci.yml` and `fastlane/Fastfile` to the project.

* This [.gitlab-ci.yml](https://gitlab.com/gitlab-org/incubation-engineering/mobile-devops/demo-projects/ios-demo/-/blob/main/.gitlab-ci.yml) has all the configuration needed to run the `build` and `beta` jobs.
* The [ios/fastlane/Fastfile](https://gitlab.com/gitlab-org/incubation-engineering/mobile-devops/demo-projects/ios-demo/-/blob/main/fastlane/Fastfile) should work for most projects. You may need to customize the options in `build_app`, but since Flutter uses the name `Runner` the iOS projects, this probablly won't need to be changed.
