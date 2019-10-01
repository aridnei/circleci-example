# CircleCI Example

## CircleCI Config File

```

version: 2.1
defaults: &defaults
  working_directory: /tmp/persist_to_workspace
  docker:
    - image: circleci/node:10
      environment:
        TERM: xterm

jobs:
  checkout:
    <<: *defaults
    steps:
      - checkout
      - run: pwd
      - run: ls -lart
      - restore_cache:
          keys:
            - v1-dep-cache-{{ checksum "package.json" }}
            - v1-dep-cache
      - run:
          name: Checkout
          command: |
            echo "Checking out..."
            touch checkout.txt
            touch node_modules/module-$(date +"%Y%m%d").txt
            echo "Checkout Done!"
      - save_cache:
          key: v1-dep-cache-{{ checksum "package.json" }}
          paths:
            - node_modules
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  build-qas:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run: pwd
      - run: ls -lart
      - run:
          name: Build QAS
          command: |
            echo "Building QAS..."
            touch build-qas.txt
            echo "Build QAS Done!"
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  test:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - restore_cache:
          key: v1-dep-cache-{{ checksum "package.json" }}
      - run: pwd
      - run: ls -lart
      - run:
          name: Test All
          command: |
            echo "Testing All..."
            touch build-qas.txt
            echo "Test All Done!"
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  request-approval-qas:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run:
          name: Request Approval QAS
          command: |
            echo "Requesting Approval QAS"
            touch request-approval-qas.txt
            echo "Request Approval QAS Done!"
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  deploy-qas:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run: pwd
      - run: ls -lart
      - run:
          name: Deploy QAS
          command: |
            echo "Deploying QAS"
            touch request-approval-qas.txt
            echo "Deploy QAS Done!"

  build-stg:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run: pwd
      - run: ls -lart
      - run:
          name: Build STG
          command: |
            echo "Building STG..."
            touch build-stg.txt
            echo "Building STG Done!"
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  request-approval-stg:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run:
          name: Request Approval STG
          command: |
            echo "Requesting Approval STG"
            touch request-approval-stg.txt
            echo "Request Approval STG Done!"
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  deploy-stg:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run: pwd
      - run: ls -lart
      - run:
          name: Deploy STG
          command: |
            echo "Deploying STG"
            touch deploy-stg.txt
            echo "Deploy STG Done!"

workflows:
  version: 2.1
  workflow-project:
    jobs:
      - checkout:
          context: org-global
      - build-qas:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - checkout
      - test:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - build-qas
      - request-approval-qas:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - test
      - approve-qas:
          type: approval
          filters:
            branches: { ignore: 'master' }
          requires:
            - request-approval-qas
      - deploy-qas:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - approve-qas
      - build-stg:
          context: org-global
          filters:
            branches: { only: 'master' }
          requires:
            - checkout
      - request-approval-stg:
          context: org-global
          filters:
            branches: { only: 'master' }
          requires:
            - build-stg
      - approve-stg:
          type: approval
          filters:
            branches: { only: 'master' }
          requires:
            - request-approval-stg
      - deploy-stg:
          context: org-global
          filters:
            branches: { only: 'master' }
          requires:
            - approve-stg

```

## Config Basics

```

version: 2.1
defaults: &defaults
  working_directory: /tmp/persist_to_workspace
  docker:
    - image: circleci/node:10
      environment:
        TERM: xterm

jobs:
  checkout:
    <<: *defaults
    steps:
    ...

  buil-qas:
    <<: *defaults
    steps:
    ...

  ...

workflows:
  version: 2.1
  workflow-project:
    jobs:
      - checkout:
          context: org-global
      - build-qas:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - checkout
  ...

```


## Persistence

1. Cache Persistence

```

jobs:
  checkout:
    <<: *defaults
    steps:
      - checkout
      - run: pwd
      - run: ls -lart
      - restore_cache:
          keys:
            - v1-dep-cache-{{ checksum "package.json" }}
            - v1-dep-cache
      - run:
          name: Checkout
          command: |
            echo "Checking out..."
            touch checkout.txt
            touch node_modules/module-$(date +"%Y%m%d").txt
            echo "Checkout Done!"
      - save_cache:
          key: v1-dep-cache-{{ checksum "package.json" }}
          paths:
            - node_modules
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

```

2. Workspace percistence

```

jobs:
  checkout:
    <<: *defaults
    steps:
      - checkout
      - run: pwd
      - run: ls -lart
      ...
      - persist_to_workspace:
          root: /tmp/persist_to_workspace
          paths:
            - .

  build-qas:
    <<: *defaults
    steps:
      - attach_workspace:
          at: /tmp/persist_to_workspace
      - run: pwd
      - run: ls -lart
      - run:
          name: Build QAS
          command: |
            echo "Building QAS..."
            echo "Build QAS Done!"

```

## Jobs Sequence

```

workflows:
  version: 2.1
  workflow-project:
    jobs:
      - checkout:
          context: org-global
      - build-qas:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - checkout
      - test:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - build-qas
      - request-approval-qas:
          context: org-global
          filters:
            branches: { ignore: 'master' }
          requires:
            - test
      ...

```
