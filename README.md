# How to sign an iOS app on Bitrise.io

This is a small tutorial about the process of creating a Bitrise app from an iOS repository, as well as signing and distributing it.

## Creating the App
• Log in to https://bitrise.io.
• In the top-right corner, click the **Add New App** button.
• Choose an account.
• Set privacy to **Private**.
• Select your repository.
• Choose **Auto-add SSH key**. If this doesn't work, you probably don't have Admin access to the repository, so copy the SSH key and add it to your repo manually.
• Choose the branch Bitrise will clone to determine your project's configuration
• Bitrise will then try to validate your repository - this may fail, if so, proceed anyway.
• In the Project build configuration, choose a xcodeproj / xcworkspace (most likely xcworkspace), a scheme, and an IPA export method (**ad-hoc** is a good first one)
• You will be asked to review the project configuration, you can optionally click **Edit** if your project requires a specific version of Xcode. When done, click **Confirm**.
• Optionally, upload an image for your Bitrise app.
• Set up a Webhook, or not, your call.
• Go ahead and cancel the first build, it will most likely fail because the configuration is not complete yet.

## Configuring Users

When configuring the app, you will need to select a Bitrise user(s) whose accounts (both to the repository and to the Apple Developer Program) will be used. If you don't have a Bitrise user with these configured yet, click on the icon in the top-right, and select **Account settings**.
There, in the left pane, enable your repository provider and follow the setup instructions.
Then, click on **Apple Developer Account** and set it up as well. You may be required to generate an [app-specific password](https://support.apple.com/en-us/HT204397) from your AppleID.

## Configuring the App
Go to the app's main page. You'll know you're there when you see a **Start/Schedule a Build** button in the top-right corner. Then, for each tab, follow these instructions :
#### Settings
In the `Email notifications`, for the `On failed builds` dropdown, select `Send email when build status changes on the same branch`. Your team will thank you.
#### Team
• In the `Service credential User` dropdown, pick the user whose Bitrise account should be used for cloning the project.
• In the `Connected Apple Developer Portal Account` dropdown, pick the user whose Apple Developer account should be used for signing the app.
If you have an issue with these two settings, please refer to the `Configuring Users` section.

## Configuring the Workflow
Next, from the app's main page, select the **Workflow** tab. This should open the Workflow Editor. Here are the needed steps for each tab.

#### Workflows
In the top-left dropdown, pick the workflow you wish to edit - I recommand working on the **primary** workflow for now, as you'll be able to generate other workflows from this one later.
In the left pane, you should see a list of the steps your workflow contains. The default workflow is a bit useless as is. Also, it seems like Bitrise insists on creating an empty `Do anything with Script step` for that first workflow. They seem very proud of it, but we don't need it, so you can go ahead and delete it.

Here is a quick rundown on the steps you need in your workflow :

• `Activate SSH key` : this step should be already be configured since the "Creating the App" part.
• `Git Clone Repository`
• (Optional) `Bitrise.io Cache:Pull` : although you're not _required_ to use this step, we highly recommend it (along with its `Cache:Push counterpart`) as it will greatly reduce your build times.
• `iOS Auto Provision` : this step should replace the default "Certificate and profile installer" step. Here's how to configure it :
 - `Distribution type` : select `ad-hoc` for Bitrise distribution, or `app-store` for TestFlight/AppStoreConnect.
 - `The Developer Portal team id` : your Apple Developer team's ID can be found by going the [Apple Developer Portal](https://developer.apple.com), picking your team in the top-right dropdown, then going in the "Membership" option in the left pane, and finally looking at the "Team ID" line.
 - For the `Should the step try to generate Provisioning Profiles even if Xcode managed signing is enabled in the Xcode project ?` option, set `yes`.

• (Optional) `Run CocoaPods Install` : only if you use CocoaPods
• `Xcode Archive & Export for iOS`
• `Deploy to Bitrise.io` (alternatively, `Deploy to iTunes Connect - Application Loader` if you're targeting TestFlight/AppStore. This requires a `Distribution type` of `app-store`).
• (Optional) `Bitrise.io Cache:Push`

Next, go to the `Code Signing` tab. There, in the `Code signing identity` section, you should upload two .p12 files : one of type **Development** and one of type **Distribution**. See the next section to learn how to generate those.
Once uploaded, you can start your first build.

## Apple Certificates
In order to create certificates, log in to the [Developer Portal](https://developer.apple.com/account/resources/certificates/list), then head to the **Certificates** section.
#### Creating a Certificate Signing Request
• Open `KeychainAccess.app` on your mac
• In the Menu Bar, select **Keychain Access**, then **Certificate Assistant**, and finally **Request a Certificate from a Certificate Authority**.
• Fill the `User Email Address` and `Common Name` fields, then tick the `Saved to disk` option, and click **Continue**.
• Choose where to save your `.certSigningRequest` file.

Note : this Certificate Signing Request contains your mac's private key. You can technically keep it and use it for different projects and teams, no need to recreate one each time.

#### Development Certificate
• Click the blue `+` button next to Certificates.
• Tick the `iOS App Development` option, and click **Continue**.
• Upload the `.certSigningRequest` file you created previously, and click **Continue**.
• Download the certificate you just created (`.cer`), and open it to add it to your Keychain.

#### Distribution Certificate
• Click the blue `+` button next to Certificates.
• Tick the `iOS Distribution (App Store and Ad Hoc)` option, and click **Continue**.
• Upload the `.certSigningRequest` file you created previously, and click **Continue**.
• Download the certificate you just created (`.cer`), and open it to add it to your Keychain.

#### Exporting the certificates
• Open `KeychainAccess.app` on your mac.
• In the bottom-left pane (`Category`), select **My Certificates**.
• Look for your certificates here. The Development one will be called `iPhone Developer: YourName (TeamID)`, and should have a small arrow left of it. Click on this arrow to reveal the private key associated with this certificate.
• Select both the certificate and private key (cmd+click), then right-click and click **Export 2 items**. Name your certificate (typically `projectName-dev`) and choose where to save it on disk.
• You will be prompted for a password to set on this `p12` file - don't put any.
• You will probably be prompted for your session password (in order to read from the Keychain) - do it.
• Repeat the same steps for the Distribution certificate. It will probably be called `iPhone Distribution: TeamName (TeamID)` in the Keychain. Export it as `projectName-distrib`.

Great ! You now have two `p12` files which represent your signing identity and will allow Bitrise to sign your app.

## It doesn't work !

Reach to jsarda@appstud.com or switch to Android development :)
