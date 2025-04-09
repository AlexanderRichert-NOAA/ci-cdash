# ci-cdash

Follow these steps to add a CDash workflow to an NCEPLIBS repository.

## Create the project on my.cdash.org

- Enter basic repo info:

![cdash1](https://github.com/user-attachments/assets/c5ea860f-b467-42f8-a82b-d6f2ab40665a)

- Enter URL/branch:

![cdash2](https://github.com/user-attachments/assets/3615e9e4-08ab-4aff-9964-f7b2805cff25)

- Configure test settings:

![cdash3](https://github.com/user-attachments/assets/585d0315-69e2-4f87-a106-6f17c64f9925)

- Set retention time/max builds:

![cdash4](https://github.com/user-attachments/assets/b23607ec-38cd-4db1-932d-c1c6fafa457c)

- Set maintainers (remove default cdash.org maintainer):

![cdash5](https://github.com/user-attachments/assets/d0fb3680-494f-41c6-8cb9-bfc02a5b771c)

- Create an authentication token if you do not already have one configured at https://my.cdash.org/user, under "My Authentication Tokens." Set permission to "Submit only."

## Add workflow:
- Create .github/workflows/cdash.yml:
```yaml
name: cdash
on:
  merge: # make this 'pull_request' or 'push' for initial testing, then use 'merge'
    branches:
    - develop

jobs:
  cdash:
    runs-on: ubuntu-latest
    env:
      FC: gfortran
      CC: gcc
      CDASH_TOKEN: ${{ secrets.CDASH_TOKEN }}

    steps:

    # Install dependencies, typically with the NOAA-EMC/ci-build-nceplibs@oneinstalldir action

    - name: CDash
      uses: NOAA-EMC/ci-cdash@develop
      with:
        package-name: NCEPLIBS-ip
```

- Populate the CDASH_TOKEN under your repository's Settings>Secrets and Variables>Actions>Repository secrets>New repository secret:

![cdashGH](https://github.com/user-attachments/assets/045f383f-e1fb-48e6-a4df-b606123c0ab5)

