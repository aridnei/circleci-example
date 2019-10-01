# CircleCI Example

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
