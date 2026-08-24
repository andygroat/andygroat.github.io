---
title: "Copilot Instructions"
date: 2026-08-26
---

## Repos

[Classic N-Tier Repo Link](https://github.com/andygroat/todo-classic-api)

[VSA Repo Link](https://github.com/andygroat/todo-vsa-api)

## Intro

In my previous posts I have implemented an initial setup of the projects with some endpoints, added testing and setup continuous integration when checking in updates to the repositories in GitHub. Before adding any more functionality to the APIs, its time to bring in AI to assist with the development. I have been using GitHub Copilot for a while and this post will give an overview of customising Copilot to get the most of it.

## Instructions

With GitHub Copilot we can pass custom instructions to it by using a `copilot-instructions.md`. To have these instructions apply to the entire repository it needs to be located in the `.github` folder at repository level. This is a [markdown](https://www.markdownguide.org/) file, so you can use all your normal formatting etc that you are used to doing in `readme.md` files.

## Benefits

Having an instructions file for copilot is not required in you projects and prompts to copilot will still work without one, however, you may find that the responses it returns or code that it generates do not make sense for you project, or go against some design principles that you have implemented.

This is where the copilot instructions come in.

- **Coding Standards**: Tells copilot your preferred libraries, design patterns and specific syntax rules so it does not guess or use bad habits.
- **Saves Time**: Eliminates the need to re-explain project structure or folder layouts each time you open a new chat.
- **Better Inline & Chat Context**: Automatically attaches guidelines to workspace requests, making suggestions more accurate to your repository.
- **Targeted Rules**: Allows both global repository and path specific rules.

## Structure

When I first started looking at the instructions file, I wasn't sure what should be included. Luckily, Copilot can come in handy for this situation, with a simple prompt such as `generate a copilot-instructions.md file for this repo`, Copilot can generate the instructions for you that you can then customise to your needs.

In the repos above I have tried to keep a consistent format between them, this is really just for human readability, if you wish they could vary considerably between repositories. The important thing to remember is that the instructions should be kept short, no large sentances explaining the rule, simple short bullets where possible.

Here are the sections that are included in the instructions for the above repositories.

### Overview

A very quick single line or sentance explaining the project.

### Tech Stack

A list of the technologies used in the repository, .NET 10, Entity Framework etc.

### Project Structure

A list of the projects within the solution with a quick purpose of each project. Also detail any key infrastructure details such as global exception handlers or validators.

### Code Conventions

This is the first part of telling Copilot how the code should look. Containing things like naming conventions, file-scoped namespaces, the target framework, when to use the EF context, if you want to use async/await, results patterns.

### When Adding Code

This is some rules to follow when adding new features, such as where to put code, naming of classes or projects or making sure that nuget packages stay consistent if new projects get added.

### Error Handling

Specifies how errors should be handled. Are they handled through exception handlers, or do they need specific handling and what should happen for specific types of errors if applicable.

### Testing

Tells Copilot what testing framework is being used, what should be used for mocking, where to find the tests and any extra relevant testing information such if Microsoft Testing Platform is being used or if there is any sections of code that should be excluded from test coverage.

### Logging

Specifies how logging should be handled in the application, for example using Serilog and use contextual properties rather than string interpolation.

### Do

Any extra rules that Copilot should follow, could be things like using primary constructors, or when to use records over classes, or if your following VSA then ensure that all feature code is kept within its slice folder.

### Don't

A list of rules that Copilot shouldn't do.  This could things like not changing the framework or package versions, not accessing EF context directly from controllers, basically things that you really don't want Copilot to do.

## Summary

This a quick overview of the copilot instructions files that I have setup in the repositories that will aid AI assisted development. You can use these as you basis, or build or own from scratch. And if you use Claude rather Copilot, it has its own equivalent instructions file called CLAUDE.md which follows the same style.

Its also perfectly fine to start of small with only a couple of instructions if your not sure what to include and then build it up over time, and as projects develop, the instructions that you set will change.

Happy Coding!
