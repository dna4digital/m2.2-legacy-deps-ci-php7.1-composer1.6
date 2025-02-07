# **CI/CD Artifacts Vendors Magento 2.2.x**

---

### <u>**Important:**</u> 

🔸 **This image is intended for legacy Magento 2.2.x projects and should be used for development purposes only.**  
🔸 **Magento 2.2.x, PHP 7.1, Composer 1.6 are deprecated.**

---

### <u>**Overview:**</u>

This repository provides an example of a CI/CD pipeline for Magento 2.2.x, designed to:
-  Install dependencies
-  Retrieve configuration
-  Generate an applicationgit stau package
-  Deployment using artifacts

<u>**Note:**</u> This template serves as a foundation and should be customized based on specific project requirements.

Container image is hosted in [GitHub Container Registry](https://github.com/dna4digital/m2.2-legacy-deps-ci-php7.1-composer1.6/pkgs/container/m2.2-legacy-deps-ci-php7.1-composer1.6).

| Container image  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;           |
|-------------------------------------------------------------|
| `ghcr.io/dna4digital/m2.2-legacy-deps-ci-php7.1-composer1.6` |

| &#8203;Magento Version &#8203; | &#8203;Supported PHP &#8203; | &#8203;Supported Composer &#8203; |
|:---------------:|:--------------:|:-------------------:|
| `Magento 2.2.0`  | `PHP 7.0 / 7.1`  | `Composer 1.4+`  | 
| `Magento 2.2.1`  | `PHP 7.0 / 7.1`  | `Composer 1.4+`  |
| `Magento 2.2.2`  | `PHP 7.0 / 7.1`  | `Composer 1.4+`  |
| `Magento 2.2.3`  | `PHP 7.0 / 7.1`  | `Composer 1.4+`  |
| `Magento 2.2.4`  | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.5`  | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.6`  | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.7`  | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.8`  | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.9`  | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.10` | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |
| `Magento 2.2.11` | `PHP 7.0 / 7.1`  | `Composer 1.6+`  |

---

### 🦊 **Example:**


#### **.Variables**

```yaml
variables:
  GIT_DEPTH: 1
  REPO_CONFIG: "your-repo.git"
  BRANCH_CONFIG: "develop"
  GIT_USER_EMAIL: "your-email@jo.com"
  GIT_USER_NAME: "Your Name"
  CI_PROJECT_NAME: "your-project"
  DEPLOY_SERVER: "your.server.com"
  
  stages:
  - install
  - build
```

---

#### **.Git / SSH**

```yaml
.dev-requirements:
  image: ubuntu:latest
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client git -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh && chmod 700 ~/.ssh
    - ssh-keyscan $DEPLOY_SERVER >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" >> ~/.ssh/config'
    - git config --global user.email "$GIT_USER_EMAIL"
    - git config --global user.name "$GIT_USER_NAME"
```

---

#### **.Dependencies**

```yaml
dev-composer-install:
  stage: install
  image: ghcr.io/dna4digital/m2.2-legacy-deps-ci-php7.1-composer1.6
  script:
    - composer --working-dir=./path/to/your/project install --no-dev --prefer-dist
    - find ./path/to/your/project/vendor -type d -name .git -exec rm -rf {} +
    - composer --working-dir=./path/to/your/project/update install --no-dev --prefer-dist
    - find ./path/to/your/project/update/vendor -type d -name .git -exec rm -rf {} +
  artifacts:
    expire_in: 1 week
    paths:
      - path/to/your/project/vendor/
      - path/to/your/project/update/vendor/
      - path/to/your/project/var/
      - path/to/your/project/bin/
  tags:
    - docker
  only:
    - develop
```

---

#### **.Config**

```yaml
dev-config-deploy:
  stage: install
  extends: .dev-requirements
  script:
    - git clone --depth $GIT_DEPTH $REPO_CONFIG -b $BRANCH_CONFIG dev-package
  artifacts:
    expire_in: 4 weeks
    paths:
      - dev-package/
  tags:
    - docker
  only:
    - develop
```

---

#### **.Package**

```yaml
dev-package-app:
  stage: build
  image: ghcr.io/dna4digital/m2.2-legacy-deps-ci-php7.1-composer1.6
  dependencies:
    - dev-composer-install
    - dev-config-deploy
  script:
    - git archive --format=tar --output=$CI_PROJECT_NAME.tar HEAD
    - tar -rf $CI_PROJECT_NAME.tar path/to/your/project/vendor
    - tar -rf $CI_PROJECT_NAME.tar path/to/your/project/update/vendor
    - tar -rf $CI_PROJECT_NAME.tar path/to/your/project/bin
  artifacts:
    name: $CI_PROJECT_NAME
    expire_in: 1 week
    paths:
      - $CI_PROJECT_NAME.tar
  tags:
    - docker
  only:
    - develop
```

---

##### **.Additional steps:**  
🔹 Prepare the environment configuration package (env/.htaccess/.htpassword...)  
🔹 Deploy the package to the remote server  
🔹 Execute Magento post-deployment tasks or run a shell script on the server after deployment  
🔹 And other necessary configurations based on the project requirements  