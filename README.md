# Currently Working on Project - Bitbucket pipe for Scanning container image using HCL AppScan on Cloud SCA 
This repo contains windows/linux docker image that uses python to download the SAClientUtil from HCL AppScan on Cloud and run SCA analysis against an container image in Bitbucket pipelines. The script also will wait for the scan to complete and download a scan summary json file and a scan report. These files are all placed in a directory "reports" so they can be saved as artifacts of the pipeline. See the bitbucket-pipelines.yml example below. Most builds can happen on the linux image.

### Variables

The pipe has 11 variables.

| Variable |  Required | Description |
|---|---|---|
| API_KEY_ID | Required | The HCL AppScan on Cloud API Key ID |
| API_KEY_SECRET | Required | The HCL AppScan on Cloud API Key Secret |
| APP_ID | Required | The application Id of the app in AppScan on Cloud |
| TARGET_DIR | Required | The directory to be scanned. Place scan targets here. |
| CONFIG_FILE_PATH | Optional | Relative path from the repo root to an appscan config xml file. |
| SECRET_SCANNING | Optional | True or False (default). Enables the secret scanning feature of ASoC SAST. |
| REPO | Optional | The Repository name. Only really used to make filenames and comments relevant. |
| BUILD_NUM | Optional | The Bitbucket build number. Used to make filenames and comments relevant. |
| SCAN_NAME | Optional | The name of the scan in AppScan on Cloud |
| DATACENTER | Optional | ASoC Datacenter to connect to: "NA" (default) or "EU" |
| DEBUG | Optional | If true, prints additional debug info to the log. |

**Note about specifying a config file. Providing a config file can override other settings like `TARGET_DIR` or `SECRET_SCANNING`

### Example bitbucket-pipelines.yml step

The following is the bitbucket-pipelines.yml file from my demo repository that makes use of this custom pipe.

```yaml
image: gradle:6.6.0

pipelines:
  default:
    - step:
        name: Build and Test
        caches:
          - gradle
        script:
          - cd "AltoroJ 3.1.1"
          - gradle build
          - ls -la build/libs
        artifacts:
          - AltoroJ 3.1.1/build/libs/altoromutual.war
        after-script:
          - pipe: atlassian/checkstyle-report:0.3.0
    - step:
        name: ASoC SAST Scan
        script:
          # Custom Pipe to run Static Analysis via HCL AppScan on Cloud
          # View README: https://github.com/jaisonyi/asoc-container-scan-bitbucket
          - pipe: docker://jaisonyi/bitbucket_asoc_sca:test
            variables:
              # Required Variables
              API_KEY_ID: $API_KEY_ID
              API_KEY_SECRET: $API_KEY_SECRET
              APP_ID: $APP_ID
              DATACENTER: "NA"
              SECRET_SCANNING: "true"
              CONFIG_FILE_PATH: "appscan-config.xml"
              TARGET_DIR: $BITBUCKET_CLONE_DIR/AltoroJ 3.1.1/build/libs
              # Optional Variables
              REPO: $BITBUCKET_REPO_FULL_NAME
              BUILD_NUM: $BITBUCKET_BUILD_NUMBER
              SCAN_NAME: "ASoC_SCA_BitBucket"
              DEBUG: "true"
        artifacts:
          - reports/*
```

### Building The Image

Feel free to use my docker images just as shown in the example pipeline above. You can also use the following commands to build your own images and push to your dockerhub. Replace `<YOUR_DOCKERHUB>` with your dockerhub username.

Build and Push the Linux Image:
```shell
git clone https://github.com/jaisonyi/asoc-container-scan-bitbucket.git
cd bitbucket-asoc-sast/linux
docker build -t asoc_sca_linux .
docker login
docker tag asoc_sast_linux <YOUR_DOCKERHUB>/bitbucket_asoc_sca:linux
docker push <YOUR_DOCKERHUB>/bitbucket_asoc_sca:linux
```

Once your image is built, you can use them as in the example pipeline above.

```yaml
...
    - step:
        name: ASoC SAST Scan
        script:
          - pipe: docker://<YOUR_DOCKERHUB>/bitbucket_asoc_sca:linux
            variables:
              # Required Variables
              API_KEY_ID: $API_KEY_ID
              API_KEY_SECRET: $API_KEY_SECRET
              APP_ID: $ASOC_APP_ID
              DATACENTER: "NA"
              SECRET_SCANNING: "true"
              CONFIG_FILE_PATH: "appscan-config.xml"
              TARGET_DIR: $BITBUCKET_CLONE_DIR/AltoroJ 3.1.1/build/libs
              # Optional Variables
              REPO: $BITBUCKET_REPO_FULL_NAME
              BUILD_NUM: $BITBUCKET_BUILD_NUMBER
              SCAN_NAME: "HCL_ASoC_SAST"
              DEBUG: "false"
        artifacts:
          - reports/*
```

### Windows image is still under construction and does not work. 

If you have any questions raise an issue in this repo.

