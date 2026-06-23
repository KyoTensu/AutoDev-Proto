# AutoDev-Proto

This repository is a testing/prototyping place for fully automated software development with self-hosted and Open-Source element.

## Goal

With tools like GitHub Actions, Hermes Agent, OpenHands, and others, the goal is to put in place a workflow that builds apps from a project description to something functional, and this without typing code by hand.

## Initial idea

The first architecture idea is the following:
- Using Hermes to define the projects features and let it define the tasks and sub-tasks in the form of GitHub issues
- Use GitHub actions to send tasks to Hermes when it is mentioned in an issue (as a first approach before seeking full autonomous work)
- Let the agent complete the task and create a PR, along with a brief explanation of the why
- Have an admin either accept the PR, or comment on it to get the agent modify the PR
- Repeat for every issues until all features are complete

## Initial tools
- GitHub actions for triggering, workflow and issues management
- Hermes as the agent manager
