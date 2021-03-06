---
layout: docs
---

# Architecting Sceptre

Sceptre is written in a way that aims to be unopinionated in how it is used. It is designed to work equally well with simple and complex infrastructures. Sceptre's flexible nature, and the variation in how people organise their AWS accounts, makes it difficult to give generic advice on how best to use it, as it can be use-case specific. However, the following patterns have emerged from our use of Sceptre at Cloudreach.

## Project layout

Sceptre's nested environments means it's possible to store an entire company or department's infrastructure in a single Sceptre project. While this is possible, it isn't recommended. Having a large number of developers interact with a single repository can be difficult from a version control point of view, and it might be dangerous to let developers touch infrastructure that they are not directly involved with.

We recommend different Sceptre projects for each large 'section' of the infrastructure being built.

## Directory layout

Sceptre allows you to store your CloudFormation templates anywhere on disk. We've found it most useful to store them in a directory `templates/`, next to the `config/` directory:

```
$ ls
config
templates
```

## Environment structure

Environments can be arbitrarily nested, and environment commands can be applied to any level of the environment tree. This can make it difficult to know how to split environments up. 

When considering how to split environemnts up, it's useful to remember the following properties of environments:

1. Region specific. There is no way for an environment to launch stack in multiple regions (an environment can, however, contain sub-environments in different regions). An environment should therefore contain stacks which belong in the same region.

2. Command related. Environment level commands (like `launch-env`) are applied to every stack in an environment. There is no way to exclude stacks from an environemnt level command. Therefore, environments should contain stacks that can be launched deleted together.
    Some stacks are inherently longer-lived than others. Stacks containing VPC-level infrastructure are likely to be longer lived than stacks containing an ephemeral testing environment. It therefore makes sense to split these up into separate environments.

## Example architectures

The following examples demonstrate how we might architect various Sceptre projects. Only configuration layout is shown, and the examples are merely meant to demonstrate ways of organising different projects.

- Application development within an externally defined VPC

    ```
    - config
        - prod
            - application
                - asg.yaml
                - security-group.yaml
            - database
                - rds.yaml
                - security-group.yaml
    ```

- DevOps team who manage all the infrastructure for thier service

    ```
    - config
        - prod
            - network
                - vpc.yaml
                - subnet.yaml
            - frontend
                - api-gateway.yaml
            - application
                - lambda-get-item.yaml
                - lambda-put-item.yaml
            - database
                - dynamodb.yaml
    ```

- Centralised, company-wide networking

    ```
    - config
        - prod
            - vpc.yaml
            - public-subnet.yaml
            - application-subnet.yaml
            - database-subnet.yaml
        - dev
            - vpc.yaml
            - public-subnet.yaml
            - application-subnet.yaml
            - database-subnet.yaml
    ```

- IAM management

    ```
    - config
        - account-1
            - iam-role-admin.yaml
            - iam-role-developer.yaml
        - account-2
            - iam-role-admin.yaml
            - iam-role-developer.yaml
    ```

