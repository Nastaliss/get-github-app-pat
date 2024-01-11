# get-github-app-pat
This github action allows the generation of a Personal Access Token (PAT) for a github application.
This can be very useful as some interaction requires a PAT, and in an organization you would then need to supply a account-bound PAT. That is problematic as if the user leaves the organization this can cause problems

## Usage
To use this action you will need 3 variables. You will need admin access to the organization to install a github application to your org.
```
- uses: Nastaliss/get-github-app-pat@v1
  with:
  # Application's id, see "Getting all the required variables for your app" for more information
  app-id: ''

  # Application's installation id in your organization, see "Getting all the required variables for your app" for more information
  app-installation-id: ''

  # Application's private key for the installation in your organization, see "Getting all the required variables for your app" for more information
  app-private-key: ''
```

### Outputs
This action exposes the output `access-token` that you can use in other actions

## Application setup

There is some setup required to make this application run. You will need to create and configure an application in your organization.

### Installing an application to your org
Follow the [official github steps](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app#registering-a-github-app)

Your application is now created !

### Getting all the required variables for your app
- Still in the developer settings, click *Edit* on your application.
- Get the `App ID`, this is your app's `id`
- Scroll down to the *Private keys* section.
- Click "Generate a private key", this will download a .pem file. This is your app's `private key`

- Go back to your organization settings
- Go to Third-party Access > Github apps (this is a different menu from the one in Developers settings)
- Click `Configure` on your application
- Get the installation id in the url. The url should look like `https://github.com/organizations/<organization>/settings/installations/<installation-id>` This is your app's `installation id`

### Setting theses variables in your organization / repository
Make sure you set these variables as Github applications variables / secrets.
You can find out how to do this [for a single repository](https://docs.github.com/en/actions/learn-github-actions/variables#creating-configuration-variables-for-a-repository) or [your whole organization](https://docs.github.com/en/actions/learn-github-actions/variables#creating-configuration-variables-for-an-organization) in the official github doc.

### Using the action
The primary goal of this action is to use the [checkout](https://github.com/actions/checkout) recursively with submodules located in private repositories.
The default github token can only be used to interact with the current repository, and cannot checkout other private repos.

```
name: Checkout recursively with private submodules

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: Nastaliss/get-github-app-pat@v1
        id: githubAppAuth
        with:
          app-id: ${{ vars.submodules-app-id }}
          app-installation-id: ${{ vars.submodules-app-installation-id }}
          app-private-key: ${{ secrets.submodules-app-private-key }}
      - name: "Checkout with submodules"
        uses: actions/checkout@v3
        with:
          submodules: 'recursive'
          token: ${{ steps.githubAppAuth.outputs.access-token }}
      - name:  "enjoy"
        run: "echo yay"
```
